# Tailwind `inline-flex` Discovery

**Date:** 2026-01-22  
**Topic:** Mastering `inline-flex` for mathematical content layout

---

## The discovery

While working on math exercises for paul.craft(), discovered the power of `inline-flex`:
```html
<span class="inline-flex gap-60">
  $\ln(e^3)=\ldots$
  $e^{\ln(5)}=\ldots$
  $\ln(e^{-2})=\ldots$
</span>
```

**Result**: Mathematical formulas aligned horizontally with uniform spacing, staying in text flow.

---

## What is `inline-flex`?

### Combination of two concepts

**`inline`**: Element stays in text flow (like `<span>`)  
**`flex`**: Element uses flexbox for its children

### Visual difference

**`flex` (block-level)**:
```text
┌─────────────────────────────┐
│ A    B                      │
└─────────────────────────────┘
Text below (new line)
```

**`inline-flex` (inline-level)**:
```text
Text before A    B text after
```

**Advantages**:
- `gap` is applied automatically between all elements
- spacing stays consistent
- changing spacing means changing one class
- the layout stays responsive-friendly

---

## Useful variations

### Vertical alignment
```html
<span class="inline-flex gap-8 items-center">
  $\ln(e^3)$ $e^{\ln(5)}$ $\ln(e^{-2})$
</span>
```

### Responsive spacing
```html
<span class="inline-flex gap-4 md:gap-8 lg:gap-16">
  $\ln(e^3)$ $e^{\ln(5)}$ $\ln(e^{-2})$
</span>
```

### Auto-wrap
```html
<span class="inline-flex gap-8 flex-wrap">
  $\ln(e^3)$ $e^{\ln(5)}$ $\ln(e^{-2})$ $e^{\ln(7)}$ $\ln(e^{10})$
</span>
```

---

## Practical use cases

### Multiple exercise questions
```html
<div class="inline-flex gap-8 flex-wrap">
  <span>a) $\ln(e^3)=\ldots$</span>
  <span>b) $e^{\ln(5)}=\ldots$</span>
  <span>c) $\ln(e^{-2})=\ldots$</span>
</div>
```

### Step-by-step solutions
```html
<div class="space-y-4">
  <div class="inline-flex gap-4 items-center">
    <span class="font-bold">Step 1:</span>
    <span>$\ln(e^3) = 3\ln(e)$</span>
  </div>
  <div class="inline-flex gap-4 items-center">
    <span class="font-bold">Step 2:</span>
    <span>$= 3 \times 1$</span>
  </div>
  <div class="inline-flex gap-4 items-center">
    <span class="font-bold text-orange-600">Result: $3$</span>
  </div>
</div>
```

### Property comparison
```html
<div class="inline-flex gap-8 items-baseline">
  <span class="text-gray-600">Property:</span>
  <span>$\ln(e^x) = x$</span>
  <span class="text-gray-400">and</span>
  <span>$e^{\ln(x)} = x$</span>
</div>
```

### Horizontal MCQ
```html
<p>What is $\ln(e^5)$?</p>
<div class="inline-flex gap-6 flex-wrap">
  <label class="inline-flex items-center gap-2">
    <input type="radio" name="q1">
    <span>a) $1$</span>
  </label>
  <label class="inline-flex items-center gap-2">
    <input type="radio" name="q1">
    <span>b) $5$</span>
  </label>
  <label class="inline-flex items-center gap-2">
    <input type="radio" name="q1">
    <span>c) $e$</span>
  </label>
</div>
```

---

## Related utilities

### Grid for matrix layout
```html
<div class="inline-grid grid-cols-3 gap-4">
  <span>$a_{11}$</span> <span>$a_{12}$</span> <span>$a_{13}$</span>
  <span>$a_{21}$</span> <span>$a_{22}$</span> <span>$a_{23}$</span>
  <span>$a_{31}$</span> <span>$a_{32}$</span> <span>$a_{33}$</span>
</div>
```

### Vertical stack
```html
<div class="inline-flex flex-col gap-2">
  <span>$\ln(e^3) = 3$</span>
  <span>$\ln(e^5) = 5$</span>
  <span>$\ln(e^{-2}) = -2$</span>
</div>
```

---

## Gap values reference

```css
gap-1  -> 4px
gap-2  -> 8px
gap-4  -> 16px
gap-8  -> 32px
gap-12 -> 48px
gap-16 -> 64px
gap-60 -> 240px
```

Recommendation: for math content, `gap-4` to `gap-8` works best.

---

## Key takeaway

`inline-flex` + `gap` gives clean, consistent inline layouts for formulas and exercises, without manual spacing hacks.
