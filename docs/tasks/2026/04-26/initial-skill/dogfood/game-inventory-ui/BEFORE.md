# Before: Tangled Inventory Frame Handler

Problem scenario: an action RPG inventory panel updates every frame while the player types in a search box, changes filters, drags items, and asset icons stream in from disk/network.

```pseudo
state inventoryUi = {
  query: "",
  rarityFilter: "all",
  sortMode: "power",
  selectedItemId: null,
  visibleItems: [],
  itemCards: [],
  iconCache: {},
  pendingIconLoads: {},
  scrollTop: 0,
  isLoading: false
}

function onInventoryFrame(input, dt) {
  // Input, source state, async IO, derived model, and render commits all happen here.
  if input.searchTextChanged {
    // Debounced to avoid sorting too much, so typed text appears late.
    debounce(120ms, () => {
      inventoryUi.query = input.searchText
    })
  }

  if input.clickedRarity {
    inventoryUi.rarityFilter = input.rarity
  }

  if input.clickedSort {
    inventoryUi.sortMode = input.sortMode
  }

  if input.clickedItem {
    inventoryUi.selectedItemId = input.itemId
    loadItemDetail(input.itemId).then(detail => {
      // Can overwrite detail after the player selected something else.
      inventoryUi.selectedDetail = detail
      renderSelectedDetailPanel(detail)
    })
  }

  if input.scrollChanged {
    inventoryUi.scrollTop = input.scrollTop
  }

  inventoryUi.isLoading = true

  // Runs on every frame, even while typing or scrolling.
  let matching = []
  for item in playerInventory.items {
    if matchesQuery(item, inventoryUi.query)
      and matchesRarity(item, inventoryUi.rarityFilter) {
      matching.push(item)
    }
  }

  matching.sort((a, b) => compareItems(a, b, inventoryUi.sortMode))
  inventoryUi.visibleItems = matching

  // Builds all card presentation, including rows outside the viewport.
  inventoryUi.itemCards = []
  for item in inventoryUi.visibleItems {
    if !inventoryUi.iconCache[item.iconId] {
      inventoryUi.pendingIconLoads[item.iconId] = true

      loadIconAsync(item.iconId).then(icon => {
        // No operation id, no screen check, no item visibility check.
        inventoryUi.iconCache[item.iconId] = icon
        inventoryUi.pendingIconLoads[item.iconId] = false
        renderInventoryGrid(inventoryUi.itemCards)
      })
    }

    inventoryUi.itemCards.push({
      id: item.id,
      name: item.name,
      powerText: formatPower(item.power),
      rarityTint: rarityColor(item.rarity),
      icon: inventoryUi.iconCache[item.iconId] ?? placeholderIcon,
      equippedBadge: item.id == playerInventory.equipped[item.slot]
    })
  }

  inventoryUi.isLoading = false

  // Presentation commit is owned by the frame handler and may contain stale async updates.
  renderSearchBox(inventoryUi.query)
  renderFilterTabs(inventoryUi.rarityFilter)
  renderSortDropdown(inventoryUi.sortMode)
  renderInventoryGrid(inventoryUi.itemCards)
  renderItemCount(inventoryUi.visibleItems.length)
}
```

Failure modes:

- Typed search text is debounced with the expensive filter/sort work, so source input feels delayed.
- Filtering, sorting, card construction, and full-grid rendering run in the urgent frame path.
- Icon and item-detail async results mutate UI without checking current screen, selected item, generation, or visibility.
- Derived presentation is stored and treated like source state.
- The frame handler owns everything, so there is no clear freshness or presentation commit boundary.
