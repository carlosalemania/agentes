# JavaScript Performance Optimizer - Examples

---

## Example: Optimized Data Table

```jsx
import { useVirtualizer } from '@tanstack/react-virtual';

function DataTable({ rows }) {
  const parentRef = useRef();

  const rowVirtualizer = useVirtualizer({
    count: rows.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
  });

  return (
    <div ref={parentRef} style={{ height: '600px', overflow: 'auto' }}>
      <div
        style={{
          height: `${rowVirtualizer.getTotalSize()}px`,
          position: 'relative',
        }}
      >
        {rowVirtualizer.getVirtualItems().map((virtualRow) => (
          <div
            key={virtualRow.index}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualRow.size}px`,
              transform: `translateY(${virtualRow.start}px)`,
            }}
          >
            {rows[virtualRow.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

**Versión:** 1.0.0
