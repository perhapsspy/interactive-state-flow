# After: Refactored Interactive State Flow

The refactor keeps source state immediate and moves expensive or stale-prone work
behind an owned presentation boundary.

```tsx
type SearchSourceState = {
  query: string;
  operationId: number;
};

type SearchPresentationState = {
  querySnapshot: string;
  status: "idle" | "pending" | "ready" | "stale" | "error";
  rows: CustomerRow[];
  error: string | null;
};

function CustomerSearch({ allCustomers }: { allCustomers: Customer[] }) {
  const [source, setSource] = useState<SearchSourceState>({
    query: "",
    operationId: 0,
  });
  const [presentation, setPresentation] = useState<SearchPresentationState>({
    querySnapshot: "",
    status: "idle",
    rows: [],
    error: null,
  });

  const latestOperation = useRef(0);

  function onQueryChange(event: React.ChangeEvent<HTMLInputElement>) {
    const query = event.target.value;
    const operationId = latestOperation.current + 1;
    latestOperation.current = operationId;

    // Commit source-of-truth state immediately.
    setSource({ query, operationId });

    // Presentation may lag, but the UI tells the truth about that lag.
    setPresentation((current) => ({
      ...current,
      status: current.rows.length > 0 ? "stale" : "pending",
      error: null,
    }));
  }

  useEffect(() => {
    if (!source.query.trim()) {
      setPresentation({
        querySnapshot: source.query,
        status: "idle",
        rows: [],
        error: null,
      });
      return;
    }

    const querySnapshot = source.query;
    const operationId = source.operationId;
    const controller = new AbortController();

    setPresentation((current) => ({
      ...current,
      querySnapshot,
      status: current.rows.length > 0 ? "stale" : "pending",
      error: null,
    }));

    searchCustomers({
      allCustomers,
      query: querySnapshot,
      signal: controller.signal,
    })
      .then((rows) => {
        // Freshness gate owned by this screen boundary.
        if (controller.signal.aborted) return;
        if (latestOperation.current !== operationId) return;

        setPresentation({
          querySnapshot,
          status: "ready",
          rows,
          error: null,
        });
      })
      .catch((cause) => {
        if (controller.signal.aborted) return;
        if (latestOperation.current !== operationId) return;

        setPresentation({
          querySnapshot,
          status: "error",
          rows: [],
          error: "Search failed",
        });
      });

    return () => controller.abort();
  }, [allCustomers, source.query, source.operationId]);

  return (
    <section>
      <input
        value={source.query}
        onChange={onQueryChange}
        placeholder="Search customers"
      />

      {presentation.status === "pending" && <Spinner />}
      {presentation.status === "stale" && <StaleSearchBanner />}
      {presentation.error && <p role="alert">{presentation.error}</p>}

      <VirtualizedCustomerTable
        rows={presentation.rows}
        query={presentation.querySnapshot}
      />
    </section>
  );
}
```

The expensive work is no longer owned by the input handler.

```tsx
async function searchCustomers({
  allCustomers,
  query,
  signal,
}: {
  allCustomers: Customer[];
  query: string;
  signal: AbortSignal;
}): Promise<CustomerRow[]> {
  const remotePromise = fetch(`/api/customers?q=${encodeURIComponent(query)}`, {
    signal,
  }).then((response) => response.json());

  const remote = await remotePromise;
  if (signal.aborted) return [];

  // In a larger app this could move to a worker. The boundary remains the same:
  // input is query, output is rows, commit is owned by CustomerSearch.
  return allCustomers
    .concat(remote.customers)
    .filter((customer) => matchesCustomer(customer, query))
    .sort(compareCustomerRelevance(query))
    .slice(0, 500)
    .map((customer) => ({
      customer,
      highlightedName: buildHighlightedName(customer.name, query),
      score: calculateSearchScore(customer, query),
    }));
}
```

Commit boundary:

- `source.query` and `source.operationId` commit immediately in `onQueryChange`.
- `searchCustomers` can be cancelled and cannot mutate UI state directly.
- `CustomerSearch` owns the freshness gate and presentation commit.
- `presentation.querySnapshot` makes stale rows explicit when the source query has moved on.
- `VirtualizedCustomerTable` only renders useful visible rows instead of forcing all row work onto the interaction path.
