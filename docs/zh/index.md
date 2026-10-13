# 前言 {.unnumbered .unlisted}

本文档为《香炉峰用户手册》，适用于 {{processor_name}} 。香炉峰（Xianglufeng，项目代号 XSAI）是在香山昆明湖 V2R2A（KMHv2R2A）基础上扩展的、面向 AI 推理的 RISC-V 处理器：在保留 CPU 编程模型的前提下，增加了 AME 矩阵计算扩展、矩阵控制通路（AMU）、高带宽 L2（HBL2）等能力。

本手册**只描述香炉峰相对昆明湖 V2R2A 新增或修改的部分**，以及仿真、调试与软件开发的使用说明；与基线处理器相同的内容（特权模式、中断控制器、总线接口、内存模型基础语义等）请参见《香山开源处理器用户手册》。

您可从以下渠道获取最新版的本文档：

* 网页版：待发布
* 手册源码：[https://github.com/OpenXiangShan/XiangShan-User-Guide](https://github.com/OpenXiangShan/XiangShan-User-Guide)（分支 `xsai`）

您可访问香山文档网站 [docs.xiangshan.cc](https://docs.xiangshan.cc/) 获取更多文档。

## 阅读约定 {.unnumbered .unlisted}

* 本手册仍在持续完善中，带 `TODO` 标记的段落表示内容尚未定稿。
* 指令编码、CSR 编号、寄存器位域等以最新版 ISA 规范与 RTL 为准。
* 文中示例命令以 `xsai-env` 统一环境为前提，见《环境准备》一章。

## 声明 {.unnumbered .unlisted}

版权所有 © 2024 - 2026 香山开源处理器团队·北京开源芯片研究院

本文档采用“[知识共享 署名 4.0](https://creativecommons.org/licenses/by/4.0/deed.zh-hans)”协议公开发布。您可以根据该协议的规定，自由地使用、修改、分发本文档，但必须给出恰当的署名、表明原始许可协议、标注是否对原始文档作了修改。具体许可条款以知识共享组织公布的完整法律文本为准。

本文档仅提供阶段性信息，可能不定期根据实际情况更新。除非另有约定，本文档中的所有陈述、信息和建议不构成任何明示或暗示的担保。

## 版本历史 {.unnumbered .unlisted}

| 日期       | 版本   | 说明               |
| ---------- | ------ | ------------------ |
| 2026-10-13 | draft1 | 第一版框架与增量章节 |
