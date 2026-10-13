# VCS 仿真与用例框架 {#sec:sim-vcs}

## 定位

VCS 主要用于矩阵单元与 L2 的模块级/系统级验证、覆盖率收集，以及与 Verilator 的双引擎交叉确认。

## 用例框架

* **CUTE-Test**：矩阵单元用例集，支持与 NEMU 参考模型逐步比对；`make test` 可跑全量回归。
* **CUTE-CTRL**：控制通路用例与调试入口。
* **TestTopWithL2**：带 L2 的新 DUT，存储侧改用一致性接口（无监听子集），自建 SN 侧功能模型替代真实 SN，并提供协议检查器（事务合法性 + 超时事务定位）。
* **CUTE-UVM**：UVM 层次化验证环境，按"核心计算 → 数据流控制 → 存储加载 → 寄存器文件"四层组织。

| 层次 | 典型模块 |
| --- | --- |
| 第 1 层 核心计算 | `FReducePE`、`MatrixTE` 及其子模块（MAC 流水、比较树、缩放/向量解码器） |
| 第 2 层 数据流控制 | `ADataController`、`BDataController`、`CDataController`、缩放控制器 |
| 第 3 层 存储加载 | `AMemoryLoader`、`BMemoryLoader`、`CMemoryLoader`、缩放加载器 |
| 第 4 层 寄存器文件 | `ABMatrixReg`、`CMatrixReg`、`ABScaleMatrixReg`、`SRAMTemplate` |

## 双引擎对齐

同一批用例分别在 Verilator 与 VCS 上运行，比较**通过/失败集合**与**超时签名**；两者逐字节一致时，才能把故障收敛到 DUT 本身。首次搭建带 L2 的环境时，两引擎曾同时给出"25 通过 / 32 失败"且 32 条超时签名完全一致，由此确认行为等价。

## 构建注意事项

1. 先用 **关闭 ChiselDB / DRAMsim3** 的配置跑通 Verilator，再进入 VCS：

   ```bash
   make -C XSAI emu -j8 CONFIG=DefaultMatrixConfig \
     WITH_DRAMSIM3=0 WITH_CHISELDB=0 WITH_CONSTANTIN=0 \
     EMU_THREADS=8 EMU_TRACE=fst
   ```

2. 将 L2 子模块更新到修复了已知缺陷的版本（不必跟随 XSAI 的锁定版本）。
3. difftest 侧需处理 `mma_backend` 的 C++17 特性。

## 覆盖率

以 Verilator 收集行/分支覆盖率作为日常基线（例如某轮 line 96% / branch 74%，未达标主要来自挂死用例的下游路径未执行，而非覆盖遗漏）。VCS 侧用于补充确认与时序相关检查。
