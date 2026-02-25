# Changelog - v1.4.0

**发布日期**: 2026-02-26

**配套**: lianwall ≥ 5.3.0

## 概述

跟进 lianwall 5.3.0 的 VRAM 扩展功能。设置页"显存监控"区块新增**后端选择器**和**自定义命令输入框**，支持用户配置 Intel 等非主流 GPU 的自定义显存查询脚本。

---

## 🔵 新功能

### 1. VRAM 后端选择器

**新增**: 设置页"显存监控"区块新增"后端"下拉选择框（`StyledSelect`），对应 `vram.backend` 配置项：

| 选项 | 对应值 | 说明 |
|------|--------|------|
| Auto（自动检测） | `"auto"` | 自动探测 nvidia-smi / rocm-smi（原有默认行为） |
| Custom（自定义） | `"custom"` | 使用下方"自定义命令"中指定的 Shell 脚本 |

选择后实时通过 `SetConfig` 发送给 daemon，无需重启。

### 2. 自定义命令输入框

**新增**: 设置页"显存监控"区块新增"自定义命令"文本框（`StyledTextInput`），对应 `vram.custom_command` 配置项：

- 仅在后端选择为 `Custom` 时显示（`visible: vramBackend === "custom"`）
- 占位文字：`~/.config/lianwall/intel_vram.sh`
- 失去焦点或按 Enter 时触发 `SetConfig`

---

## 🟣 改进

### 3. ConfigManager 新增 vramBackend / vramCustomCommand 属性

`ConfigManager` 新增两个 `Q_PROPERTY`：

| 属性 | 类型 | 说明 |
|------|------|------|
| `vramBackend` | `QString` | 当前后端值（`"auto"` 或 `"custom"`） |
| `vramCustomCommand` | `QString` | 当前自定义命令字符串 |

均通过事件驱动更新（`ConfigChanged` 推送）：
- `applyFullConfig()` 从 JSON 的 `vram.backend` / `vram.custom_command` 字段解析
- `applySingleKey()` 响应 daemon 的 `SetConfig` 确认回调，实时同步

---

## 📁 变更文件清单

| 文件 | 变更 |
|------|------|
| `CMakeLists.txt` | 版本号 1.4.0 |
| `src/Constants.h` | 版本号 1.4.0 |
| `src/ConfigManager.h` | 新增 `vramBackend`、`vramCustomCommand` 属性、getter、setter 声明、signal、成员变量 |
| `src/ConfigManager.cpp` | `applyFullConfig()` / `applySingleKey()` 解析新字段；实现 `setVramBackend()` / `setVramCustomCommand()` |
| `qml/pages/SettingsPage.qml` | VRAM 区块新增后端 `StyledSelect` 和 `StyledTextInput` |
