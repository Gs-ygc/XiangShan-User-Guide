# Compiler and Operator DSL {#sec:sw-compiler}

## LLVM backend

* A RISC-V backend extension based on LLVM, supporting `ZAMES` (matrix
  computation) and `ZMASYNC` (matrix synchronization);
* Provides intrinsics and builtin types, and handles pseudo-instruction
  expansion together with automatic insertion and de-duplication of
  configuration instructions;
* Matrix registers follow the callee-saved convention: whole-register spill /
  reload across calls.

TODO: supported LLVM versions, build steps and examples.

## Triton

Compilation path:

```plain
Triton IR → Linalg → Zames → LLVM IR → assembly
```

* Explicit asynchronous operations can be expressed with `tl.regreset`,
  `tl.release`, `tl.acquire` and `tl.fence`;
* Compared with the CPU backend, the matrix path requires tile sizes, K
  blocking and synchronization points to be expressed explicitly.

TODO: installation, compile commands and operator examples.

## TileLang

* Matrix intrinsics are already wired in, so operators such as GEMM can be
  expressed;
* Its purpose is to bring the development efficiency of an operator DSL to the
  matrix unit and avoid hand-written assembly.

TODO: supported scope and usage.

## buddy-mlir

* Used as the intermediate layer of the multi-level compilation flow, carrying
  the lowering from high-level operators down to backend code.

TODO: its exact position and responsibility boundaries in the Xianglufeng
toolchain.

## Version matrix

TODO: mapping between compiler, ISA and simulator versions.
