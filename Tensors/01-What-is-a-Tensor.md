# What is a Tensor

A tensor is a multi-dimensional array of numbers. That's it. The word sounds intimidating, but you already know the first three ranks:

| Rank | Name | Shape | Example |
|------|------|-------|---------|
| 0 | Scalar | `()` | Temperature: `22.5` |
| 1 | Vector | `(n,)` | RGB pixel: `[128, 64, 255]` |
| 2 | Matrix | `(m, n)` | Grayscale image: 28×28 grid of intensities |
| 3 | 3D tensor | `(d₁, d₂, d₃)` | Color image: 3×28×28 (channels × height × width) |
| 4 | 4D tensor | `(d₁, d₂, d₃, d₄)` | Batch of color images: 32×3×28×28 |

The pattern continues to any rank.

---

## Rank, shape, and axes

**Rank** (also called *order* or *ndim*) is the number of indices you need to pick out a single element.

- Scalar `s`: zero indices → rank 0
- Vector `v`: one index `v_i` → rank 1
- Matrix **M**: two indices `M_ij` → rank 2
- 3D tensor **T**: three indices `T_ijk` → rank 3

**Shape** is the tuple of sizes along each axis. A (3, 4, 5) tensor has 3 "slabs," each containing a 4×5 matrix — totaling 3 × 4 × 5 = 60 elements.

**Axes** are the individual dimensions. For a matrix with shape (m, n):
- Axis 0 runs along rows (m elements)
- Axis 1 runs along columns (n elements)

```
Matrix A, shape (3, 4):

        axis 1 →
        j=0  j=1  j=2  j=3
axis 0  ┌────┬────┬────┬────┐
  i=0   │ a₀₀│ a₀₁│ a₀₂│ a₀₃│
  ↓     ├────┼────┼────┼────┤
  i=1   │ a₁₀│ a₁₁│ a₁₂│ a₁₃│
        ├────┼────┼────┼────┤
  i=2   │ a₂₀│ a₂₁│ a₂₂│ a₂₃│
        └────┴────┴────┴────┘
```

---

## Why tensors in deep learning?

Neural networks process data in batches. A single grayscale image is a matrix (H×W). A batch of B images is a 3D tensor (B×H×W). A batch of color images is 4D (B×C×H×W). The weights of a convolutional layer form a 4D tensor. Attention scores in a transformer are 4D (batch × heads × seq_len × seq_len).

Every operation in a neural network — forward pass, loss computation, backpropagation — is a tensor operation. Learning what these operations *are* and how their dimensions interact is the subject of this course.

---

## Notation used in this course

| Symbol | Meaning |
|--------|---------|
| `x`, `y` | Vectors (lowercase, plain) |
| **W**, **X**, **Y** | Matrices and higher tensors (bold uppercase) |
| `x_i` | Element i of vector x |
| `W_ij` | Element at row i, column j of **W** |
| `T_ijk` | Element of a rank-3 tensor |
| `(m×n)` | Shape annotation, e.g. **W** is (m×n) |
| `Σ_j` | Sum over index j |

Dimensions are always annotated on operations:

```
y = W·x       (m,) = (m×n)·(n,)
```

This tells you: **W** has shape (m×n), **x** has shape (n,), and the result **y** has shape (m,). The shared dimension n is summed over ("contracted") — this is the core mechanic you'll see in every product.

---

Next: [02 - Element-wise Operations](02-Element-wise-Operations)
