# Batch Dimensions

A single input is a vector or matrix. A batch of inputs is a tensor with one (or more) extra leading axes. This simple idea — "stack examples along a new axis" — is what turns expensive loops into fast parallel operations.

---

## From one example to a batch

A single dense layer processes one example:

```
y = W·x + b       (m,) = (m×n)·(n,) + (m,)
```

To process B examples, stack them into a matrix **X** with shape (B×n), one row per example:

```
Y = X·Wᵀ + b      (B×m) = (B×n)·(n×m) + (m,)
```

Note the transpose: the single-example formula uses `W·x` with x as a column, but the batch formula uses `X·Wᵀ` because each *row* of **X** is one example. The result is equivalent: row b of **Y** equals `W · x_b + b`.

---

## Why batching matters: GEMV → GEMM

Processing B examples one at a time means B separate matrix-vector multiplies (GEMV). Batching them is one matrix-matrix multiply (GEMM).

| | Single example × B | Batch |
|---|---|---|
| Operation | B × GEMV | 1 × GEMM |
| BLAS level | Level-2 | Level-3 |
| Arithmetic intensity | ~1 (memory-bound) | ~n (compute-bound) |
| Loop overhead | B iterations in Python/Go | 0 |

The arithmetic is identical — same number of multiply-adds. But GEMM reuses data across the batch, saturating caches and SIMD pipelines. On a GPU, the B examples can be processed in parallel across cores.

---

## The batch axis convention

Deep learning frameworks use a consistent convention: **axis 0 is the batch dimension**.

| Data type | Single | Batched | Shape |
|-----------|--------|---------|-------|
| Dense input | (n,) | (B, n) | B vectors |
| Grayscale image | (H, W) | (B, H, W) | B images |
| Color image | (C, H, W) | (B, C, H, W) | B images |
| Sequence of embeddings | (S, d) | (B, S, d) | B sequences |
| Attention scores | (S, S) | (B, H, S, S) | B sequences × H heads |

Everything to the right of axis 0 is the "per-example" structure. Everything at axis 0 indexes into the batch.

---

## Batched matrix multiply (BMM)

For 3D tensors, batched matrix multiply applies an independent matrix-matrix multiply along the batch axis:

```
C = bmm(A, B)       C_bij = Σ_k  A_bik · B_bkj

A: (B×m×k)   B: (B×k×n)   C: (B×m×n)
```

Each "slice" `A[b]` is multiplied with `B[b]` independently. The batch dimension B is preserved — it's not contracted.

Example: in multi-head attention, after splitting into heads, you have queries (B×H×S×d) and keys (B×H×S×d). The attention scores are computed with BMM:

```
scores = bmm(Q, Kᵀ)    (B×H×S×S) = (B×H×S×d) · (B×H×d×S)
```

Here both B (batch) and H (heads) are "batch dimensions" — the multiply happens independently over both.

---

## Gradient accumulation across the batch

Weights are shared across all examples. Per-example gradients must be **summed** to get the weight gradient.

For a dense layer `Y = X·Wᵀ`:

```
dLdW = dLdYᵀ · X       (m×n) = (m×B) · (B×n)
```

This single GEMM computes the sum of B [outer products](06-The-Outer-Product):

```
dLdW = Σ_b  dLdY_b ⊗ x_b
```

The batch dimension B is contracted in the multiply — it appears as the shared inner dimension. No explicit loop needed.

For biases:

```
dLdB = Σ_b dLdY_b = 1ᵀ · dLdY       (m,) = (B,)ᵀ · (B×m)
```

A sum over the batch axis — one operation replaces B vector additions.

---

## Memory cost of batching

Batching stores all intermediate activations simultaneously:

```
Single:  one (width,) vector per layer → O(width) memory
Batch:   one (B×width) matrix per layer → O(B × width) memory
```

For a 5-layer network with width 64 and batch size 128: 5 × 128 × 64 × 8 bytes ≈ 320 KB. For large transformers with multi-head attention, the activation memory can be the dominant cost, which is why techniques like gradient checkpointing exist.

---

Previous: [08 - Broadcasting](08-Broadcasting) | Next: [10 - Tensor Contraction and Einsum](10-Tensor-Contraction-and-Einsum)
