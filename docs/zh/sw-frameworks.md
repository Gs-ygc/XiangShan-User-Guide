# 算子库与推理框架 {#sec:sw-frameworks}

## xsai-blas

- 分层结构：API → Interface → Kernel（generic / rvv / ame / triton）→ Platform
- 统一 validate/execute 接口，覆盖 gemm、gemv、matmul、softmax、silu、rms_norm 等

TODO：补充算子清单与示例代码。

## llama.cpp / GGML

- 通过 `GGML_USE_AME` 后端接入矩阵单元
- 注意 K 维对齐（如 K 不是 128 的整数倍时的 padding 与尾数处理）

TODO：补充构建开关、精度对齐方法与性能数据。

## MNN 等端侧框架

TODO：补充接入路径、验证方法与已知坑。

## 适配方法论

- 接缝选择：优先在算子库层下沉，而非直接改框架
- 差分验证：与框架 CPU 后端逐层/逐算子对比
- 幂等门禁与回落：异常输入回落到通用实现

TODO：补充完整流程与踩坑目录。
