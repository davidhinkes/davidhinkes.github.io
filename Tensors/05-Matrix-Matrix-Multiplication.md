# Matrix-Matrix Multiplication

Matrix-matrix multiplication is the workhorse of deep learning. Every dense layer, every attention head, every convolutional layer (under the hood) comes down to this operation.

---

## Definition

```
C = A·B        C_ij = Σ_k  A_ik · B_kj       (m×n) = (m×k) · (k×n)
```

The shared dimension k is contracted (summed over). Each element C_ij is the [dot product](03-The-Dot-Product) of row i of **A** with column j of **B**.

---

## Worked example

```
A = [1  2]    B = [5  6]
    [3  4]        [7  8]

A is (2×2), B is (2×2) → C is (2×2)

C₀₀ = 1·5 + 2·7 = 19    C₀₁ = 1·6 + 2·8 = 22
C₁₀ = 3·5 + 4·7 = 43    C₁₁ = 3·6 + 4·8 = 50

C = [19  22]
    [43  50]
```

---

## The dimension rule

```
(m×k) · (k×n) → (m×n)
    ↑     ↑
    must match
    (contracted)
```

The inner dimensions must agree. The outer dimensions form the output shape.

Non-square example:

```
(3×2) · (2×4) → (3×4)        ✓  inner = 2
(3×2) · (3×4) → error        ✗  2 ≠ 3
```

---

## Three ways to read the same multiply

All three are equivalent — same computation, different groupings.

### 1. Element view (dot products)

Each output element is a dot product:

```
C_ij = row_i(A) · col_j(B)
```

The output matrix contains m×n dot products, each over vectors of length k.

### 2. Column view (matrix-vector products)

Each column of the output is a [matrix-vector product](04-Matrix-Vector-Multiplication):

```
col_j(C) = A · col_j(B)        (m,) = (m×k) · (k,)
```

The matrix **A** transforms each column of **B** independently. If **B** has n columns, you get n separate mat-vec products, packaged into one GEMM call.

### 3. Row view

Each row of the output comes from left-multiplying a row of **A**:

```
row_i(C) = row_i(A) · B        (n,) = (k,) · (k×n)
```

---

## Non-commutativity

Matrix multiplication is **not commutative**: A·B ≠ B·A in general.

```
A·B = [19  22]    but    B·A = [23  34]
      [43  50]                 [31  46]
```

Even the shapes can be incompatible: if A is (3×2) and B is (2×4), then A·B is (3×4) but B·A doesn't exist (4 ≠ 3).

This matters in deep learning: the order of operands in forward vs. backward passes is different, and getting it wrong gives you wrong shapes and wrong gradients.

---

## Associativity

While not commutative, matrix multiplication *is* associative:

```
(A · B) · C = A · (B · C)
```

This means you can choose which pair to multiply first. In a multi-layer network, `W₃ · W₂ · W₁ · x` can be computed as `W₃ · (W₂ · (W₁ · x))` (right-to-left, processing the input through layers) or `(W₃ · W₂ · W₁) · x` (pre-composing all transformations). The result is the same, but computational cost can differ dramatically depending on the shapes.

---

## Composing linear transformations

If **A** maps ℝᵏ → ℝᵐ and **B** maps ℝⁿ → ℝᵏ, then **A·B** maps ℝⁿ → ℝᵐ. Matrix multiplication *composes* linear maps.

A two-layer network without activations:

```
y = W₂ · (W₁ · x) = (W₂ · W₁) · x
```

This collapses into a single matrix. That's why non-linear activations are essential: without them, stacking layers gives you nothing that a single layer can't do.

---

## Computation cost

Multiplying (m×k) · (k×n) requires m·k·n multiply-adds.

This is GEMM (general matrix-matrix multiply) — a level-3 BLAS operation. It has O(mkn) arithmetic on O(mk + kn) data, giving arithmetic intensity ~n. This is why GEMM saturates hardware (CPUs, GPUs) far better than GEMV — the same data gets reused across multiple output elements, enabling cache tiling and SIMD vectorization.

---

Previous: [04 - Matrix-Vector Multiplication](04-Matrix-Vector-Multiplication) | Next: [06 - The Outer Product](06-The-Outer-Product)
