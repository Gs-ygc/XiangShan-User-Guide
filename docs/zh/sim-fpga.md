# FPGA 上板与云平台 {#sec:sim-fpga}

## 本地流程

```bash
make fpga-rtl        # 生成面向 FPGA 的 RTL
make fpga-synth      # Vivado 工程/综合
make fpga-bitstream  # 生成 bitstream
make fpga-program    # 烧录
make fpga-run-bitstream  # 烧录后直接跑负载
```

默认 FPGA RTL 为单核 `FpgaMinimalMatrix256KConfig`：LLC 使用 ZhuJiang、CHI issue E.b、256 KiB 八 bank L2 与 256 KiB LLC，并默认应用面向综合的 VipArbiter 补丁。

常用旋钮：

| 分类 | 变量 |
| --- | --- |
| RTL 生成 | `FPGA_RTL_CONFIG`、`FPGA_RTL_NUM_CORES`、`FPGA_RTL_LLC`、`FPGA_RTL_L2_CACHE_SIZE`、`FPGA_RTL_OUT_ROOT` |
| 综合 | `FPGA_SYNTH_PROJECT`、`FPGA_SYNTH_JOBS`、`FPGA_SYNTH_RUN_TO` |
| ILA | `FPGA_ILA_DEPTH`、`FPGA_ILA_PROFILE(full\|compact\|minimal\|axi\|axi-valid)`、`FPGA_ILA_CAPTURE_TIMEOUT_MS` |
| 运行 | `FPGA_HOST`、`PAYLOAD`、`FPGA_XDMA_*`、`FPGA_UART_CMD`、`FPGA_TIMEOUT` |

抓取一次 ILA 窗口：

```bash
make fpga-insert-zhujiang-ila
make fpga-capture-ila
```

## 云平台（思沃云）

平台提供 GitLab 流水线、板卡队列与串口终端。推荐用封装脚本，避免手工轮询：

```bash
# 从零触发一轮负载测试（默认烧 default 比特流，串口输出自动采集）
bash run-payload.sh <payload.bin> --bitstream default

# 比特流选择：keep=沿用板上现值（最快）| default | mini（minimalMatrix）
bash run-payload.sh <payload.bin> --bitstream keep

# 交互式：板上 shell 可读写
bash run-payload.sh <payload.bin> --interactive
```

监视已有任务：

```bash
python3 watch.py job --job <id> --mode uart|stages|full   # --once 单次采样
python3 watch.py status                                   # 板卡/runner 状态
```

判读规则：

* `job pending` 且另有 job 在跑 → **正常排队**，等待即可；
* `job running` 但串口长时间无输出 → 多为负载静默期；若阶段停留在构建/烧录阶段超过 5 分钟才可疑；
* 负载完成由哨兵字符串判定，不必等窗口到期；连续 600 s 无串口输出会触发**卡死看门狗**终止任务，避免占板。

## 负载准备

建议维护一组"板上已验证"的可演示 payload，并为每个负载提供：

* 明确的完成标记（供看门狗与判定使用）；
* 预期运行时间量级；
* 所需比特流（`default` / `mini`）与是否依赖中断/UART 输入。

## 已知限制

* 裁剪配置（minimal matrix）与完整配置的行为/性能存在差异，比较时需固定配置；
* 云平台上传带宽有限，固件镜像较大的场合需预留传输时间。
