# Transpose and Reshaping

Three operations change how tensor data is organized without doing any arithmetic: **transpose**, **reshape**, and **permute**. They show up constantly in deep learning code and are essential for making dimensions line up before a multiply.

---

## Transpose

Transpose swaps the two axes of a matrix: rows become columns, columns become rows.

```
Aᵀ_ij = A_ji       if A is (m×n), then Aᵀ is (n×m)
```

```
A = [1  2  3]     Aᵀ = [1  4]
    [4  5  6]          [2  5]
                       [3  6]

A is (2×3)         Aᵀ is (3×2)
```

### Why transpose appears everywhere in backprop

Consider the forward pass of a dense layer:

```
Forward:  y = W·x       (m,) = (m×n) · (n,)
```

The backward pass for the input gradient is:

```
Backward: dLdX = Wᵀ · dLdY     (n,) = (n×m) · (m,)
```

**Why Wᵀ?** Derive it from the chain rule:

```
∂L/∂x_j = Σ_i (∂L/∂y_i) · (∂y_i/∂x_j)
         = Σ_i dLdY_i · W_ij
         = Σ_i Wᵀ_ji · dLdY_i
         = (Wᵀ · dLdY)_j
```

The summation index i is over the *output* dimension. In the forward pass, **W** maps input→output. In the backward pass, **Wᵀ** maps output→input — it reverses the flow.

This pattern holds universally: **the backward pass through a linear operation uses the transpose of the forward operator**.

---

## Reshape

Reshape reinterprets the same flat block of memory with a different shape. The total number of elements must stay the same.

```
A = [1  2  3  4  5  6]     shape (6,)

reshape(A, (2, 3)) = [1  2  3]     shape (2×3)
                      [4  5  6]

reshape(A, (3, 2)) = [1  2]        shape (3×2)
                      [3  4]
                      [5  6]

reshape(A, (1, 6)) = [1  2  3  4  5  6]   shape (1×6)
```

Reshape does **not** move data in memory — it just changes the shape metadata. This makes it essentially free.

### Common reshapes in deep learning

| From | To | Why |
|------|-----|-----|
| (B, C, H, W) | (B, C·H·W) | Flatten spatial dims before a dense layer |
| (B, seq, heads·d) | (B, seq, heads, d) | Split into attention heads |
| (m×n) | (m, 1, n) | Add a broadcast dimension |

---

## Permute (generalized transpose)

Permute reorders axes. Transpose is the rank-2 special case (swap axes 0 and 1). For higher-rank tensors, you specify the new axis order.

```
T has shape (B, C, H, W) — axes [0, 1, 2, 3]

permute(T, [0, 2, 3, 1]) → shape (B, H, W, C)
```

This moves the channel axis from position 1 to position 3. Unlike reshape, permute **does** (potentially) move data in memory because the elements need to appear in a different order.

### Permute vs. reshape

They are fundamentally different:

```
A = [1  2  3]
    [4  5  6]     shape (2, 3)

transpose(A) = [1  4]     permute axes [1, 0]
               [2  5]     data is reordered
               [3  6]

reshape(A, (3, 2)) = [1  2]     same memory layout,
                      [3  4]     different shape
                      [5  6]
```

Transpose of (2×3) gives [[1,4],[2,5],[3,6]]. Reshape to (3×2) gives [[1,2],[3,4],[5,6]]. Completely different results. Confusing them is a common source of bugs.

**Rule of thumb**: if you want to "move data around" (e.g., swap height and width), use permute/transpose. If you want to "reinterpret the grid" (e.g., flatten 2D to 1D), use reshape.

---

## Contiguous memory and performance

After a transpose or permute, the data may no longer be contiguous in memory (the strides change). Many operations require contiguous tensors. In PyTorch, you'd call `.contiguous()` after a permute if needed. This performs an actual copy — which is why unnecessary transposes cost time.

---

## Summary of operations

| Operation | What it does | Moves data? | Cost |
|-----------|-------------|-------------|------|
| Transpose | Swap two axes | Logical (stride change) | Free |
| Reshape | Reinterpret shape, same total elements | No | Free |
| Permute | Reorder arbitrary axes | Logical (stride change) | Free (but may need `.contiguous()` copy later) |

---

Previous: [[06 - The Outer Product]] | Next: [[08 - Broadcasting]]
