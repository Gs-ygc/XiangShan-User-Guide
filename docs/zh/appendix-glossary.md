# 附录 B：术语表 {#sec:appendix-glossary}

| 缩写 | 全称 | 说明 |
| --- | --- | --- |
| AME | Attached Matrix Extension | 附着的矩阵扩展 |
| AMU | AME Management Unit | 矩阵控制通路，负责矩阵单元的访存与命令管理 |
| CUTE | — | 矩阵计算单元 |
| HBL2 | High Bandwidth L2 | 高带宽二级缓存 |
| LLC | Last Level Cache | 末级缓存 |
| MSC | Matrix System Control | 矩阵系统控制基址，软件侧需钉住其基址寄存器 |
| tile | — | 矩阵分块，香炉峰只有 default / mini 两套原生几何 |
| difftest | — | 差分验证 |
| UB | Undefined Behaviour | 未定义行为 |
| checkpoint | — | 程序状态快照，用于切片仿真 |
| GCPT | — | 用于快速启动的程序镜像格式 |
| 尾包 | tail packet | 像素数少于 `DST_XUNIT` 的最后一个包，布局为紧凑排布 |
| 工作量门禁 | workload guard | 以 `oc × K` 阈值决定是否交给矩阵单元 |
| 位级一致 | bit-level identical | 矩阵开关开、关下输出逐字节相同的判据 |

TODO：补充完整术语。
