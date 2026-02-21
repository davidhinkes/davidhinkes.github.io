# Matrix-Vector Multiplication

Matrix-vector multiplication is *m* [dot products](03-The-Dot-Product) computed in parallel. Each row of the matrix is dotted with the input vector to produce one element of the output.

---

## Definition

```
y = W·x        y_i = Σ_j  W_ij · x_j       (m,) = (m×n) · (n,)
```

The shared dimension n is contracted. The output has one element per row of **W**.

---

## Worked example

```
W = [1  2  3]    x = [2]
    [4  5  6]        [1]
                     [3]

W is (2×3), x is (3,) → y is (2,)

y₀ = 1·2 + 2·1 + 3·3 = 2 + 2 + 9 = 13
y₁ = 4·2 + 5·1 + 6·3 = 8 + 5 + 18 = 31

y = [13, 31]
```

Row 0 of **W** dotted with x → 13. Row 1 of **W** dotted with x → 31.

---

## The dimension rule

```
(m×n) · (n,) → (m,)
  ↑       ↑      ↑
  rows    must    output has
  of W    match   m elements
          length
          of x
```

The inner dimensions must agree. If **W** is (m×n) and x is (k,), you need n = k. The output shape comes from the "outer" dimensions: m.

This is the same rule as matrix-matrix multiplication (see [05 - Matrix-Matrix Multiplication](05-Matrix-Matrix-Multiplication)) with the right operand having one column.

---

## Geometric view: linear transformation

Matrix-vector multiplication is a **linear transformation**. It maps a vector in ℝⁿ to a vector in ℝᵐ.

What does the matrix "do" to the vector?

- **Rotation**: orthogonal matrices rotate without stretching
- **Scaling**: diagonal matrices scale each axis independently
- **Projection**: non-square matrices can change dimensionality

A (2×3) matrix maps 3D vectors to 2D vectors. This is exactly what a neural network layer does: a layer with 3 inputs and 2 outputs has a (2×3) weight matrix.

---

## As a weighted combination of columns

There's an equivalent way to read the same operation. Instead of "row i dots with x," think of it as "the output is a weighted combination of the columns of **W**":

```
y = x₀ · col₀(W) + x₁ · col₁(W) + x₂ · col₂(W)
```

```
y = 2·[1] + 1·[2] + 3·[3] = [2] + [2] + [ 9] = [13]
     [4]    [5]    [6]   [8]   [5]   [18]   [31]
```

Same result, different perspective. The "column view" is useful for understanding embeddings: looking up an embedding vector is just multiplying by a one-hot vector, which selects one column.

---

## In a neural network

A single dense layer with n inputs and m outputs:

```
y = W·x + b        (m,) = (m×n)·(n,) + (m,)
```

That's one matrix-vector multiply followed by a [bias addition](02-Element-wise-Operations). The weight matrix **W** stores m learned direction vectors (the rows), and the output measures how much the input aligns with each one.

This is GEMV (general matrix-vector multiply) — a level-2 BLAS operation. When you process a batch of inputs simultaneously, GEMV becomes GEMM — see [09 - Batch Dimensions](09-Batch-Dimensions).

---

Previous: [03 - The Dot Product](03-The-Dot-Product) | Next: [05 - Matrix-Matrix Multiplication](05-Matrix-Matrix-Multiplication)
