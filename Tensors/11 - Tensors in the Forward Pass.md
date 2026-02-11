# Tensors in the Forward Pass

Every layer in a neural network is a tensor operation. This lesson traces a complete forward pass through a 2-layer network, showing exactly which operations from [[01 - What is a Tensor|Part 1]] appear where.

---

## A 2-layer network

Architecture: 3 inputs → 4 hidden (ReLU) → 2 outputs (softmax).

Weights:
- **W₁**: (4×3), **b₁**: (4,) — first layer
- **W₂**: (2×4), **b₂**: (2,) — second layer

Input batch **X**: (B×3) — B examples, 3 features each.

---

## Layer 1: Linear + ReLU

### Step 1 — Linear transformation

```
Z₁ = X · W₁ᵀ + b₁     (B×4) = (B×3) · (3×4) + (4,)
```

This is a [[05 - Matrix-Matrix Multiplication|matrix multiply]] (GEMM) followed by a [[08 - Broadcasting|broadcasted]] bias addition.

Worked example with B=2:

```
X = [1  0  2]    W₁ = [ 1  0  1]    b₁ = [0.1]
    [0  1  1]         [-1  2  0]         [0.2]
                      [ 0  1  1]         [0.3]
                      [ 1  1 -1]         [0.4]

W₁ᵀ = [ 1 -1  0  1]
      [ 0  2  1  1]
      [ 1  0  1 -1]

X · W₁ᵀ:
  row 0: [1·1+0·0+2·1, 1·(-1)+0·2+2·0, 1·0+0·1+2·1, 1·1+0·1+2·(-1)] = [3, -1, 2, -1]
  row 1: [0·1+1·0+1·1, 0·(-1)+1·2+1·0, 0·0+1·1+1·1, 0·1+1·1+1·(-1)] = [1, 2, 2, 0]

Z₁ = [3  -1  2  -1] + [0.1  0.2  0.3  0.4] = [3.1  -0.8  2.3  -0.6]
     [1   2  2   0]                             [1.1   2.2  2.3   0.4]
```

### Step 2 — ReLU activation

```
H₁ = ReLU(Z₁)       H₁_bi = max(0, Z₁_bi)       (B×4)
```

This is a [[02 - Element-wise Operations|element-wise operation]] — no contraction, no shape change.

```
H₁ = [3.1   0   2.3   0 ]    (negatives clipped to 0)
     [1.1  2.2  2.3  0.4]
```

---

## Layer 2: Linear + Softmax

### Step 3 — Linear transformation

```
Z₂ = H₁ · W₂ᵀ + b₂     (B×2) = (B×4) · (4×2) + (2,)
```

Another GEMM + broadcast add. Same structure as Layer 1.

### Step 4 — Softmax

```
P_bi = exp(Z₂_bi) / Σ_j exp(Z₂_bj)       (B×2)
```

Softmax has two sub-operations:
1. **Element-wise exp**: apply exp to every element — [[02 - Element-wise Operations|element-wise]]
2. **Normalization**: divide each element by the sum along axis 1 — a reduction (sum) followed by [[08 - Broadcasting|broadcasted]] division

The sum along axis 1 produces a (B, 1) tensor, which broadcasts back to (B, 2) for the division.

---

## Full forward pass — operation summary

| Step | Operation | Tensor op | Shape |
|------|-----------|-----------|-------|
| 1a | Z₁ = X·W₁ᵀ | GEMM | (B×4) = (B×3)·(3×4) |
| 1b | Z₁ += b₁ | Broadcast add | (B×4) + (4,) |
| 1c | H₁ = ReLU(Z₁) | Element-wise | (B×4) |
| 2a | Z₂ = H₁·W₂ᵀ | GEMM | (B×2) = (B×4)·(4×2) |
| 2b | Z₂ += b₂ | Broadcast add | (B×2) + (2,) |
| 2c | P = softmax(Z₂) | Element-wise + reduce | (B×2) |

The entire forward pass is: two GEMMs, two broadcast additions, one element-wise ReLU, and one softmax. No loops over examples, no special-case logic.

---

## Other common layers as tensor ops

### Embedding lookup

```
E = embedding_table[token_ids]     (B×S×d)
```

Select rows from a (V×d) matrix by index. Equivalent to multiplying by a one-hot matrix — a [[04 - Matrix-Vector Multiplication|matrix-vector product]] where the vector is one-hot, which just selects a column (see the "column view").

### Layer normalization

```
μ = mean(X, axis=-1)                  (B, 1) — reduction
σ = std(X, axis=-1)                   (B, 1) — reduction
X_norm = (X - μ) / σ                  (B, n) — broadcast sub and div
Y = γ ⊙ X_norm + β                   (B, n) — Hadamard + broadcast add
```

Two reductions, two broadcast operations, one [[02 - Element-wise Operations|Hadamard product]].

### Dropout

```
mask ~ Bernoulli(p), shape (B, n)
Y = X ⊙ mask / (1-p)                  (B, n) — Hadamard
```

Element-wise multiply with a random binary mask, then scale.

---

## Key insight

Every layer follows the same pattern:
1. A **linear operation** (GEMM) that mixes features via weighted sums
2. A **non-linear operation** (activation, normalization) that acts element-wise or with simple reductions

The linear part does the heavy computation. The non-linear part is cheap but essential — without it, the whole network collapses to a single [[05 - Matrix-Matrix Multiplication|matrix multiply]].

---

Previous: [[10 - Tensor Contraction and Einsum]] | Next: [[12 - Tensors in Backpropagation]]
