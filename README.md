# blender-skp-importer

> SketchUp 模型（.skp）导入 Blender 5.x 的完整方案：插件安装、兼容修复、纹理与实例处理。

把 SketchUp 的 `.skp` 模型导入 Blender，同时尽可能保留**组件实例**、**材质纹理**和**层级结构**。本仓库记录了经过实际验证的插件安装流程、Blender 5.x 兼容性修复，以及常见问题的排查方法。

## 特性

- ✅ 直接导入 `.skp`，组件 → Blender 实例（共享网格数据，编辑一个全部同步）
- ✅ 保留材质颜色与纹理贴图（实测 83 个纹理材质全部正确提取）
- ✅ 兼容 Blender 5.1 / 5.2（Python 3.13）
- ✅ 兼容 SketchUp 2026.1 及以下版本文件

## 快速开始

### 1. 下载 Windows 版插件

从 [RedHaloStudio/Sketchup_Importer](https://github.com/RedHaloStudio/Sketchup_Importer/releases) 下载最新 release 的 `sketchup_importer-<版本>.zip`（Windows 版）。

> ⚠️ 注意：release 页面同时提供 Mac 版，Windows 用户务必认准包含 `sketchup.cpXXX-win_amd64.pyd` 和 `SketchUpAPI.dll` 的包。

### 2. 安装到 Blender

将 `sketchup_importer` 文件夹复制到 Blender 的插件目录：

```
%APPDATA%\Blender Foundation\Blender\<版本>\scripts\addons\
```

如果你装了多个 Blender（如 5.1 和 5.2 并存），**每个版本的 addons 目录都要复制一份**。

### 3. 启用

- 打开 Blender → 编辑（Edit）→ 偏好设置（Preferences）→ 插件（Add-ons）
- 搜索 `sketchup` → 勾选 `Import-Export: SketchUp Importer`

### 4. 导入

- 文件（File）→ 导入（Import）→ SketchUp
- 选择 `.skp` 文件 → 等待导入完成

## 已知限制

| 项目 | 说明 |
|---|---|
| 大场景性能 | 数百组件的模型导入时内存峰值可达 10GB+，耗时较长，属正常现象 |
| 手动调整过的贴图 | SketchUp 里用「纹理 → 位置」手动移动/旋转过的贴图，导入后位置可能对不上（插件仅处理缩放比例） |
| 透明贴图 | 依赖贴图 Alpha 的透明（如镂空栏杆），导入后需手动将贴图 Alpha 接到材质 |

## 文档索引

- [AGENTS.md](AGENTS.md) — AI/协作者规则与关键坑（自动加载）
- [DEVELOPMENT.md](DEVELOPMENT.md) — 架构说明与问题记录（一坑一篇）
- [CHANGELOG.md](CHANGELOG.md) — 版本变更记录
