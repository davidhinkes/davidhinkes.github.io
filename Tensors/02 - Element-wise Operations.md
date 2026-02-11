# Element-wise Operations

Element-wise operations apply independently to each corresponding pair of elements. Both inputs must have the same shape (or be [[08 - Broadcasting|broadcastable]] to the same shape), and the output has that same shape.

---

## Addition and subtraction

```
C = A + B       C_ij = A_ij + B_ij
```

Example with shape (2, 3):

```
A = [1  2  3]    B = [10  20  30]    A + B = [11  22  33]
    [4  5  6]        [40  50  60]            [44  55  66]
```

Both inputs must match in shape. `(2×3) + (2×3) → (2×3)`. You cannot add a (2×3) to a (3×2) — the shapes don't align.

---

## Scalar multiplication

Multiply every element by a single number:

```
B = αA          B_ij = α · A_ij
```

```
3 × [1  2] = [3   6]
    [3  4]   [9  12]
```

Shape is unchanged: scalar × (m×n) → (m×n).

---

## Hadamard product (⊙)

Element-wise multiplication. Same shape in, same shape out.

```
C = A ⊙ B       C_ij = A_ij · B_ij
```

```
[1  2  3]     [10  20  30]     [10   40   90]
[4  5  6]  ⊙  [40  50  60]  =  [160  250  360]
```

Shape: `(2×3) ⊙ (2×3) → (2×3)`.

### Why does this matter?

The Hadamard product appears constantly in deep learning:

- **Activation derivatives**: `dLdX_i = dLdY_i · σ'(x_i)` is a Hadamard product between the upstream gradient and the element-wise derivative
- **Masking**: dropout multiplies activations by a binary mask — that's a Hadamard product
- **Gating**: LSTMs and GRUs use element-wise products to gate information flow

---

## The critical distinction

This is the source of most confusion:

| Operation | Symbol | What happens | Shape |
|-----------|--------|--------------|-------|
| Hadamard product | A ⊙ B | Multiply matching elements | (m×n) ⊙ (m×n) → (m×n) |
| Matrix multiplication | A·B | Dot products of rows with columns | (m×k)·(k×n) → (m×n) |

They are **completely different operations**. The Hadamard product requires the same shape and produces the same shape. Matrix multiplication requires the inner dimensions to match and *contracts* (sums over) that shared dimension — see [[05 - Matrix-Matrix Multiplication]].

Concrete example showing they give different results:

```
A = [1  2]    B = [5  6]
    [3  4]        [7  8]

Hadamard:  A ⊙ B = [1·5   2·6] = [ 5  12]
                    [3·7   4·8]   [21  32]

Matrix:    A · B = [1·5+2·7   1·6+2·8] = [19  22]
                   [3·5+4·7   3·6+4·8]   [43  50]
```

Different operations, different results. When a paper or framework says "multiply," you need to know which one they mean.

---

## Element-wise with scalars vs. vectors vs. matrices

Sometimes one operand is smaller and gets "stretched" to match. This is called [[08 - Broadcasting]]:

```
A + b:   (m×n) + (n,)  →  add b to every row of A
A · α:   (m×n) · ()    →  multiply every element by α
```

We'll cover the full rules in [[08 - Broadcasting]]. For now, just know that element-wise operations can work on mismatched shapes under specific, predictable rules.

---

Previous: [[01 - What is a Tensor]] | Next: [[03 - The Dot Product]]
