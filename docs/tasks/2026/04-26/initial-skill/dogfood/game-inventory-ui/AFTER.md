# After: Inventory State Flow With Fresh Presentation Commits

Refactor goal: keep player intent current in the same frame, let expensive inventory presentation lag safely, and allow async asset/detail results to commit only through freshness gates.

## State Boundaries

```pseudo
source InventorySource = {
  screenKey: "inventory:player-42",
  query: "",
  rarityFilter: "all",
  sortMode: "power",
  selectedItemId: null,
  scrollTop: 0,
  generation: 0
}

presentation InventoryPresentation = {
  modelGeneration: 0,
  isModelPending: false,
  itemCount: 0,
  visibleCards: [],
  selectedDetail: null,
  staleReason: null
}

assetStore IconAssets = {
  iconsById: {},
  inFlightByIconId: {}
}
```

## Frame Input Capture

```pseudo
function onInventoryFrame(input, dt) {
  let changed = false

  if input.searchTextChanged {
    InventorySource.query = input.searchText
    changed = true
  }

  if input.clickedRarity {
    InventorySource.rarityFilter = input.rarity
    changed = true
  }

  if input.clickedSort {
    InventorySource.sortMode = input.sortMode
    changed = true
  }

  if input.clickedItem {
    InventorySource.selectedItemId = input.itemId
    changed = true
    scheduleSelectedDetailLoad(captureSourceSnapshot())
  }

  if input.scrollChanged {
    InventorySource.scrollTop = input.scrollTop
    changed = true
  }

  if changed {
    InventorySource.generation += 1
    PresentationOwner.markPending(InventorySource.generation)
    InventoryJobs.scheduleModel(captureSourceSnapshot(), visibleRangeFrom(InventorySource.scrollTop))
  }

  // Immediate feedback comes from source state, not from the lagging model.
  PresentationOwner.commitShell({
    query: InventorySource.query,
    rarityFilter: InventorySource.rarityFilter,
    sortMode: InventorySource.sortMode,
    selectedItemId: InventorySource.selectedItemId,
    pending: InventoryPresentation.isModelPending
  })
}

function captureSourceSnapshot() {
  return {
    screenKey: InventorySource.screenKey,
    generation: InventorySource.generation,
    query: InventorySource.query,
    rarityFilter: InventorySource.rarityFilter,
    sortMode: InventorySource.sortMode,
    selectedItemId: InventorySource.selectedItemId,
    scrollTop: InventorySource.scrollTop
  }
}
```

## Deferred Model Generation

```pseudo
InventoryJobs.scheduleModel(snapshot, visibleRange) {
  cancelOlderModelJobs(snapshot.screenKey)

  workerPool.enqueue("inventory-model", {
    snapshot,
    inventoryItems: playerInventory.items.versionedRead(),
    equippedBySlot: playerInventory.equipped.versionedRead(),
    visibleRange
  })
}

worker "inventory-model" message job {
  let matching = filterItems(job.inventoryItems, job.snapshot.query, job.snapshot.rarityFilter)
  sortItems(matching, job.snapshot.sortMode)

  let windowed = matching.slice(job.visibleRange.start, job.visibleRange.end)
  let cards = windowed.map(item => ({
    id: item.id,
    iconId: item.iconId,
    name: item.name,
    powerText: formatPower(item.power),
    rarityTint: rarityColor(item.rarity),
    equippedBadge: item.id == job.equippedBySlot[item.slot]
  }))

  postResult({
    screenKey: job.snapshot.screenKey,
    generation: job.snapshot.generation,
    query: job.snapshot.query,
    rarityFilter: job.snapshot.rarityFilter,
    sortMode: job.snapshot.sortMode,
    itemCount: matching.length,
    visibleCards: cards
  })
}
```

## Freshness Gates And Commit Ownership

```pseudo
PresentationOwner.acceptModelResult(result) {
  if result.screenKey != InventorySource.screenKey {
    return
  }

  if result.generation != InventorySource.generation {
    return
  }

  if result.query != InventorySource.query
    or result.rarityFilter != InventorySource.rarityFilter
    or result.sortMode != InventorySource.sortMode {
    return
  }

  InventoryPresentation.modelGeneration = result.generation
  InventoryPresentation.isModelPending = false
  InventoryPresentation.itemCount = result.itemCount
  InventoryPresentation.visibleCards = attachFreshIcons(result.visibleCards)
  InventoryPresentation.staleReason = null

  requestIconsForVisibleCards(result.visibleCards, result.generation)
  commitInventoryPresentation()
}

PresentationOwner.markPending(generation) {
  InventoryPresentation.isModelPending = true
  InventoryPresentation.staleReason = "model-lagging"
  InventoryPresentation.modelGeneration = min(InventoryPresentation.modelGeneration, generation - 1)
}

function commitInventoryPresentation() {
  renderInventoryGrid({
    cards: InventoryPresentation.visibleCards,
    stale: InventoryPresentation.isModelPending,
    itemCount: InventoryPresentation.itemCount
  })
}
```

## Async Asset Freshness

```pseudo
function requestIconsForVisibleCards(cards, generation) {
  for card in cards {
    if IconAssets.iconsById[card.iconId] {
      continue
    }

    let existing = IconAssets.inFlightByIconId[card.iconId]
    if existing and existing.screenKey == InventorySource.screenKey {
      continue
    }

    let request = {
      iconId: card.iconId,
      screenKey: InventorySource.screenKey,
      generation
    }

    IconAssets.inFlightByIconId[card.iconId] = request

    loadIconAsync(card.iconId).then(icon => {
      PresentationOwner.acceptIconResult({
        ...request,
        icon
      })
    }).catch(error => {
      PresentationOwner.acceptIconFailure({
        ...request,
        error
      })
    })
  }
}

PresentationOwner.acceptIconResult(result) {
  let currentRequest = IconAssets.inFlightByIconId[result.iconId]
  if currentRequest != result {
    return
  }

  if result.screenKey != InventorySource.screenKey {
    return
  }

  // Icons can be reused after the model generation changes, but only if a visible card still needs them.
  if !InventoryPresentation.visibleCards.any(card => card.iconId == result.iconId) {
    return
  }

  IconAssets.iconsById[result.iconId] = result.icon
  delete IconAssets.inFlightByIconId[result.iconId]

  InventoryPresentation.visibleCards = attachFreshIcons(InventoryPresentation.visibleCards)
  commitInventoryPresentation()
}
```

## Selected Detail Freshness

```pseudo
function scheduleSelectedDetailLoad(snapshot) {
  let itemId = snapshot.selectedItemId
  let detailRequest = {
    screenKey: snapshot.screenKey,
    generation: snapshot.generation,
    itemId
  }

  loadItemDetail(itemId).then(detail => {
    PresentationOwner.acceptSelectedDetail({
      ...detailRequest,
      detail
    })
  })
}

PresentationOwner.acceptSelectedDetail(result) {
  if result.screenKey != InventorySource.screenKey {
    return
  }

  if result.itemId != InventorySource.selectedItemId {
    return
  }

  InventoryPresentation.selectedDetail = result.detail
  renderSelectedDetailPanel(result.detail)
}
```

## Contract Tests

```pseudo
test "search text commits before model job completes" {
  typeSearch("ruby")
  expect(InventorySource.query).toEqual("ruby")
  expect(searchBox.text).toEqual("ruby")
  expect(InventoryPresentation.isModelPending).toEqual(true)
}

test "older model result cannot replace newer filter result" {
  let a = scheduleFor({ query: "r", generation: 1 })
  let b = scheduleFor({ query: "ruby", generation: 2 })

  complete(b)
  complete(a)

  expect(InventoryPresentation.modelGeneration).toEqual(2)
  expect(grid.cards).toMatchQuery("ruby")
}

test "icon result only commits when still visible on the active screen" {
  showVisibleIcon("sword-rare")
  leaveInventoryScreen()
  completeIconLoad("sword-rare")

  expect(grid.wasMutatedAfterLeave).toEqual(false)
}
```
