# 附录 C：参考文档与仓库 {#sec:appendix-references}

## 相关手册

* 《香山开源处理器用户手册》——本手册的基线行为
* 香炉峰 ISA 规范与提案（ZAMES / ZMASYNC / 低精度转换）

## 代码仓库

TODO：补充 RTL、编译器、模拟器、软件栈、算子库与文档仓库地址。

## 工具与资料

* `xsai-env` 环境说明与工作流文档（`docs/`、`AGENTS.md`）
* 环境自检：`make test-smoke` / `make test`
* 问题上报：`scripts/bug-report.sh`、`scripts/create-issue.sh`

## 构建产物与负载

| 路径 | 内容 |
| --- | --- |
| `XSAI/build/emu` | Verilator 仿真器 |
| `XSAI/build/rtl/SimTop.sv` | 仿真专用 RTL 顶层 |
| `XSAI/build/rtl/XSTop.sv` | 综合 / FPGA 风格 RTL 顶层 |
| `XSAI/ready-to-run/*.bin` | 预置裸机镜像 |
| `firmware/gcpt_restore/build-*/build/gcpt.bin` | checkpoint 恢复启动镜像 |
| `firmware/checkpoints/**/*.zstd` | checkpoint 分片 |
