# 算子库与推理框架 {#sec:sw-frameworks}

## 算子库

* 分层结构：API → Interface → Kernel（generic / rvv / 矩阵 / triton）→ Platform；
* 统一 validate/execute 接口，覆盖 gemm、gemv、matmul、softmax、silu、rms_norm 等。

TODO：补充算子清单与示例代码。

## llama.cpp / GGML

* 通过矩阵后端接入，模型侧可细分为算子 diff 测试与逐层采样；
* 注意 K 维对齐：K 不是 128 整数倍时需要 padding 或尾数处理（相关算子曾因此产生偏差）。

TODO：补充构建开关、精度对齐方法与性能数据。

## 端侧框架

TODO：补充接入路径、验证方法与已知坑。

## 适配方法论

1. 接缝选择：优先在算子库层下沉，而非直接改框架；
2. 差分验证：与框架 CPU 后端逐层/逐算子对比；
3. 幂等门禁与回落：异常输入回落到通用实现；
4. 记录踩坑：把每轮适配的坑固化到文档，避免重复排查。
