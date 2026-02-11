# Attention is All Tensor Ops

The attention mechanism — the core of transformers — is built entirely from operations covered in this course: linear projections ([[05 - Matrix-Matrix Multiplication|matrix multiplies]]), [[03 - The Dot Product|dot products]], [[02 - Element-wise Operations|element-wise operations]], and a softmax. Nothing new. Once you see the tensor shapes, it's straightforward.

---

## Setup: the input

A batch of sequences, where each token is a d_model-dimensional embedding:

```
X: (B, S, d_model)     B = batch size, S = sequence length
```

For example: B=2 sentences, S=10 tokens each, d_model=512.

---

## Step 1: Project Q, K, V

Three separate linear projections (no bias for clarity):

```
Q = X · W_Q       (B, S, d_model) · (d_model, d_k)   → (B, S, d_k)
K = X · W_K       (B, S, d_model) · (d_model, d_k)   → (B, S, d_k)
V = X · W_V       (B, S, d_model) · (d_model, d_v)   → (B, S, d_v)
```

Each is a [[09 - Batch Dimensions|batched]] matrix multiply: the (d_model, d_k) weight matrix is applied independently to every sequence position in every batch. In [[10 - Tensor Contraction and Einsum|einsum]]:

```
Q = einsum("bsd, dk -> bsk", X, W_Q)
```

The "d" (d_model) index is contracted. Batch and sequence indices are kept.

---

## Step 2: Compute attention scores

How much should token s attend to token t? Measure it with a [[03 - The Dot Product|dot product]] between the query at position s and the key at position t:

```
score_st = Q_s · K_t / √d_k
```

For all positions at once, this is a [[09 - Batch Dimensions|batched matrix multiply]]:

```
scores = Q · Kᵀ / √d_k     (B, S, S) = (B, S, d_k) · (B, d_k, S) / √d_k
```

In einsum:

```
scores = einsum("bsd, btd -> bst", Q, K) / sqrt(d_k)
```

The head dimension d_k is contracted — each score is a dot product over d_k. The result is an (S×S) attention matrix per batch: score at (s, t) measures relevance of key t to query s.

The `/√d_k` is a scalar that scales every element — an [[02 - Element-wise Operations|element-wise]] operation. Without it, dot products grow with d_k, pushing softmax into regions with vanishing gradients.

---

## Step 3: Softmax → attention weights

```
A = softmax(scores, axis=-1)     (B, S, S)
```

Applied independently to each row (each query position). After softmax, each row sums to 1 — the weights over keys form a probability distribution.

### Causal masking

In autoregressive models (GPT-style), token s can only attend to tokens t ≤ s. Before softmax, set future positions to -∞:

```
mask: upper-triangular matrix of -∞
scores = scores + mask     (B, S, S) + (1, S, S)   — broadcast
```

The [[08 - Broadcasting|broadcast]] adds -∞ to future positions. After softmax, exp(-∞) = 0 — future tokens get zero weight.

---

## Step 4: Weighted sum of values

```
output = A · V     (B, S, d_v) = (B, S, S) · (B, S, d_v)
```

Another batched matrix multiply. Each output token is a weighted combination of all value vectors, with weights from the attention matrix.

In einsum:

```
output = einsum("bst, btv -> bsv", A, V)
```

The key position index t is contracted — summing over all positions that each query attends to.

---

## Full single-head attention

```
Q = X · W_Q                              (B, S, d_k)     — GEMM
K = X · W_K                              (B, S, d_k)     — GEMM
V = X · W_V                              (B, S, d_v)     — GEMM
scores = Q · Kᵀ / √d_k                  (B, S, S)       — BMM + scalar
A = softmax(scores)                      (B, S, S)       — element-wise + reduce
output = A · V                           (B, S, d_v)     — BMM
```

Six operations. Four are matrix multiplies (or batched variants). One is element-wise + reduction. One is scalar multiplication.

---

## Multi-head attention

Instead of one large attention, split into H heads with smaller dimensions:

```
d_k = d_v = d_model / H
```

### Reshape into heads

After projection, reshape and permute to create a head dimension:

```
Q: (B, S, d_model) → reshape → (B, S, H, d_k) → permute → (B, H, S, d_k)
K: (B, S, d_model) → reshape → (B, S, H, d_k) → permute → (B, H, S, d_k)
V: (B, S, d_model) → reshape → (B, S, H, d_v) → permute → (B, H, S, d_v)
```

The [[07 - Transpose and Reshaping|reshape]] splits d_model into (H, d_k). The permute moves the head axis to position 1 so it acts as a batch dimension.

### Attention per head

Now both B (batch) and H (heads) are "batch dimensions":

```
scores = einsum("bhsd, bhtd -> bhst", Q, K) / √d_k     (B, H, S, S)
A = softmax(scores, axis=-1)                              (B, H, S, S)
output = einsum("bhst, bhtv -> bhsv", A, V)              (B, H, S, d_v)
```

Each head independently computes attention. The operations are the same as single-head — just with an extra batch-like dimension H.

### Concatenate and project

```
output: (B, H, S, d_v) → permute → (B, S, H, d_v) → reshape → (B, S, H·d_v)

final = output_concat · W_O     (B, S, d_model) = (B, S, d_model) · (d_model, d_model)
```

The reshape concatenates all head outputs. The final projection is one more GEMM.

---

## Operation inventory for multi-head attention

| Step | Operation type | Count |
|------|---------------|-------|
| QKV projection | GEMM | 3 (or 1 fused) |
| Reshape + permute | [[07 - Transpose and Reshaping\|Shape manipulation]] | 3 (free) |
| QKᵀ scores | BMM | 1 |
| Scale by 1/√d_k | Element-wise | 1 |
| Causal mask | Broadcast add | 1 (optional) |
| Softmax | Element-wise + reduce | 1 |
| Attention × V | BMM | 1 |
| Concat + output projection | Reshape + GEMM | 1 |

Total: 5 matrix multiplies, 3 element-wise/reductions, a few free reshapes. Every operation is from lessons 2-10 of this course.

---

## Why attention scales quadratically

The attention matrix has shape (S, S) — one score for every query-key pair. Computing it costs O(S²·d_k) and storing it costs O(S²) per head per batch. This is why long-context models are expensive and why techniques like FlashAttention (which fuses operations to avoid materializing the full S×S matrix) matter.

---

Previous: [[13 - Convolutions as Tensor Operations]] | Course index: [[00 - Tensor Course Index]]
