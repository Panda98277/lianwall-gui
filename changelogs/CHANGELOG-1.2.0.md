# Changelog - v1.2.0

**发布日期**: 2026-02-23

## 概述

大规模质量修复与功能补全。SettingsPage 从仅有路径和外观两个分区扩展为涵盖全部 daemon 配置项的完整设置页面；修复了 DaemonClient 通信层、壁纸模型层、缩略图提供器等多个核心 bug；补全了全部 33 条缺失的英文翻译。

---

## ✨ 新功能

### 完整设置页面 (SettingsPage)

新增 5 个配置分区，覆盖 daemon 全部可配置项:

- **🔀 运行模式** — 设置 daemon 启动时的默认模式 (Video/Image)
- **🎬 动态壁纸引擎** — 切换间隔、目标显示器、mpvpaper/mpv 参数
- **🖼️ 静态壁纸引擎** — 切换间隔、输出目标、swww 参数
- **🎮 显存监控** — 启用开关、降级/恢复阈值、检测间隔、冷却时间
- **🔧 守护进程** — 日志级别选择

新增 3 个内联 UI 组件:

- `NumberInput` — 带范围验证的数字输入框
- `StyledTextInput` — Enter/失焦提交的文本输入框
- `StyledSlider` — 带数值显示的滑块

### 倒计时进度条 (CountdownBar)

- Dashboard 新增倒计时卡片，显示距下次自动切换的剩余时间
- 本地 Timer 每秒递减，daemon 事件驱动重置
- 接近切换时（≤60s）自动变为警告色

### 主色调切换

- 新增 MTF 蓝/MTF 粉两色切换，基于跨性别骄傲旗配色

---

## 🐛 修复

### P0 — 通信层

- **DaemonClient 错误响应空 payload** — 错误响应现在包含有意义的 `{code, message}` 而非空对象，避免 QML 层取到 undefined
- **WallpaperFilterModel.invalidateFilter() 未调用** — 搜索/筛选条件变化后现在正确触发过滤刷新

### P0 — 操作安全

- **toggleLock 回调使用固定行索引** — 异步回调期间列表可能已排序/过滤，改为按路径查找目标行
- **WallpaperDetailDialog 基于行索引操作** — 改为全部使用 `path` 作为操作标识，避免行号漂移导致操作错误壁纸

### P1 — 状态管理

- **Settings 模式按钮触发运行时切换** — `ConfigManager.setMode()` 错误调用 `SetMode`（运行时模式切换命令），改为 `SetConfig("paths.mode", ...)` 修改持久化配置。标签改为"默认模式"
- **VRAM 进度条在监控关闭时仍显示** — 条件从 `vramTotalMb > 0` 改为同时要求 `ConfigManager.vramEnabled`
- **VramChanged 事件降级状态判断错误** — `action == Keep` 时不再错误重置 degraded 标志，仅 Downgrade/Upgrade 改变状态
- **倒计时在关闭窗口再打开后失效** — `main.qml` 新增 `onVisibleChanged` 监听，窗口可见时调用 `DaemonState.refresh()` 刷新状态
- **LibraryPage 刷新触发不必要的 reloadConfig** — 刷新按钮移除 `daemonReloadConfig()` 调用，仅重新拉取壁纸列表
- **壁纸切换后 Library 不反映 isCurrent 变化** — 新增 `onWallpaperChanged` 监听自动刷新

### P2 — 缩略图

- **ThumbnailProvider 固定在第 0 秒截帧** — 改为使用 `Constants.h` 中定义的 `SEEK_SECONDS`（5 秒）
- **缩略图缓存不校验源文件变化** — 新增 mtime 对比，源文件更新后自动重新生成缩略图
- **重试参数索引错误** — `args[2]` 修正为 `args[3]`

---

## 🌐 国际化

- 补全 **33 条缺失的英文翻译**，涵盖:
  - Application 托盘菜单（显示/隐藏、下一张、上一张、切换模式、重启 Daemon、退出）
  - DashboardPage（切到图片/视频）
  - LibraryPage（刷新）
  - SettingsPage 全部新增区段
- 清理 26 条过时 (vanished) 翻译条目
- 最终状态: **104 translations, 104 finished, 0 unfinished**

---

## 📁 变更文件

| 文件 | 变更 |
|------|------|
| `CMakeLists.txt` | 版本号 1.1.0 → 1.2.0 |
| `src/Constants.h` | APP_VERSION 1.1.0 → 1.2.0 |
| `src/DaemonClient.cpp` | Error 响应包含 code/message |
| `src/DaemonState.cpp` | VramChanged 事件 degraded 判断修复 |
| `src/ConfigManager.cpp` | setMode() 改用 SetConfig 而非 SetMode |
| `src/WallpaperListModel.h` | 新增 byPath 系列方法声明 |
| `src/WallpaperListModel.cpp` | invalidateFilter、toggleLock 安全回调、byPath 方法实现 |
| `src/ThumbnailProvider.cpp` | seek 时间、mtime 校验、重试索引修复 |
| `qml/main.qml` | onVisibleChanged → DaemonState.refresh() |
| `qml/pages/DashboardPage.qml` | CountdownBar 卡片、VRAM 可见性条件 |
| `qml/pages/LibraryPage.qml` | 移除 reloadConfig、新增 wallpaperChanged 监听 |
| `qml/pages/SettingsPage.qml` | 5 个新配置分区 + 3 个内联组件 (555→976 行) |
| `qml/dialogs/WallpaperDetailDialog.qml` | 全部操作改为 path-based |
| `translations/lianwall-gui_en.ts` | 33 条新翻译 + 清理过时条目 |
| `translations/lianwall-gui_zh_CN.ts` | 同步补全 |
