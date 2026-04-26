# Before

Small interactive control with immediate source state, no expensive derived work, and no async result ordering risk.

```tsx
function CounterControl() {
  const [count, setCount] = useState(0);
  const [enabled, setEnabled] = useState(true);

  return (
    <section>
      <button onClick={() => setEnabled((current) => !current)}>
        {enabled ? "Enabled" : "Disabled"}
      </button>

      <button disabled={!enabled} onClick={() => setCount((current) => current - 1)}>
        -
      </button>

      <output>{count}</output>

      <button disabled={!enabled} onClick={() => setCount((current) => current + 1)}>
        +
      </button>
    </section>
  );
}
```

The interaction path records user intent directly:

- Toggle commits `enabled` immediately.
- Increment and decrement commit `count` immediately.
- Rendering the current value is cheap and synchronous.
- There is no IO, delayed computation, background work, or stale async result.
