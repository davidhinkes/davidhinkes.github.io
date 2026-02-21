# The Dot Product

The dot product (also called *inner product*) is the simplest operation that **reduces** two vectors to a single scalar. It's the fundamental building block of matrix multiplication, attention, and similarity search.

---

## Definition

Given two vectors of the same length n:

```
a · b = Σ_i  a_i · b_i       (n,) · (n,) → scalar
```

Multiply corresponding elements, then sum:

```
a = [1, 2, 3]    b = [4, 5, 6]

a · b = 1·4 + 2·5 + 3·6 = 4 + 10 + 18 = 32
```

The input is two vectors of shape (n,). The output is a scalar — shape (). The dimension n is **contracted** (summed over and eliminated).

---

## Geometric interpretation

The dot product has a clean geometric meaning:

```
a · b = ||a|| · ||b|| · cos(θ)
```

where `||a||` is the length (norm) of a, and θ is the angle between them.

This tells you:
- **a · b > 0** → vectors point in roughly the same direction (θ < 90°)
- **a · b = 0** → vectors are perpendicular / orthogonal (θ = 90°)
- **a · b < 0** → vectors point in roughly opposite directions (θ > 90°)

The magnitude depends on both the lengths of the vectors and their alignment.

---

## Cosine similarity

If you only care about direction (not magnitude), normalize first:

```
cos_sim(a, b) = (a · b) / (||a|| · ||b||)
```

This gives a value in [-1, 1]:
- `1` = identical direction
- `0` = perpendicular
- `-1` = opposite direction

This is how embedding similarity works in retrieval systems and how attention scores measure query-key alignment before the softmax.

---

## The dot product as projection

The dot product of a with a unit vector û gives the **signed length** of a's projection onto û:

```
proj = a · û
```

```
        a
       /|
      / |
     /  |
    /   |
   /    |
  û─────┘
  |←proj→|
```

This is exactly what a single neuron does: it projects the input onto its weight vector and measures "how much of the input lies in this direction."

---

## Where the dot product appears in deep learning

| Context | Operation | What it means |
|---------|-----------|---------------|
| Single neuron | `w · x + b` | Project input onto weight direction |
| Attention score | `q · k` | How relevant is key k to query q? |
| Cosine similarity | `(a · b) / (‖a‖·‖b‖)` | Normalized alignment of embeddings |
| Loss computation | `y · log(ŷ)` | Cross-entropy sums element-wise products |

Every one of these is the same operation: multiply-and-sum over a shared dimension.

---

## Relation to other products

The dot product is a special case in a family:

| Operation | Input shapes | Output shape | Shared dim |
|-----------|-------------|--------------|------------|
| Dot product | (n,) · (n,) | () | n summed |
| Matrix-vector multiply | (m×n) · (n,) | (m,) | n summed |
| Matrix-matrix multiply | (m×k) · (k×n) | (m×n) | k summed |

See the pattern? In all three, a shared dimension is summed over. The dot product is the rank-1 version. [04 - Matrix-Vector Multiplication](04-Matrix-Vector-Multiplication) is what happens when you compute *m* dot products at once.

---

Previous: [02 - Element-wise Operations](02-Element-wise-Operations) | Next: [04 - Matrix-Vector Multiplication](04-Matrix-Vector-Multiplication)
