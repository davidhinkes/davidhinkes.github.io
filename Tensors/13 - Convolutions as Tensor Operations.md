# Convolutions as Tensor Operations

Convolutions look different from dense layers — they have kernels, strides, and spatial structure. But under the hood, every convolution can be reformulated as a [[05 - Matrix-Matrix Multiplication|matrix multiplication]]. Understanding this connection is the key to seeing convolutions through the tensor lens.

---

## What a convolution does

A 2D convolution slides a small filter (kernel) over an image, computing a dot product at each position.

Single-channel example: input (5×5), kernel (3×3), no padding, stride 1:

```
Input (5×5):                  Kernel (3×3):
[1  0  1  2  1]              [1  0  1]
[0  1  0  1  0]              [0  1  0]
[2  1  2  0  1]              [1  0  1]
[1  0  1  1  0]
[0  1  0  2  1]

Output (3×3):
  position (0,0): dot product of kernel with top-left 3×3 patch
  = 1·1 + 0·0 + 1·1 + 0·0 + 1·1 + 0·0 + 2·1 + 1·0 + 2·1 = 7
```

Each output element is a [[03 - The Dot Product|dot product]] of the flattened kernel (9 elements) with a flattened image patch (9 elements). The kernel slides to produce all H_out × W_out outputs.

---

## The shapes involved

For a full convolution layer:

```
Input:   (B, C_in, H, W)        — batch of multi-channel images
Kernel:  (C_out, C_in, kH, kW)  — C_out filters, each spanning all C_in channels
Output:  (B, C_out, H', W')     — batch of multi-channel feature maps
```

Output spatial size (with padding p and stride s):

```
H' = (H + 2p - kH) / s + 1
W' = (W + 2p - kW) / s + 1
```

---

## The im2col trick: convolution as matrix multiply

The key insight: rearrange the input so that each patch the kernel slides over becomes a row (or column) in a matrix. Then the convolution is a single GEMM.

### Step 1 — im2col

Extract every (C_in × kH × kW) patch and stack them as rows:

```
Input:  (B, C_in, H, W)
Patches: (B·H'·W', C_in·kH·kW)     — one row per patch
```

For B=1, C_in=1, H=W=5, kH=kW=3: there are 3×3=9 patches, each with 1×3×3=9 elements. The patch matrix is (9, 9).

### Step 2 — Reshape the kernel

```
Kernel: (C_out, C_in·kH·kW)     — one row per output filter
```

### Step 3 — GEMM

```
Output = Patches · Kernelᵀ       (B·H'·W', C_out) = (B·H'·W', C_in·kH·kW) · (C_in·kH·kW, C_out)
```

Then reshape the output back to (B, C_out, H', W').

### Why this works

Each output pixel is a dot product of one filter with one patch. The matrix multiply computes all (H'·W' × C_out) dot products simultaneously. The convolution *is* a matrix multiplication — just with a specially-constructed input matrix.

---

## Concrete example

Input (1 channel, 4×4), one 3×3 filter:

```
Input:                    Kernel:
[1 2 3 4]               [1 0 1]
[5 6 7 8]               [0 1 0]
[9 0 1 2]               [1 0 1]
[3 4 5 6]

Output size: (4-3+1) × (4-3+1) = 2×2
```

im2col extracts 4 patches (one per output position), each flattened to 9 elements:

```
Patches (4×9):
[1 2 3 5 6 7 9 0 1]    ← top-left patch
[2 3 4 6 7 8 0 1 2]    ← top-right
[5 6 7 9 0 1 3 4 5]    ← bottom-left
[6 7 8 0 1 2 4 5 6]    ← bottom-right

Kernel flattened (1×9):
[1 0 1 0 1 0 1 0 1]
```

Each output = dot product of a patch row with the kernel:

```
pos (0,0): 1+3+6+9+1 = 20
pos (0,1): 2+4+7+0+2 = 15
pos (1,0): 5+7+0+3+5 = 20
pos (1,1): 6+8+1+4+6 = 25

Output = [20  15]
         [20  25]
```

---

## The memory tradeoff

im2col duplicates input data — overlapping patches share elements that get copied into multiple rows. For a 3×3 kernel with stride 1, each input pixel appears in up to 9 patches. The patch matrix is C_in·kH·kW times larger than the original input.

This is the classic space-time tradeoff: im2col uses more memory but converts the convolution into a single GEMM, which BLAS libraries are heavily optimized for.

Modern implementations (cuDNN, etc.) use variants:
- **Implicit im2col**: compute patch elements on-the-fly without materializing the full matrix
- **Winograd transforms**: reduce the number of multiplications for small kernels
- **FFT-based**: convert to frequency domain where convolution becomes element-wise multiplication (the [[02 - Element-wise Operations|Hadamard product]])

---

## 1×1 convolutions are literally matrix multiplies

A 1×1 convolution with C_out filters on (B, C_in, H, W) input:

```
Kernel: (C_out, C_in, 1, 1) → reshape to (C_out, C_in)
Input:  (B, C_in, H, W) → reshape to (B·H·W, C_in)

Output = Input_reshaped · Kernelᵀ     (B·H·W, C_out)
       → reshape to (B, C_out, H, W)
```

No im2col needed. No overlapping patches. It's a dense layer applied independently to every spatial position. This is why 1×1 convolutions are cheap and appear everywhere in architectures like ResNet and Inception.

---

## Depthwise convolutions

A depthwise convolution applies a separate filter to each channel:

```
Output_bc = Input_bc ★ Kernel_c     (B, C, H', W')
```

No cross-channel mixing. Followed by a 1×1 convolution ("pointwise") that does mix channels. Together, depthwise separable convolution factorizes the full convolution into two cheaper operations — reducing parameter count from (C_out × C_in × kH × kW) to (C_in × kH × kW) + (C_out × C_in).

---

Previous: [[12 - Tensors in Backpropagation]] | Next: [[14 - Attention is All Tensor Ops]]
