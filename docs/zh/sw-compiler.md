# 编译器与算子 DSL {#sec:sw-compiler}

## LLVM 后端

* 基于 LLVM 的 RISC-V 后端扩展，支持 ZAMES / ZMASYNC；
* 提供 intrinsic、内建类型，以及伪指令展开、配置指令自动插入与去重；
* 矩阵寄存器的 callee-saved 约定（整寄存器 spill/reload）。

TODO：补充支持的 LLVM 版本、构建步骤与示例。

## Triton

* 路径：Triton IR → Linalg → Zames → LLVM IR → 汇编；
* 支持显式异步操作（`tl.regreset` / `tl.release` / `tl.acquire` / `tl.fence`）。

TODO：补充安装、编译命令与算子示例。

## TileLang

* 已接入矩阵 intrinsic，可用于表达 GEMM 等算子。

TODO：补充支持范围与用法。

## 版本对应

TODO：编译器版本与 ISA 版本、模拟器版本的对应表。
