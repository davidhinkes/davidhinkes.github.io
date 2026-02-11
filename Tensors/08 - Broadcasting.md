# Broadcasting

Broadcasting is the mechanism that lets you combine tensors of different shapes in [[02 - Element-wise Operations|element-wise operations]] without manually copying data. It's how a bias vector gets added to every row of a matrix, and how a scalar scales an entire tensor.

---

## The rules

When two tensors have different shapes, broadcasting aligns their dimensions right-to-left and applies two rules:

1. **If one tensor has fewer axes**, pad its shape with 1s on the left
2. **For each axis**, the sizes must either match or one of them must be 1. The size-1 axis is "stretched" to match the other.

If any axis has two different sizes, neither of which is 1, it's an error.

---

## Examples

### Bias addition: (B×n) + (n,)

```
Shape of Y:  (4, 3)
Shape of b:     (3,)

Step 1: pad b → (1, 3)
Step 2: axis 0: 4 vs 1 → stretch b to 4
        axis 1: 3 vs 3 → match

Result shape: (4, 3)
```

Each row of **Y** gets b added to it:

```
Y = [1  2  3]    b = [10, 20, 30]
    [4  5  6]
    [7  8  9]
    [0  1  2]

Y + b = [11  22  33]
        [14  25  36]
        [17  28  39]
        [10  21  32]
```

This is how `Y = X·Wᵀ + b` works in a dense layer — the bias (n,) is broadcast across the batch dimension.

### Scalar multiplication: (m×n) * ()

```
Shape of A: (3, 4)
Shape of α:      ()

Step 1: pad α → (1, 1)
Step 2: axis 0: 3 vs 1 → stretch
        axis 1: 4 vs 1 → stretch

Result shape: (3, 4)
```

Every element of **A** is multiplied by α.

### Column scaling: (m×n) * (m, 1)

```
Shape of A: (3, 4)
Shape of s: (3, 1)

axis 0: 3 vs 3 → match
axis 1: 4 vs 1 → stretch s

Result shape: (3, 4)
```

Each row i of **A** is multiplied by s_i. The (m, 1) shape broadcasts along columns.

### Row scaling: (m×n) * (1, n) or (m×n) * (n,)

```
Shape of A: (3, 4)
Shape of s:    (4,)  → padded to (1, 4)

axis 0: 3 vs 1 → stretch s
axis 1: 4 vs 4 → match

Result shape: (3, 4)
```

Each column j of **A** is multiplied by s_j.

---

## Broadcasting failures

```
(3, 4) + (3,)  →  ERROR
```

Why? After padding: (3, 4) vs (1, 3). Axis 1: 4 vs 3 — neither is 1, so broadcasting fails.

This is a common bug: you meant to add a vector to each row (axis 1 has length 4, so the vector should have length 4), but accidentally passed a length-3 vector.

---

## The (m, 1) vs. (m,) distinction

This trips people up. A vector (m,) and a column vector (m, 1) behave differently under broadcasting:

```
A has shape (3, 4)

A + v where v is (3,):
  pad → (1, 3)
  axis 1: 4 vs 3 → ERROR

A + v where v is (3, 1):
  axis 0: 3 vs 3 → match
  axis 1: 4 vs 1 → stretch
  → (3, 4) ✓
```

If you want to add a vector along axis 0 (one value per row), you need shape (m, 1) — an explicit column. In code: `v.reshape(-1, 1)` or `v[:, None]`.

---

## No data is actually copied

Broadcasting is a **virtual** expansion. The framework uses stride tricks: a size-1 axis gets stride 0, so the same element is read repeatedly. Memory usage stays at the original size. This is why broadcasting is efficient — it's an addressing trick, not a data copy.

---

## Broadcasting in deep learning

| Operation | Shapes | What broadcasts |
|-----------|--------|----------------|
| Bias add | (B×n) + (n,) | bias across batch |
| Layer norm | (B×n) ⊙ (n,) | scale across batch |
| Attention mask | (B×S×S) + (1×1×S) | mask across batch and queries |
| Weight decay | **W** - α·**W** | scalar α across all weights |
| Per-channel scale | (B×C×H×W) ⊙ (1×C×1×1) | channel weights across spatial dims |

---

Previous: [[07 - Transpose and Reshaping]] | Next: [[09 - Batch Dimensions]]
