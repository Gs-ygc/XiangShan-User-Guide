# Kernel Libraries and Inference Frameworks {#sec:sw-frameworks}

## Kernel library

* Layered structure: API → Interface → Kernel (generic / rvv / matrix / triton)
  → Platform;
* Unified validate/execute interface covering gemm, gemv, matmul, softmax, silu,
  rms_norm and more.

TODO: operator list and example code.

## llama.cpp / GGML

* Integrated through the matrix backend, with operator-level diff tests and
  per-layer sampling on the model side;
* Mind the K-dimension alignment: when K is not a multiple of 128, padding or
  tail handling is required (a known source of numerical deviation).

TODO: build switches, accuracy alignment method and performance data.

## Edge frameworks

TODO: integration path, verification method and known pitfalls.

## Integration methodology

1. Choose the seam carefully: prefer pushing down into the kernel library rather
   than patching the framework;
2. Differential verification against the framework's CPU backend, layer by
   layer and operator by operator;
3. Idempotent guards and fallback to generic paths for abnormal inputs;
4. Record pitfalls in documentation so they are not re-debugged.
