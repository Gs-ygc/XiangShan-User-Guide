# Kernel Libraries and Inference Frameworks {#sec:sw-frameworks}

## Kernel library

* Layered structure: API → Interface → Kernel (generic / rvv / matrix / triton)
  → Platform;
* Unified validate/execute interface covering gemm, gemv, matmul, softmax, silu,
  rms_norm and more.

TODO: operator list and example code.

## General method for integrating the matrix unit

To integrate the matrix unit into any inference framework, follow these five
steps in order; the order cannot be swapped.

1. **Probe establishes the spec**: read the native tile geometry of the current
   bit from the CSR (default 128×64×128, mini 64×64×64), and generate kernels
   only from the probe result. A spec that does not match the bit always
   computes wrong results.
2. **Find the framework's native seam**: prefer the function table where
   "replacing the implementation replaces the kernel"; next the arch-layer
   layer implementation; and only then a drop-in at the call site.
3. **Single translation unit**: all matrix instructions live in one `.c` file
   compiled with the `zames` march, and other files reach them only through the
   header.
4. **Guards + fallback**: three guards, namely capability, shape and workload;
   if any one of them is not satisfied, let the upper layer fall back to the
   framework's own kernel, and never return wrong data.
5. **Differential verification → staged bring-up on the board**: bit-level
   agreement comes first, QEMU verifies arithmetic only, and a board-level
   conclusion is the only authoritative one.

### Hard constraints

| Constraint | Content |
| --- | --- |
| tile spec | Only two versions, `128×64×128` (default) and `64×64×64` (mini); the spec is read from the probe |
| bf16 K parameter | `msettilek=32` (counted in elements), whereas the N parameter of MLA/MLB is counted in **64-byte** units; accumulator C is always fp32 |
| Operand pool | Must be physically contiguous (`/dev/mem` @ `0x100000000`); AMU row addressing does not walk the TLB, so non-contiguous memory is read incorrectly without any error |
| Base register | The MSC base is pinned to `s11` and kept live until `macquire`; every tile uses `msyncregreset` |
| Synchronization | Always use `mfence`; `mrelease` / `macquire` must cover all exit branches |

### Guards

| Guard | Criterion | When not satisfied |
| --- | --- | --- |
| Spec available | The probe returns trustworthy geometry | The whole matrix path is disabled |
| Shape | K ≤ upper bound (8192 elements by default), `real_count` within the legal range | Fall back to the framework's own kernel |
| Workload | `oc × K ≥ 16384` (adjustable through an environment variable) | Fall back to the framework's own kernel |

Why the workload guard is necessary: the convolution hook point only hands
over 8 pixels at a time, so with `tile_m = 128` the M utilization is
constantly 6%; occupying an almost empty tile is worthwhile only when `oc × K`
is large enough. For small layers the framework's own RVV kernel is faster
instead.

## Framework seam comparison

| Framework | Seam | Note |
| --- | --- | --- |
| MNN | core functions table (`MNNPackedMatMul` and others) | "Swapping the table swaps the kernel", with the widest coverage; the LLM paths `CPUMatMul`, `CPUAttention` and `BatchMatMul` all go through the same table |
| NCNN | arch-layer layer implementation (registry + `Convolution_riscv::forward`) | There is no function table, so do not do a drop-in at the call site (each variant must be inserted separately and does its own packing); riscv int8 convolution was missing to begin with, so writing it from scratch is cleaner |
| ggml | `ggml-cpu/arch/riscv/ame/ame-matmul.c` | Reference implementation; kernel selection requires `M ≥ 64` and `N ≥ 64`, so decode (M=1) does not take tiles and only prefill benefits |

## llama.cpp / GGML

* Integrated through the matrix backend, with operator diff tests and per-layer
  sampling on the model side;
* The kernel selection guard is `M ≥ 64` and `N ≥ 64`: decode goes through
  vec-dot/GEMV, and only prefill uses the matrix unit;
* Mind K-dimension alignment: when K is not a multiple of 128, padding or tail
  handling is required (this has caused deviations in the relevant operators
  before).

TODO: build switches and performance data.

## Edge frameworks

* **MNN**: the int8 pack convention is `UNIT=4(oc)` / `SRC_UNIT=16(ic)` /
  `DST_XUNIT=8(px)`; bf16 layers keep `bytes=4` and only the matmul operands use
  `matmulBytes=2`; the `[l][eP]` layout of `CPUMatMul` (transpose plus in-place
  lowp) is naturally compatible with the layout the bf16 kernel expects, so do
  not add an extra conversion layer; on riscv LLMs go through fp32 by default,
  so `Precision_Low_BF16` must be set explicitly.
* **NCNN**: cross-compilation must switch off the host ISA options (`NCNN_SSE2`
  / `AVX` / `AVX2` / `AVXVNNI` / `AVX512` / `XOP` / `FMA=OFF`) and use a
  toolchain file; the tests are **silent on success** (`rc=0` with no output),
  so pass/fail relies on the return code plus a call counter; `onnx2ncnn`
  mishandles dynamic shapes (it writes NaN into the bin header and emits an
  unregistered Shape layer), so first fold
  `Shape→Gather→Unsqueeze→Concat→Reshape` into constants at the ONNX level, and
  after folding verify with ONNX Runtime that argmax is unchanged before
  converting.
* **fp16 weight models**: just convert them to the bf16 backend (the ggml
  `F16_BF16` idea); there is no need to write a separate kernel for fp16.

## Principles for handling quantization formats

The matrix kernel does GEMM only (int8 → int32 with exact accumulation); the
scale table, zero-point compensation, requant/dequant, clamp and fused
activation all copy the epilogue of the framework's reference implementation.
This way the output is bit-identical with the framework reference, and format
differences are covered by construction. That is, **the seam is GEMM,
not the operator**, so do not reimplement the quantization format inside the
kernel.

| Dimension | Variant | Handling |
| --- | --- | --- |
| scale granularity | per-tensor / per-output-channel | per-oc is the most common; take the tail table by oc, and the read path is exactly the same as per-tensor |
| Input zero-point | symmetric / asymmetric | MNN compensates with `srcSum × weightBias`: first compute the row sums of A, then add that correction term in the epilogue |
| Output type | requant→int8 / dequant→fp32 | Both epilogue branches must be implemented (NCNN decides requant with `int8_scale_term > 100`; division by zero must be checked) |
| Saturation / clamping | int8 `[-127,127]` / fp32 `fp32minmax` | Copy the reference implementation |
| Fused activation | Yes | Reuse the framework's activation parameters; do not approximate them yourself |

**Unsupported formats are guarded and fall back directly, with no
approximation**: weights that are not 1 byte per element (4-bit / 6-bit /
mixed-precision compressed weights), packed layouts (`elempack > 1` requires
unpacking first, or making A's packing pack-aware), out-of-range
shapes, and insufficient workload.

bf16 and fp16 are a matter of **precision**, not of quantization format: bf16
takes the high 16 bits of fp32, accumulates in fp32, and writes C as fp32 or
truncates it to the bf16 grid according to the framework contract. The switches
for bf16 and int8 must be kept separate, because bf16 changes floating-point
precision and a shared switch would make the official fp32 tests fail on
tolerance.

## Verification criteria

```plain
framework's own tests (operator level) → QEMU user mode (network level, arithmetic criteria only) → NEMU → cloud platform FPGA (authoritative)
```

* **Prefer the framework's own tests; do not build a second suite**: for MNN use
  `run_test.out <test> 0 <precision>` (3 means `Low_BF16`); for NCNN every
  operator has its own executable and `rc=0` means it passes; for ggml use
  `test-backend-ops` and `test-op`.
* Ranking of evidence strength: (1) the same model produces **byte-identical**
  output with the matrix switch on and off (the hardest evidence, independent
  of precision); (2) the framework's own tests pass in both modes, and failures
  appear only with the switch on; (3) a self-built reference implementation is
  compared element by element in both modes (bit-level agreement for int8, a
  two-level tolerance for bf16); (4) argmax or top-5 is compared against an
  external ground truth (ONNX Runtime or a host framework).
* **The quantization calibration method is unrelated to matrix adaptation**:
  whether int8 calibration uses percentile or KL changes the argmax (measured
  278 versus 282), and this is not an error introduced by the adaptation;
  whether the adaptation is correct is judged only by bit-level agreement with
  the vector switch on and off.
* A self-built differential tool only fills the gaps that the framework tests
  do not cover (a particular shape, tail block or deep-K combination), or is
  used when element-by-element bit-level comparison is required.
