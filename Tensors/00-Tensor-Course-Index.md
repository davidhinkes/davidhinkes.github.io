# Tensors and Deep Neural Networks

A course on tensor operations and how they compose to build neural networks. Each lesson is self-contained but builds on earlier material.

---

## Part 1: Foundations

1. [01 - What is a Tensor](01-What-is-a-Tensor) — Scalars, vectors, matrices, higher-order tensors. Rank, shape, axes.
2. [02 - Element-wise Operations](02-Element-wise-Operations) — Addition, scalar multiply, Hadamard product (⊙). Why element-wise ≠ matrix multiply.
3. [03 - The Dot Product](03-The-Dot-Product) — Inner product of vectors. Geometric meaning: projection and cosine similarity.
4. [04 - Matrix-Vector Multiplication](04-Matrix-Vector-Multiplication) — m dot products in parallel. Linear transformations. GEMV.
5. [05 - Matrix-Matrix Multiplication](05-Matrix-Matrix-Multiplication) — Composing maps. Row-by-column dot products. GEMM.
6. [06 - The Outer Product](06-The-Outer-Product) — Vector ⊗ vector → matrix. Rank-1 matrices. Role in gradient computation.
7. [07 - Transpose and Reshaping](07-Transpose-and-Reshaping) — Transpose, reshape, permute. Why Wᵀ appears in backprop.

## Part 2: Scaling Up

8. [08 - Broadcasting](08-Broadcasting) — Automatic shape alignment. Rules, examples, common pitfalls.
9. [09 - Batch Dimensions](09-Batch-Dimensions) — 3D+ tensors, batch-first convention. GEMV → GEMM.
10. [10 - Tensor Contraction and Einsum](10-Tensor-Contraction-and-Einsum) — The unifying framework. Every product as index contraction.

## Part 3: Application to Deep Learning

11. [11 - Tensors in the Forward Pass](11-Tensors-in-the-Forward-Pass) — Dense layers, ReLU, softmax as tensor ops. Full worked example.
12. [12 - Tensors in Backpropagation](12-Tensors-in-Backpropagation) — Chain rule over tensors. Backward is "forward with transposes."
13. [13 - Convolutions as Tensor Operations](13-Convolutions-as-Tensor-Operations) — im2col trick. Convolutions are matrix multiplies.
14. [14 - Attention is All Tensor Ops](14-Attention-is-All-Tensor-Ops) — QKV, scaled dot-product attention, multi-head attention.

---

## Product cheat sheet

All the "multiplications" in one place:

| Name | Symbol | Input shapes | Output shape | Einsum | What it does |
|------|--------|-------------|--------------|--------|-------------|
| Scalar multiply | αA | (), (m×n) | (m×n) | — | Scale every element |
| Hadamard product | A ⊙ B | (m×n), (m×n) | (m×n) | `ij, ij -> ij` | Multiply matching elements |
| Dot product | a · b | (n,), (n,) | () | `i, i ->` | Multiply and sum → scalar |
| Outer product | a ⊗ b | (m,), (n,) | (m×n) | `i, j -> ij` | All pairs, no sum → matrix |
| Mat-vec multiply | W·x | (m×n), (n,) | (m,) | `ij, j -> i` | m dot products |
| Mat-mat multiply | A·B | (m×k), (k×n) | (m×n) | `ik, kj -> ij` | Compose linear maps |
| Batched mat-mat | bmm(A,B) | (B×m×k), (B×k×n) | (B×m×n) | `bik, bkj -> bij` | Independent GEMM per batch |

**The pattern**: multiply elements, then either sum over an index (contraction → dimension disappears) or keep it (→ dimension stays). Every product is a variation on this theme.

---

## Related notes

- [batch-math](batch-math) — Batch input/output migration for a Go neural network library
