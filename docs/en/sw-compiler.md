# Compiler and Operator DSL {#sec:sw-compiler}

## LLVM backend

* RISC-V backend extension supporting ZAMES / ZMASYNC;
* Provides intrinsics and builtin types, pseudo-instruction expansion,
  automatic insertion and de-duplication of configuration instructions;
* callee-saved convention for matrix registers (whole-register spill/reload).

TODO: supported LLVM versions, build steps and examples.

## Triton

* Path: Triton IR → Linalg → Zames → LLVM IR → assembly;
* Supports explicit asynchronous operations (`tl.regreset` / `tl.release` /
  `tl.acquire` / `tl.fence`).

TODO: installation, compile commands and operator examples.

## TileLang

* Matrix intrinsics are wired in, which is enough to express GEMM-like operators.

TODO: supported scope and usage.

## Version matrix

TODO: mapping between compiler, ISA and simulator versions.
