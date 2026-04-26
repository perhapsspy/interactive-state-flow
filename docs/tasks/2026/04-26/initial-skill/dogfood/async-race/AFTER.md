# After: Source State, Operation Identity, Freshness Gate

The refactor keeps the user's query current immediately, gives each request an operation id and abort handle, and routes all presentation commits through one owner.

```tsx
type SourceState = {
  query: string;
  activeSearchId: number;
};

type PresentationState = {
  shownForQuery: string;
  status: "idle" | "pending" | "ready" | "error";
  results: Product[];
  error?: string;
};

type SearchOwner = {
  latestId: number;
  latestQuery: string;
  abort?: AbortController;
};

function ProductSearch() {
  const [source, setSource] = useState<SourceState>({
    query: "",
    activeSearchId: 0,
  });
  const [presentation, setPresentation] = useState<PresentationState>({
    shownForQuery: "",
    status: "idle",
    results: [],
  });
  const searchOwner = useRef<SearchOwner>({ latestId: 0, latestQuery: "" });

  function onQueryChange(nextQuery: string) {
    // 1. Commit source-of-truth state immediately.
    const searchId = searchOwner.current.latestId + 1;
    searchOwner.current.latestId = searchId;
    searchOwner.current.latestQuery = nextQuery;

    setSource({
      query: nextQuery,
      activeSearchId: searchId,
    });

    // 2. Cancel obsolete async work.
    searchOwner.current.abort?.abort();
    const abort = new AbortController();
    searchOwner.current.abort = abort;

    // 3. Presentation is allowed to lag, but pending state is explicit.
    setPresentation((current) => ({
      ...current,
      status: nextQuery ? "pending" : "idle",
      error: undefined,
    }));

    if (!nextQuery) {
      commitSearchPresentation({
        searchId,
        query: nextQuery,
        status: "idle",
        results: [],
      });
      return;
    }

    void runSearch({ searchId, query: nextQuery, signal: abort.signal });
  }

  async function runSearch(input: {
    searchId: number;
    query: string;
    signal: AbortSignal;
  }) {
    try {
      const results = await api.searchProducts(input.query, {
        signal: input.signal,
      });

      commitSearchPresentation({
        searchId: input.searchId,
        query: input.query,
        status: "ready",
        results,
      });
    } catch (error) {
      if (input.signal.aborted) return;

      commitSearchPresentation({
        searchId: input.searchId,
        query: input.query,
        status: "error",
        results: [],
        error: "Search failed",
      });
    }
  }

  function commitSearchPresentation(next: {
    searchId: number;
    query: string;
    status: PresentationState["status"];
    results: Product[];
    error?: string;
  }) {
    // 4. Freshness gate: async completion is not commit permission.
    const isLatestOperation = next.searchId === searchOwner.current.latestId;
    const isCurrentQuery = next.query === searchOwner.current.latestQuery;
    const screenStillOwnsResult = isLatestOperation && isCurrentQuery;

    if (!screenStillOwnsResult) return;

    // 5. Presentation commits only through this owner.
    setPresentation({
      shownForQuery: next.query,
      status: next.status,
      results: next.results,
      error: next.error,
    });
  }

  return (
    <>
      <input
        value={source.query}
        onChange={(event) => onQueryChange(event.target.value)}
      />
      {presentation.status === "pending" && <Spinner />}
      <ProductResults
        query={presentation.shownForQuery}
        products={presentation.results}
        stale={presentation.shownForQuery !== source.query}
      />
    </>
  );
}
```

Behavior contracts:

- Typing updates `source.query` immediately.
- Starting query B cancels query A and advances `activeSearchId`.
- If A resolves after B, A fails the freshness gate and cannot overwrite presentation.
- The async function never mutates presentation directly; it asks `commitSearchPresentation`.
- The UI can keep showing older results while B is pending, but `stale` makes that lag explicit.
