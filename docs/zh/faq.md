# 常见问题与已知限制 {#sec:faq}

## 已知限制

* 裁剪配置与完整配置在行为、性能上存在差异，比较时必须固定配置；
* 矩阵访存 miss 通路长期未被软件触发，相关验证仍在补充；
* 低精度格式与部分自定义指令仍在推进中；
* 裁剪配置下存在向量指令特定场景卡死问题，完整配置不复现。

## 仿真相关

* RTL 下 Linux 启动异常缓慢 → 检查 `timebase-frequency`（见[常见故障排查](debug-troubleshooting.md#sec:debug-troubleshooting)）；
* 云平台 job 长时间 `pending` → 多为正常排队，不必重推；
* 仿真器导不出波形 → 确认编译期已启用 `EMU_TRACE`。

## 构建相关

* rootfs 产物 Nix / 非 Nix 混用导致 patchelf 报错 → 清理后重建；
* 内核 panic `No working init found` → 重建 rootfs。

## 与基线手册的关系

本手册只写增量；未列出的内容默认与昆明湖 V2R2 一致，请查阅《香山开源处理器用户手册》。
