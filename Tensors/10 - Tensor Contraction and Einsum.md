# Tensor Contraction and Einsum

Every product you've seen so far — dot product, matrix multiply, outer product, batched multiply — is a special case of one general operation: **tensor contraction**. Einsum notation makes this explicit.

---

## The general pattern

Every product in this course follows the same recipe:

1. **Label each axis** with an index letter
2. **Multiply** elements that share position
3. **Sum over** (contract) any index that appears in both inputs but not in the output

The only thing that changes between products is *which indices get summed*.

---

## Einsum notation

Einsum (Einstein summation) makes the recipe explicit in a string:

```
output = einsum("input_indices -> output_indices", tensors...)
```

The rules:
- Each input tensor gets a comma-separated index string
- Indices that appear in both inputs but **not** after the `->` are summed over
- Indices that appear after the `->` are kept in the output

---

## Every product as einsum

### Dot product: (n,) · (n,) → ()

```
einsum("i, i ->", a, b)     =  Σ_i a_i · b_i
```

Index `i` appears in both inputs, not in output → contracted.

### Outer product: (m,) ⊗ (n,) → (m×n)

```
einsum("i, j -> ij", a, b)  =  a_i · b_j
```

No shared indices → nothing contracted. Both kept.

### Hadamard product: (m×n) ⊙ (m×n) → (m×n)

```
einsum("ij, ij -> ij", A, B) =  A_ij · B_ij
```

Indices `i,j` appear in both inputs *and* in the output → kept, not contracted.

### Matrix-vector: (m×n) · (n,) → (m,)

```
einsum("ij, j -> i", W, x)  =  Σ_j W_ij · x_j
```

Index `j` in both inputs, not in output → contracted. Index `i` kept.

### Matrix-matrix: (m×k) · (k×n) → (m×n)

```
einsum("ik, kj -> ij", A, B) =  Σ_k A_ik · B_kj
```

Index `k` contracted. Indices `i, j` kept.

### Batched matrix multiply: (B×m×k) · (B×k×n) → (B×m×n)

```
einsum("bik, bkj -> bij", A, B) =  Σ_k A_bik · B_bkj
```

Index `k` contracted. Indices `b, i, j` kept. The batch index `b` is just another kept index — nothing special about it.

---

## Summary table

| Operation | Einsum | Contracted | Kept |
|-----------|--------|-----------|------|
| Dot product | `i, i ->` | i | (none) |
| Outer product | `i, j -> ij` | (none) | i, j |
| Hadamard | `ij, ij -> ij` | (none) | i, j |
| Mat-vec | `ij, j -> i` | j | i |
| Mat-mat | `ik, kj -> ij` | k | i, j |
| BMM | `bik, bkj -> bij` | k | b, i, j |
| Trace | `ii ->` | i | (none) |
| Transpose | `ij -> ji` | (none) | i, j |
| Batch outer | `bi, bj -> bij` | (none) | b, i, j |
| Sum of outer products | `bi, bj -> ij` | b | i, j |

The last row is exactly the weight gradient: `dLdW = einsum("bi, bj -> ij", dLdY, X)` sums B outer products by contracting the batch index.

---

## Einsum as the Rosetta Stone

When you read a paper and see a new product-like operation, translate it to einsum:

**Multi-head attention (QKᵀ):**

```
scores = einsum("bhsd, bhtd -> bhst", Q, K)
```

b=batch, h=heads, s=query position, t=key position, d=head dimension. The head dimension d is contracted — each score is a dot product over d. Batch and heads are kept — independent attention per head per example.

**Bilinear layer (xᵀWy):**

```
out = einsum("bi, ijk, bk -> bj", x, W, y)
```

Both i (from x) and k (from y) are contracted against the weight tensor **W**. Only b and j survive.

Once you can read einsum, you can read any tensor operation.

---

## Practical notes

In PyTorch:
```python
torch.einsum("ik, kj -> ij", A, B)   # same as A @ B
torch.einsum("bi, bj -> ij", dLdY, X) # weight gradient
```

In NumPy:
```python
np.einsum("ik, kj -> ij", A, B)
```

Einsum implementations are optimized — they find efficient contraction orderings, fuse operations, and call into BLAS when the pattern matches GEMM. For simple cases like matrix multiply, `einsum` and `@` have the same performance. For complex multi-tensor contractions, einsum may find a better ordering than chaining manual operations.

---

Previous: [[09 - Batch Dimensions]] | Next: [[11 - Tensors in the Forward Pass]]
