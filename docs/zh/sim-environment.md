# 环境准备 {#sec:sim-environment}

## xsai-env 总览

`xsai-env` 是香炉峰的集成开发工作区：顶层 `Makefile` 负责编排，真正的 RTL 工程位于子目录 `XSAI/`，功能模拟器位于 `NEMU/`、`QEMU/`，固件与 rootfs 位于 `firmware/`。

```bash
git clone <xsai-env> ~/xsai-env
cd ~/xsai-env
```

## 首次准备

```bash
make deps         # 安装系统依赖（需要 sudo）
make init         # 初始化子模块与环境
make init-force   # 首次重编译前推荐，避免空目录
source env.sh     # 或 direnv allow
```

环境要点：

* `RISCV`、`RISCV_SYSROOT`、`QEMU_LD_PREFIX` 会自动探测；机器相关覆盖请写入 `.envrc.local`。
* 顶层**不导出** `CROSS_COMPILE`，固件与软件流程各自设置工具链前缀。
* Nix 为可选项，仅在需要复现 Nix 特定问题时使用 `make nix-shell` / `nix-init` / `nix-smoke`。

## 生成产物目录

`local/`、`build/`、`log/`、`firmware/checkpoints/` 均为生成产物，不入库。

## 常用目标

| 目标 | 作用 |
| --- | --- |
| `make xsai` | 构建 Verilator 仿真器 `XSAI/build/emu` |
| `make run-emu` | 运行 Verilator 仿真（见[Verilator 仿真](sim-verilator.md#sec:sim-verilator)） |
| `make run-nemu` | 运行 NEMU |
| `make run-qemu` | 运行 QEMU（可挂载 `MODEL_IMG`） |
| `make firmware` | 构建固件与 rootfs |
| `make pldm` | 打包 Palladium 版本（见[加速仿真](sim-acceleration.md#sec:sim-acceleration)） |
| `make run-fpga` | 部署并运行 FPGA（见[FPGA 上板与云平台](sim-fpga.md#sec:sim-fpga)） |
| `make test-smoke` | 快速环境自检 |
| `make test` | 完整环境自检（较慢） |

完整列表见 `make help`。

## 自检建议

1. 先 `make help` 与 `make -n <target>` 确认命令路径；注意 `run-*` 类目标内含递归 `$(MAKE)`，`-n` 并非绝对无副作用。
2. 用 `make test-smoke` 做快速验证。
3. 避免无必要的整机 RTL 重编译。

## 计算资源

RTL 仿真属重负载，请在**监控接口返回的在线低负载机器**上运行；并发按**物理核**计算（单个仿真 worker 占 8 个物理核，上限为物理核数 / 8），运行期间持续观察负载。

## 相关章节

* 仿真：[Verilator](sim-verilator.md#sec:sim-verilator)、[VCS](sim-vcs.md#sec:sim-vcs)、[加速仿真](sim-acceleration.md#sec:sim-acceleration)、[FPGA](sim-fpga.md#sec:sim-fpga)
* 排查：[常见故障排查](debug-troubleshooting.md#sec:debug-troubleshooting)
