# Before: Tangled React Search/List Flow

This component keeps the typed query, server request, expensive local filtering,
sorting, highlighting, and rendered rows in one urgent path.

```tsx
function CustomerSearch({ allCustomers }: { allCustomers: Customer[] }) {
  const [query, setQuery] = useState("");
  const [rows, setRows] = useState<CustomerRow[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function onChange(event: React.ChangeEvent<HTMLInputElement>) {
    const nextQuery = event.target.value;

    // Input state is delayed to reduce list work.
    await sleep(250);
    setQuery(nextQuery);

    setLoading(true);
    setError(null);

    try {
      const remote = await fetch(`/api/customers?q=${encodeURIComponent(nextQuery)}`)
        .then((response) => response.json());

      // Expensive presentation is mixed into the request handler.
      const filtered = allCustomers
        .concat(remote.customers)
        .filter((customer) => matchesCustomer(customer, nextQuery))
        .sort(compareCustomerRelevance(nextQuery))
        .map((customer) => ({
          customer,
          highlightedName: buildHighlightedName(customer.name, nextQuery),
          score: calculateSearchScore(customer, nextQuery),
        }));

      // A slow response for an old query can overwrite newer results.
      setRows(filtered);
    } catch (cause) {
      setError("Search failed");
    } finally {
      setLoading(false);
    }
  }

  return (
    <section>
      <input value={query} onChange={onChange} placeholder="Search customers" />
      {loading && <Spinner />}
      {error && <p role="alert">{error}</p>}
      <CustomerTable rows={rows} />
    </section>
  );
}
```

Problems:

- User intent is debounced, so the input can feel late.
- The event handler owns source state, IO, expensive derivation, and presentation.
- There is no operation identity or cancellation for stale network results.
- Local filtering, sorting, scoring, and highlight generation run before commit.
- Presentation has no explicit stale or pending state.
