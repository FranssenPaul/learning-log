# Learning Log — JSXGraph Responsive Strategy

## Focus

This file isolates the layout and rendering decisions for a JSXGraph-based React page.

---

## Key takeaways

- keep a centralized scale configuration
- convert visual pixel offsets into board units
- use `ResizeObserver` when possible
- call `resizeContainer()` and `update()` after the DOM has its real size
- use `matchMedia` when the UI should react to device class

---

## Why this matters

JSXGraph visuals can look unstable when the container changes size.
The goal is to preserve:

- square geometry
- stable labels
- readable layout across screen sizes

---

## Practical notes

- `px / unitX` helps translate UI spacing into board coordinates
- `window.resize` can help, but `ResizeObserver` is cleaner
- a small no-op use like `void fontSmall` can silence lint noise when a config value is intentionally kept
