# Before: Stale Typeahead Results Can Win

A realistic failure: the user types quickly, each input starts a search, and the slowest response gets the last write even when it belongs to an older query.

```tsx
type SearchState = {
  query: string;
  loading: boolean;
  results: Product[];
  error?: string;
};

function ProductSearch() {
  const [state, setState] = useState<SearchState>({
    query: "",
    loading: false,
    results: [],
  });

  async function onQueryChange(nextQuery: string) {
    setState((current) => ({
      ...current,
      query: nextQuery,
      loading: true,
      error: undefined,
    }));

    try {
      const results = await api.searchProducts(nextQuery);

      // Bug: this response may belong to an older query.
      // Example: "mac" starts first, "macbook" starts second,
      // "macbook" resolves first, then "mac" overwrites the UI.
      setState((current) => ({
        ...current,
        loading: false,
        results,
      }));
    } catch (error) {
      setState((current) => ({
        ...current,
        loading: false,
        error: "Search failed",
      }));
    }
  }

  return (
    <>
      <input
        value={state.query}
        onChange={(event) => onQueryChange(event.target.value)}
      />
      {state.loading && <Spinner />}
      <ProductResults query={state.query} products={state.results} />
    </>
  );
}
```

Problems:

- The input source state and async presentation state are mixed in one handler.
- No operation identity exists, so completion order becomes commit order.
- No cancellation exists, so older requests keep doing work.
- Results commit directly from the async callback instead of through an owner that knows the current query and screen context.
