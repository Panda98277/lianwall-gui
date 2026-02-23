# Changelog - v1.3.0

**发布日期**: 2026-02-23

## 概述

修复 Daemon 命令错误响应被静默丢弃的问题，新增全局错误弹窗；精细化 Dashboard 预览区占位文字（区分 4 种状态）；修复预览框在宽窗口时高度不受约束的布局问题。

---

## 🐛 修复

### P0 — Daemon 命令错误静默丢弃

**问题**: `daemonNext()`、`daemonPrev()`、`daemonSetMode()` 调用 DaemonClient 时传入空回调，守护进程返回的 `Error` 响应被完全丢弃，用户和日志均无任何提示，导致操作失败后界面无反馈。

**修复**:
- `Application::daemonNext()` / `daemonPrev()` / `daemonSetMode()` 现在传入错误回调
- Error 响应触发 `qWarning()` 日志 + `DaemonState::daemonError()` 信号
- `DaemonState` 中已有的 Error 事件处理修正为使用 `errorCodeToString()` 而非空字符串

**新增**: `DaemonTypes.h` 补充 `errorCodeToString()` 反向映射函数（与现有 `errorCodeFromString()` 对称）

**修改文件**: `src/DaemonTypes.h`, `src/DaemonState.cpp`, `src/Application.cpp`

---

### P0 — 命令错误无 UI 反馈

**问题**: 即使添加了错误回调，`daemonError` 信号之前在 QML 层没有任何监听，用户仍然看不到错误。

**修复**: `main.qml` 新增红色顶部横幅 Popup，监听 `DaemonState.daemonError` 信号：
- 显示错误消息文本
- 5 秒后自动关闭
- 支持点击 ✕ 手动关闭
- 非模态，不阻断操作

**修改文件**: `qml/main.qml`

---

### P1 — Dashboard 预览占位文字过于笼统

**问题**: 预览区占位文字只有两种状态："加载中..."（有路径时）和"暂无壁纸"（无路径时），无法区分守护进程未连接、文件缺失、缩略图生成失败等情况。尤其是 `Image.Error` 状态下永远显示"加载中..."而不提示错误。

**修复**: 占位文字和图标扩展为 4 种精细状态：

| 状态 | 图标 | 文字 |
|------|------|------|
| 守护进程未连接 | 🔌 | 守护进程未连接 |
| 已连接，无壁纸路径 | 🖼️ | 暂无壁纸 |
| 有路径，但图片/缩略图加载失败 | ⚠️ | 预览加载失败 |
| 有路径，正在异步加载 | 🎬/🖼️ | 加载中... |

**修改文件**: `qml/pages/DashboardPage.qml`

---

### P1 — 预览框高度在宽窗口时不受约束

**问题**: 预览框高度固定为 `width * 9/16`，窗口拉宽后高度随之增长，在大窗口或最大化时预览占满大半屏幕，挤压下方内容。

**修复**:
- 高度改为 `width * Screen.height / Screen.width`（跟随当前屏幕实际宽高比）
- 新增 `Layout.maximumHeight: 360` 硬上限，超出后不再增长
- `fillMode` 从 `PreserveAspectCrop` 改为 `PreserveAspectFit`，防止图片在容器尺寸限制后溢出裁剪区域

**修改文件**: `qml/pages/DashboardPage.qml`（新增 `import QtQuick.Window`）

---

## 📁 变更文件清单

| 文件 | 变更类型 |
|------|----------|
| `CMakeLists.txt` | 版本号 1.3.0 |
| `src/Constants.h` | 版本号 1.3.0 |
| `src/DaemonTypes.h` | 新增 `errorCodeToString()` |
| `src/DaemonState.cpp` | Error 事件使用 `errorCodeToString()` |
| `src/Application.cpp` | `daemonNext` / `daemonPrev` / `daemonSetMode` 添加错误回调 |
| `qml/main.qml` | 新增错误横幅 Popup |
| `qml/pages/DashboardPage.qml` | 预览占位精细化、高度限制、AspectFit |
