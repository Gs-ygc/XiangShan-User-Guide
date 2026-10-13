# 环境准备 {#sec:sim-environment}

## xsai-env 总览

`xsai-env` 是香炉峰（XSAI）的统一开发环境，把 RTL、编译器、功能模拟器、固件、rootfs 与 FPGA 流程用一条 `make` 流程串起来。

TODO：补充仓库地址、目录结构与版本对应关系。

## 依赖与安装

TODO：补充主机依赖（工具链、Verilator/VCS、Python 版本）与一键安装步骤。

## 常用命令

| 命令 | 作用 |
| --- | --- |
| `make run-nemu` | 在 NEMU 上运行负载 |
| `make run-qemu` | 在 QEMU 上运行负载（支持 user/system 模式） |
| `make run-emu` | 运行预编译仿真器 |
| `make run-fpga` | 部署到 FPGA |
| `make pldm-build` | 构建 Palladium 版本 |

TODO：逐条补充参数、示例与输出说明。

## 目录与产物

TODO：补充负载（payload）、checkpoint、日志的存放位置约定。

## 常见问题

- 固件/负载体积较大时的传输耗时问题
- 依赖版本不匹配导致的构建失败

TODO：补充排查步骤，详见[@sec:debug-troubleshooting] [常见故障排查](debug-troubleshooting.md)。
