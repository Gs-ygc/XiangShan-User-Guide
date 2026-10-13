# 编译器与算子 DSL {#sec:sw-compiler}

## LLVM 后端

- 基于 LLVM 的 RISC-V 后端扩展，支持 ZAMES / ZMASYNC
- 提供 intrinsic、内建类型与必要的伪指令展开、配置指令自动插入与去重
- 矩阵寄存器的 callee-saved 约定

TODO：补充支持的 LLVM 版本、构建步骤与示例。

## Triton

- 路径：Triton IR → Linalg → Zames → LLVM IR → 汇编
- 支持显式异步操作（`tl.regreset` / `tl.release` / `tl.acquire` / `tl.fence`）

TODO：补充安装、编译命令与算子示例。

## TileLang

TODO：补充支持范围与用法。

## 版本对应关系

TODO：补充编译器版本与 ISA 版本、模拟器版本的对应表。
