# After

No structural change is justified.

The existing shape already matches the skill's guidance for this case:

- Source-of-truth state is immediate.
- Follow-up presentation is cheap.
- There is no async freshness risk.
- Extra scheduling, operation IDs, memoization, or presentation-state boundaries would add noise.

At most, I would keep names explicit and group event handlers if the real file had enough surrounding UI to make inline handlers hard to scan:

```tsx
function CounterControl() {
  const [count, setCount] = useState(0);
  const [enabled, setEnabled] = useState(true);

  const toggleEnabled = () => setEnabled((current) => !current);
  const decrement = () => setCount((current) => current - 1);
  const increment = () => setCount((current) => current + 1);

  return (
    <section>
      <button onClick={toggleEnabled}>{enabled ? "Enabled" : "Disabled"}</button>
      <button disabled={!enabled} onClick={decrement}>-</button>
      <output>{count}</output>
      <button disabled={!enabled} onClick={increment}>+</button>
    </section>
  );
}
```

This is a readability-only option, not an interactive state-flow requirement.
