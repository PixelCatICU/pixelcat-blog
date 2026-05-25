---
title: PixelCat Pop 使用指南：macOS 菜单栏工具箱
date: 2026-05-25 10:00:00
updated: 2026-05-25 10:00:00
description: PixelCat Pop 是一个原生 macOS 菜单栏工具箱，基于 SwiftUI、AppKit、ScreenCaptureKit 和 Apple Translation，集成双击 Command-C 翻译、输入翻译、截图标注、屏幕录制、应用清理和可选菜单栏监控，并说明安装构建、系统权限、语言包、录屏导出和 GPLv3 开源协议。
tags:
  - PixelCat Pop
  - macOS
  - 菜单栏工具
  - 截图标注
  - 屏幕录制
  - Apple Translation
---

PixelCat Pop 是「像素猫 - 科学上网ICU」做的原生 macOS 菜单栏工具箱。它不是一个只做翻译的小浮窗，而是把 Mac 上经常临时需要的几个动作放进菜单栏：翻译、截图标注、屏幕录制、应用清理和轻量系统监控。

它的设计目标很明确：平时不打扰，需要时从菜单栏打开；能按需加载的功能就按需加载；系统监控、录音、交互效果这些会增加资源占用的功能，尽量放到设置里显式开启。

> 本文基于 PixelCat Pop 当前 README 和本地源码整理。项目仍处于早期迭代阶段，实际功能以 GitHub 仓库为准。

## 项目入口

- GitHub 项目：[PixelCatICU/pixelcat-pop](https://github.com/PixelCatICU/pixelcat-pop)
- 官网：[pixelcat.icu](https://pixelcat.icu)
- YouTube：[@PixelCatICU](https://www.youtube.com/@PixelCatICU)
- X：[@PixelCatICU](https://x.com/PixelCatICU)
- 开源协议：GNU General Public License v3.0

## 一、PixelCat Pop 适合解决什么问题

如果你在 Mac 上经常遇到这些场景，PixelCat Pop 会比较顺手：

- 复制一段英文或中文，想立刻看翻译结果。
- 截图之后要画矩形、箭头、文字，或者给局部打马赛克。
- 想录一小段屏幕操作，并直接导出 MP4。
- 卸载应用时想顺手检查偏好设置、缓存、日志、容器目录等残留文件。
- 只在需要时才显示 CPU、内存、磁盘或网络状态，不想常驻一个复杂监控面板。

这些功能都放在一个菜单栏 App 里，不需要为每个小需求单独打开一个大软件。

## 二、核心功能一览

| 功能 | 说明 |
| --- | --- |
| 双击 `Command-C` 翻译 | 监听剪贴板文本，连续复制两次后弹出翻译面板 |
| 输入翻译 | 从菜单栏打开输入面板，手动输入内容并自动翻译 |
| 智能目标语言 | 中文和英文优先互译，其他语言翻译到默认目标语言 |
| 固定目标语言 | 可在设置里改成始终翻译到指定语言 |
| 翻译历史 | 可保留最近翻译记录，也可以关闭或清空 |
| 截图标注 | 选择区域或吸附窗口后添加矩形、箭头、文字和马赛克 |
| 屏幕录制 | 使用和截图一致的区域选择体验，导出 HEVC MP4 |
| 录制音频 | 可分别开启系统声音和麦克风声音 |
| 交互效果 | 录制时可开启点击声、键盘声、点击波纹和输入区域 Zoom |
| 应用清理 | 扫描应用本体和常见残留目录，确认后移到废纸篓 |
| 菜单栏监控 | CPU、内存、磁盘、网络默认关闭，可按需开启 |
| 多语言界面 | 设置界面支持中文、English 和跟随系统 |

## 三、菜单栏入口

安装并启动后，PixelCat Pop 会出现在 macOS 菜单栏。主要入口包括：

- 输入翻译
- 翻译剪贴板
- 截图标注
- 开始 / 停止录屏
- 应用清理
- 设置
- 退出

Debug 构建里会额外出现“重启”菜单项，用于开发阶段快速重启；正式 Release 包不会依赖这个入口。

## 四、翻译：双击 Command-C 和手动输入

PixelCat Pop 的翻译能力基于 Apple Translation。常用方式有两种：

1. 选中文本后连续按两次 `Command-C`，弹出翻译面板。
2. 从菜单栏打开“输入翻译”，手动输入内容。

语言路由分为两种模式：

| 模式 | 行为 |
| --- | --- |
| 智能模式 | 中文翻译为英语，英文翻译为中文，其他语言翻译到默认目标语言 |
| 固定模式 | 所有文本都翻译到设置中选择的目标语言 |

当前目标语言选项包括中文、英语（美国）、日语、韩语、法语、德语和西班牙语。

需要注意的是，Apple Translation 的可用语言由 macOS 管理。如果翻译一直加载，或者提示语言不可用，先打开系统自带“翻译”应用，或到系统语言设置里安装对应语言包。

## 五、截图标注：区域选择、窗口吸附和马赛克

从菜单栏选择“截图标注”后，会出现全屏选择层。你可以自由拖拽选区，也可以把鼠标移动到窗口上，让 PixelCat Pop 自动吸附并高亮窗口。

进入编辑器后，可以添加：

- 矩形
- 箭头
- 文字
- 马赛克

标注结果可以复制到剪贴板，也可以保存为 PNG。马赛克强度支持实时调整，修改强度时已选中的马赛克标注会立即更新。

这类体验适合写教程、报错反馈、远程协作和隐私遮挡：不用先截图，再打开另一个图像编辑器处理。

## 六、屏幕录制：和截图一致的选择体验

PixelCat Pop 的录屏入口复用截图的选择逻辑：

- 点击吸附窗口，录制该窗口区域。
- 拖拽区域，录制自定义选区。
- 再次点击菜单里的“停止录屏”，结束录制。

录制文件默认保存到桌面，文件名类似：

```text
PixelCatPop-Recording-YYYYMMDD-HHMMSS.mp4
```

视频容器是 `.mp4`，视频编码使用 H.265/HEVC，音频使用 AAC。Retina 屏幕会按显示器物理像素写入，避免只按逻辑点尺寸录制导致画面发糊。

如果开启系统声音、麦克风、点击波纹、键盘输入声或输入区域 Zoom，录制时的资源消耗会增加。录屏区域越大、帧率越高，对 CPU、GPU 和硬件编码器的压力也越明显。

## 七、应用清理：确认后移到废纸篓

应用清理功能参考 AppCleaner 的常见流程，但保持了一个保守原则：只在用户主动选择或拖入 `.app` 后扫描，展示候选文件，确认后移到系统废纸篓，不直接永久删除。

当前扫描范围包括：

```text
应用本体
~/Library/Application Support
~/Library/Caches
~/Library/Preferences
~/Library/Saved Application State
~/Library/Logs
~/Library/Containers
~/Library/Group Containers
```

扫描结果会列出来，你可以取消勾选不想删除的项目。对于正在使用、数据重要或不确定用途的应用，建议先保留相关文件，确认无误后再清理。

## 八、菜单栏监控：默认关闭，按需开启

PixelCat Pop 的系统监控不是默认全开。CPU、内存、磁盘和网络显示默认关闭，只有在设置里开启对应项目后才会定时刷新。

图标默认保持系统模板色。如果打开“图标随状态变色”，可以选择按 CPU 或内存占用改变图标颜色。

这个默认值比较适合菜单栏工具箱：不需要监控时就不采样，需要观察状态时再开启。

## 九、系统要求和权限

PixelCat Pop 依赖较新的 macOS 系统能力：

| 项目 | 要求 |
| --- | --- |
| 系统版本 | macOS 15 或更新版本 |
| 截图标注 | `SCScreenshotManager`，需要 macOS 15.2 或更新版本 |
| 构建工具 | Xcode / Swift 6 工具链 |
| 翻译 | 已安装 Apple Translation 对应语言包 |

可能涉及的系统权限：

- 辅助功能权限：用于全局监听双击 `Command-C`。
- 屏幕录制权限：用于截图和录屏。
- 麦克风权限：开启麦克风录制时需要。
- Apple Translation 语言包：由 macOS 系统管理。

建议日常使用打包后的 `.app`，因为 Apple Translation、屏幕录制权限和麦克风权限在正式 App 包形态下更接近真实运行环境。

## 十、从源码构建和打包

如果你从 GitHub 拉源码，可以使用完整 Xcode 工具链：

```bash
export DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer
```

构建 Debug：

```bash
swift build
```

运行测试：

```bash
swift test
```

打包 Release App：

```bash
Scripts/package-app.sh
```

打包后的应用位于：

```text
dist/PixelCatPop.app
```

运行打包后的 App：

```bash
open dist/PixelCatPop.app
```

也可以直接用 SwiftPM 运行：

```bash
swift run PixelCatPop
```

## 十一、项目结构

PixelCat Pop 当前仍保留单个 SwiftPM target，功能按目录拆分：

```text
Sources/PixelCatPop/App/                 应用入口、AppDelegate、菜单栏生命周期
Sources/PixelCatPop/AppCleaner/          应用卸载残留扫描、勾选确认和移到废纸篓
Sources/PixelCatPop/Clipboard/           剪贴板读取和双击 Command-C 触发监听
Sources/PixelCatPop/InteractionEffects/  鼠标点击声、键盘输入声、波纹和 Zoom 效果
Sources/PixelCatPop/Recording/           录屏、系统声音、麦克风和视频写入
Sources/PixelCatPop/Screenshot/          截图选择、窗口吸附、截图捕获和标注编辑器
Sources/PixelCatPop/Settings/            设置存储和设置界面
Sources/PixelCatPop/SystemMonitor/       CPU、内存、磁盘、网络采样和菜单栏展示
Sources/PixelCatPop/Translation/         翻译服务、语言路由、翻译面板和历史记录
Tests/PixelCatPopTests/                  按功能域组织的测试
Scripts/package-app.sh                   macOS .app 打包脚本
```

这样可以先降低早期模块拆分成本，等翻译、截图、录屏、清理和监控边界稳定后，再考虑拆成独立 library target。

## 十二、适合怎样使用

PixelCat Pop 更像一个“常用动作入口”，而不是一个全功能截图软件、专业录屏软件或大型系统监控面板。

推荐用法是：

1. 平时常驻菜单栏，只保留低打扰入口。
2. 翻译用双击 `Command-C` 或输入面板。
3. 临时截图讲解时用截图标注。
4. 录制短教程或问题复现时用区域录屏。
5. 卸载应用时用应用清理检查残留。
6. 需要观察机器状态时再打开菜单栏监控。

如果你喜欢把这些小工具集中在一个原生菜单栏 App 里，PixelCat Pop 的方向就是减少切换成本，同时尽量不在后台做多余的事情。

