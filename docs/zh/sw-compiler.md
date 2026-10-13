# 编译器与算子 DSL {#sec:sw-compiler}

## LLVM 后端

* 基于 LLVM 的 RISC-V 后端扩展，支持 `ZAMES`（矩阵计算）与 `ZMASYNC`（矩阵同步）；
* 提供 intrinsic 与内建类型，并负责伪指令展开、配置指令的自动插入与去重；
* 矩阵寄存器按 callee-saved 约定处理：跨调用时整寄存器 spill / reload。

TODO：补充支持的 LLVM 版本、构建步骤与示例。

## Triton

编译路径：

```plain
Triton IR → Linalg → Zames → LLVM IR → 汇编
```

* 显式异步操作可用 `tl.regreset`、`tl.release`、`tl.acquire`、`tl.fence` 表达；
* 与 CPU 后端相比，矩阵路径需要把 tile 尺寸、K 分块与同步点显式表达出来。

TODO：补充安装、编译命令与算子示例。

## TileLang

* 已接入矩阵 intrinsic，可用于表达 GEMM 等算子；
* 定位是把算子 DSL 的开发效率带到矩阵单元上，避免手写汇编。

TODO：补充支持范围与用法。

## buddy-mlir

* 作为多层级编译流的中间层使用，承接从高层算子到后端代码的下降。

TODO：补充在香炉峰工具链中的具体位置与职责边界。

## 版本对应

TODO：编译器版本与 ISA 版本、模拟器版本的对应表。
