# Tensors in Backpropagation

Backpropagation is the chain rule applied to tensor operations. Every forward operation has a corresponding backward rule that computes gradients with respect to its inputs. The backward pass is "just the forward pass in reverse, with transposes."

---

## The chain rule for tensors

For a scalar loss L that depends on a tensor through a chain of operations:

```
X → Z → H → Y → L
```

The gradient of L with respect to any intermediate tensor is computed by working backwards:

```
dLdY → dLdH → dLdZ → dLdX
```

At each step, the chain rule says:

```
∂L/∂(input)_ij = Σ_kl  (∂L/∂(output)_kl) · (∂(output)_kl / ∂(input)_ij)
```

This looks complicated in general, but each specific layer type simplifies to familiar tensor operations.

---

## Backward rules for each operation

### Dense layer: Y = X·Wᵀ + b

Forward:
```
Y_bi = Σ_j X_bj · W_ij + b_i       (B×m) = (B×n)·(n×m) + (m,)
```

Backward (given dLdY of shape (B×m)):

```
dLdX = dLdY · W       (B×n) = (B×m) · (m×n)
dLdW = dLdYᵀ · X      (m×n) = (m×B) · (B×n)
dLdB = Σ_b dLdY_b     (m,) — column sum over batch
```

**dLdX**: The [[07 - Transpose and Reshaping|transpose]] of W appears because the backward pass reverses the forward map. This is a GEMM.

**dLdW**: This is the sum of B [[06 - The Outer Product|outer products]], computed as a single GEMM. The batch dimension becomes the contracted dimension.

**dLdB**: A reduction (sum) over the batch axis. Each example contributes its gradient, and they're added together.

### Derivation of dLdX

Start from component form:

```
Y_bi = Σ_j X_bj · W_ij
```

Apply the chain rule — X_bj affects Y_bi for all i (same batch row, all output features):

```
∂L/∂X_bj = Σ_i (∂L/∂Y_bi) · (∂Y_bi/∂X_bj)
          = Σ_i dLdY_bi · W_ij
```

The sum over i with factors `dLdY_bi` and `W_ij` is the definition of the matrix product `(dLdY · W)_bj`:

```
dLdX = dLdY · W       (B×n) = (B×m) · (m×n)    ✓
```

### Derivation of dLdW

W_ij affects Y_bi for all b (all batch examples):

```
∂L/∂W_ij = Σ_b (∂L/∂Y_bi) · (∂Y_bi/∂W_ij)
          = Σ_b dLdY_bi · X_bj
```

The sum over b with factors `dLdY_bi` and `X_bj` is `(dLdYᵀ · X)_ij`:

```
dLdW = dLdYᵀ · X      (m×n) = (m×B) · (B×n)    ✓
```

---

### ReLU: H = max(0, Z)

Forward:
```
H_bi = max(0, Z_bi)       element-wise
```

Backward:
```
dLdZ_bi = dLdH_bi · 𝟙(Z_bi > 0)       element-wise
```

where 𝟙(Z_bi > 0) is 1 if Z_bi was positive, 0 otherwise. This is a [[02 - Element-wise Operations|Hadamard product]] between the upstream gradient and the derivative mask.

The derivative of ReLU is a step function: 1 where the input was positive, 0 where it was negative. The gradient flows through unchanged where the neuron "fired," and is zeroed out where it didn't.

---

### Softmax: P_bi = exp(Z_bi) / Σ_j exp(Z_bj)

This one is more complex because each output depends on *all* inputs in its row.

```
∂P_bi/∂Z_bk = P_bi · (δ_ik - P_bk)
```

where δ_ik is 1 if i=k, 0 otherwise. The Jacobian is a full (m×m) matrix per example.

The gradient works out to:

```
dLdZ_bi = P_bi · (dLdP_bi - Σ_k dLdP_bk · P_bk)
```

In practice, when softmax is paired with cross-entropy loss, the combined gradient simplifies to:

```
dLdZ_bi = P_bi - y_bi       (B×m)
```

where y is the one-hot target. This is why softmax + cross-entropy is almost always implemented as a fused operation.

---

## The backward pass is symmetric to the forward pass

| Forward | Backward | Operation |
|---------|----------|-----------|
| Y = X·Wᵀ | dLdX = dLdY·W | GEMM (W instead of Wᵀ) |
| Y = X·Wᵀ | dLdW = dLdYᵀ·X | GEMM (contracting batch) |
| Y = Z + b | dLdB = sum(dLdY, axis=0) | Reduction |
| H = ReLU(Z) | dLdZ = dLdH ⊙ mask | Hadamard |

The backward pass uses the same BLAS operations as the forward pass, with different operands and transposes. This is why forward and backward have roughly the same computational cost (backward is ~2× forward because it computes both dLdX and dLdW).

---

## Full backward pass for the 2-layer network

Using the same architecture from [[11 - Tensors in the Forward Pass]]:

```
Forward:  X → Z₁ → H₁ → Z₂ → P → Loss

Backward (in reverse):
  dLdZ₂ = P - Y_target           (B×2) — fused softmax+CE gradient
  dLdW₂ = dLdZ₂ᵀ · H₁           (2×4) = (2×B) · (B×4)
  dLdB₂ = sum(dLdZ₂, axis=0)    (2,)
  dLdH₁ = dLdZ₂ · W₂            (B×4) = (B×2) · (2×4)
  dLdZ₁ = dLdH₁ ⊙ (Z₁ > 0)     (B×4) — ReLU backward
  dLdW₁ = dLdZ₁ᵀ · X            (4×3) = (4×B) · (B×3)
  dLdB₁ = sum(dLdZ₁, axis=0)    (4,)
```

Seven operations: 3 GEMMs, 2 reductions, 1 Hadamard, 1 subtraction. All are tensor operations you've already seen.

---

Previous: [[11 - Tensors in the Forward Pass]] | Next: [[13 - Convolutions as Tensor Operations]]
