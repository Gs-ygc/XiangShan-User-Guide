# 加速仿真（Palladium） {#sec:sim-acceleration}

## 流程总览

RTL 综合 → 网表映射到板卡资源 → 上板运行。其中"综合 + 工艺网表"与"网表映射、布局布线"是耗时最长的两步，通常在 20–30 分钟量级，因此增量编译与缓存尤为重要。

## 打包与部署

```bash
make pldm                 # 在一次性 git worktree 中构建并打包
# 可选：
#   PLDM_SKIP_BUILD=1     复用已有 build/
#   PLDM_KEEP_WORKTREE=1  保留 worktree
#   PLDM_COMPRESS=0       输出 .tar 而非压缩包
```

部署与运行在封闭区域进行：

1. 在开放区完成 RTL 编译（含 `PLDM=1` 等选项，且**不引入 difftest**，差分相关对照应先开放区完成）；
2. 通过 SFTP 传输到封闭区域（跨防火墙时以小机房为跳板）；
3. 在封闭区域激活 PLDM 环境后编译：

   ```bash
   make pldm-build CONFIG=xxx LLC=OpenLLC FPGA=1 PLDM=1 \
     WITH_CHISELDB=0 WITH_CONSTANTIN=0 DISABLE_DIFFTEST_RAM_DPIC=1 SYNTHESIS=1
   ```

   成功时输出 `FLOW SUCCESS`。
4. 运行：

   ```bash
   make pldm-run PLDM_EXTRA_ARGS="+no-diff +workload=/path/to/bin"
   ```

## 访问方式

统一走 `open10` 网关：

```bash
pldm doctor                                  # 检查链路
pldm run --target login01 --follow -- CMD    # 执行命令
pldm jobs | pldm logs JOB --follow | pldm cancel JOB
```

注意：`login01` 的 home 目录为 `noexec`，脚本请以 `/bin/sh <path>` 方式执行。

## 常见问题

| 现象 | 原因与处理 |
| --- | --- |
| 编译报 RTL syntax error | Chisel 生成 RTL 的残留片段；定位报错行并清理无意义内容后重试 |
| 运行卡住 | 先确认是否仍在推进（见[常见故障排查](debug-troubleshooting.md#sec:debug-troubleshooting)） |
| 需要差分比较 | 差分需在开放区完成；封闭区构建默认关闭 difftest |

## 与主机通信

Palladium 与主机之间的通信带宽可能成为瓶颈（实测约 200 MB/s 量级），差分验证的评估频率会受到限制；涉及大量逐指令交互的测试应评估该带宽上限。
