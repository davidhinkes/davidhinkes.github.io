# The Outer Product

The outer product is the *opposite* of the [dot product](03-The-Dot-Product). Where the dot product takes two vectors and produces a scalar, the outer product takes two vectors and produces a matrix.

---

## Definition

```
C = a ⊗ b       C_ij = a_i · b_j       (m×n) = (m,) ⊗ (n,)
```

Every element of a is multiplied with every element of b. No summation — the dimensions are kept, not contracted.

---

## Worked example

```
a = [1, 2, 3]    b = [4, 5]

a ⊗ b = [1·4  1·5] = [4   5]
        [2·4  2·5]   [8  10]
        [3·4  3·5]   [12  15]
```

Shape: (3,) ⊗ (2,) → (3×2). The output has one row per element of a and one column per element of b.

---

## Comparison: dot product vs. outer product

| | Dot product | Outer product |
|---|---|---|
| Symbol | a · b | a ⊗ b |
| Inputs | (n,) and (n,) — same length | (m,) and (n,) — any lengths |
| Output | scalar () | matrix (m×n) |
| Shared index | Summed over | None — all indices kept |
| Index notation | `Σ_i a_i · b_i` | `a_i · b_j` (no Σ) |

The dot product **contracts** a dimension (sums over it). The outer product **expands** dimensions (keeps all of them).

---

## The outer product as matrix multiplication

The outer product is actually a special case of [matrix multiplication](05-Matrix-Matrix-Multiplication). Treat a as a column vector (m×1) and b as a row vector (1×n):

```
a ⊗ b = a · bᵀ       (m×n) = (m×1) · (1×n)
```

The "shared" dimension is 1, so the sum `Σ_k` has only one term — no real summation happens.

---

## Rank-1 matrices

The output of an outer product is always a **rank-1 matrix**: every row is a scaled copy of b, and every column is a scaled copy of a.

```
a ⊗ b = [4   5]    ← 1 × [4, 5]
        [8  10]    ← 2 × [4, 5]
        [12  15]   ← 3 × [4, 5]
```

You can tell a matrix is rank-1 if every row is proportional to every other row.

---

## Why the outer product matters in deep learning

The outer product is central to **gradient computation for weight matrices**.

For a single-example dense layer `y = W·x`, the weight gradient is:

```
∂L/∂W_ij = dLdY_i · x_j = (dLdY ⊗ x)_ij
```

That's an outer product. Each training example produces one outer product, and the weight gradient is their sum:

```
dLdW = Σ_b  dLdY_b ⊗ x_b
```

This sum of outer products *is* a matrix multiplication in disguise:

```
dLdW = dLdYᵀ · X       (m×n) = (m×B) · (B×n)
```

where rows of **X** are the training examples and columns of **dLdYᵀ** are the per-example gradients. The sum over B (the batch) happens inside the matrix multiply — see [09 - Batch Dimensions](09-Batch-Dimensions) for more.

---

## The full product family so far

| Operation | Input | Output | Index form | Contracts? |
|-----------|-------|--------|------------|------------|
| Hadamard ⊙ | (m×n), (m×n) | (m×n) | `A_ij · B_ij` | No |
| Dot product | (n,), (n,) | () | `Σ_i a_i · b_i` | Yes (i) |
| Outer product ⊗ | (m,), (n,) | (m×n) | `a_i · b_j` | No |
| Mat-vec | (m×n), (n,) | (m,) | `Σ_j W_ij · x_j` | Yes (j) |
| Mat-mat | (m×k), (k×n) | (m×n) | `Σ_k A_ik · B_kj` | Yes (k) |

The unifying pattern: multiply corresponding elements, then either sum over an index (contraction) or keep it. [10 - Tensor Contraction and Einsum](10-Tensor-Contraction-and-Einsum) will make this explicit.

---

Previous: [05 - Matrix-Matrix Multiplication](05-Matrix-Matrix-Multiplication) | Next: [07 - Transpose and Reshaping](07-Transpose-and-Reshaping)
