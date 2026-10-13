# 算子库与推理框架 {#sec:sw-frameworks}

## 算子库

* 分层结构：API → Interface → Kernel（generic / rvv / 矩阵 / triton）→ Platform；
* 统一 validate/execute 接口，覆盖 gemm、gemv、matmul、softmax、silu、rms_norm 等。

TODO：补充算子清单与示例代码。

## 接入矩阵单元的通用方法

把矩阵单元接进任意推理框架，按以下五步推进，顺序不可调换。

1. **探针定规格**：从 CSR 读出当前 bit 的原生 tile 几何（default 128×64×128、mini 64×64×64），只按探针结果生成核。规格与 bit 不匹配必然算错。
2. **找框架原生接缝**：优先"换实现即换核"的函数表；其次 arch 层 layer 实现；最后才是调用点 drop-in。
3. **单一翻译单元**：所有矩阵指令集中在同一个用 `zames` march 编译的 `.c` 里，其他文件只经头文件调用。
4. **门禁 + 回落**：能力、形状、工作量三重门禁，任一不满足就让上层走框架原核，绝不返回错数据。
5. **差分验证 → 阶梯上板**：位级一致优先，QEMU 只验算术，板级结论才是权威。

### 硬约束

| 约束 | 内容 |
| --- | --- |
| tile 规格 | 只有 `128×64×128`（default）与 `64×64×64`（mini）两版，规格从探针取 |
| bf16 的 K 参数 | `msettilek=32`（按元素计），而 MLA/MLB 的 N 参数按 **64 字节**计；累加器 C 恒为 fp32 |
| 操作数池 | 必须物理连续（`/dev/mem` @ `0x100000000`）；AMU 行寻址不查 TLB，非连续内存静默读错 |
| 基址寄存器 | MSC 基址钉 `s11`，保活到 `macquire`；每张 tile 用 `msyncregreset` |
| 同步 | 统一用 `mfence`；`mrelease` / `macquire` 必须覆盖所有退出分支 |

### 门禁

| 门禁 | 判据 | 不满足时 |
| --- | --- | --- |
| 规格可用 | 探针给出可信几何 | 整条矩阵路径关闭 |
| 形状 | K ≤ 上限（默认 8192 元素）、`real_count` 在合法范围 | 回落框架原核 |
| 工作量 | `oc × K ≥ 16384`（可用环境变量调整） | 回落框架原核 |

工作量门禁的必要性：卷积的挂接点每次只给 8 个像素，`tile_m = 128` 时 M 利用率恒定 6%，只有 `oc × K` 够大才值得占用一张几乎空的 tile。小规模的层交给框架自带的 RVV 核反而更快。

## 框架接缝对照

| 框架 | 接缝 | 说明 |
| --- | --- | --- |
| MNN | core functions 函数表（`MNNPackedMatMul` 等） | "换表即换核"，覆盖面最广；LLM 的 `CPUMatMul`、`CPUAttention`、`BatchMatMul` 全走同一张表 |
| NCNN | arch 层 layer 实现（registry + `Convolution_riscv::forward`） | 无函数表，不要在调用点做 drop-in（各变体要分别插入且需自己做 pack）；riscv int8 卷积原本空缺，从零写反而干净 |
| ggml | `ggml-cpu/arch/riscv/ame/ame-matmul.c` | 参照实现；选核要求 `M ≥ 64` 且 `N ≥ 64`，即 decode（M=1）不吃 tile、prefill 才受益 |

## llama.cpp / GGML

* 通过矩阵后端接入，模型侧可细分为算子 diff 测试与逐层采样；
* 选核门禁 `M ≥ 64 且 N ≥ 64`，decode 走 vec-dot/GEMV，prefill 才用矩阵单元；
* 注意 K 维对齐：K 不是 128 整数倍时需要 padding 或尾数处理（相关算子曾因此产生偏差）。

TODO：补充构建开关与性能数据。

## 端侧框架

* **MNN**：int8 pack 约定 `UNIT=4(oc)` / `SRC_UNIT=16(ic)` / `DST_XUNIT=8(px)`；bf16 层保持 `bytes=4`、仅 matmul 操作数 `matmulBytes=2`；`CPUMatMul` 的 `[l][eP]` 布局（转置 + 原地 lowp）与 bf16 核期望布局天然兼容，不要额外加转换层；LLM 在 riscv 上默认走 fp32，需要显式设 `Precision_Low_BF16`。
* **NCNN**：交叉编译必须关掉宿主 ISA 选项（`NCNN_SSE2` / `AVX` / `AVX2` / `AVXVNNI` / `AVX512` / `XOP` / `FMA=OFF`）并使用 toolchain file；测试**成功时静默**（`rc=0` 无输出），判定靠返回码加调用计数；`onnx2ncnn` 对动态 shape 处理不正确（bin 头写 NaN、生成未注册的 Shape 层），需先在 ONNX 层把 `Shape→Gather→Unsqueeze→Concat→Reshape` 折叠为常量，折叠后用 ONNX Runtime 验证 argmax 不变再转换。
* **fp16 权重模型**：转到 bf16 后端即可（ggml `F16_BF16` 思路），不必为 fp16 单独写核。

## 量化格式的处理原则

矩阵核只做 GEMM（int8 → int32 精确累加）；scale 表、zero-point 补偿、requant/dequant、clamp、融合激活一律照抄框架参考实现的 epilogue。这样输出能与框架参考位级一致，格式差异被天然覆盖。即**接缝是 GEMM，不是算子**，不要在核里重新实现量化格式。

| 维度 | 变体 | 处理方式 |
| --- | --- | --- |
| scale 粒度 | per-tensor / per-output-channel | per-oc 最常见；按 oc 取尾表，读取路径与 per-tensor 完全一致 |
| 输入 zero-point | symmetric / asymmetric | MNN 用 `srcSum × weightBias` 补偿：先求 A 的行和，再在 epilogue 加该修正项 |
| 输出类型 | requant→int8 / dequant→fp32 | 两条 epilogue 分支都要实现（NCNN 以 `int8_scale_term > 100` 判定 requant；除零要判） |
| 饱和 / 钳位 | int8 `[-127,127]` / fp32 `fp32minmax` | 照抄参考实现 |
| 融合激活 | 有 | 沿用框架的 activation 参数，不要自行近似 |

**不支持的格式直接门禁回落，不做近似**：权重不是每元素 1 字节（4-bit / 6-bit / 混合精度压缩权重）、packed 布局（`elempack > 1` 需先 unpack 或让 A 的打包感知 pack）、形状超限、工作量不足。

bf16 与 fp16 属于**精度**而非量化格式：bf16 是 fp32 取高 16 位、fp32 累加、C 按框架契约写 fp32 或截断到 bf16 网格。bf16 与 int8 的开关必须分开，因为 bf16 会改变浮点精度，共用一个开关会让官方 fp32 测试因容差挂掉。

## 验证口径

```plain
框架自带测试（算子级）→ QEMU 用户态（网络级，仅算术判据）→ NEMU → 云平台 FPGA（权威）
```

* **优先使用框架自带测试，不要另造一套**：MNN 用 `run_test.out <test> 0 <precision>`（3 为 `Low_BF16`）；NCNN 每个算子一个可执行文件，`rc=0` 即通过；ggml 用 `test-backend-ops`、`test-op`。
* 可信度排序：① 同一模型在矩阵开关开、关两种模式下输出**逐字节相同**（最硬的证据，与精度无关）；② 框架自带测试在两种模式下都通过，且失败只出现在开关打开时；③ 自造参考实现双模式逐元素比对（int8 要求位级一致，bf16 用双层公差）；④ 与外部真值（ONNX Runtime 或 host 框架）比 argmax 或 top-5。
* **量化校准方法与矩阵适配无关**：int8 校准用百分位还是 KL 会改变 argmax（实测 278 与 282），这不是适配引入的误差；适配是否正确只看向量开关开、关下的位级一致性。
* 自造差分工具只在框架测试覆盖不到时补位（某个形状、尾块、深 K 组合），或需要逐元素位级比对时使用。
