> [!IMPORTANT]
> ## 📦 本仓库已归档（只读）—— 最新版请到统一插件集获取
> SketchUp Importer 插件已迁入 **[Simiely/blender-addons](https://github.com/Simiely/blender-addons)**，
> 后续的版本更新与问题修复**只在那边进行**，本仓库**已于 2026-10-09 归档、只读**。
>
> 最新版下载：https://github.com/Simiely/blender-addons/tree/main/addons
>
> 本仓库**保留历史代码与文档作为参考**，`plugins/sketchup_importer.zip` 与插件集内的对应文件**逐字节一致**；
> 请不要在此提交 Issue / PR —— 那边才是唯一维护处。
>
> ⚠️ 注意：它是**包目录形态**（含 cp37~cp314 多版 `.pyd` 与 `SketchUpAPI.dll`），
> 安装时需**解压**到 Blender 的 `scripts/addons/`，不能按单文件 `.py` 方式安装。
> 详细步骤见插件集 README 的「SketchUp Importer」章节。
>
> 本仓库的**文档内容仍然有效** —— SketchUp / 3ds Max 导入的兼容修复说明、
> 纹理与实例处理、常见问题排查仍在这里查阅。

# blender-skp-importer

> SketchUp（`.skp`）与 3ds Max（`.max`）模型导入 Blender 5.x 的完整方案：插件安装、兼容修复、纹理与实例处理。

把 SketchUp 的 `.skp` 模型导入 Blender，同时尽可能保留**组件实例**、**材质纹理**和**层级结构**；并支持通过官方扩展把 3ds Max 的 `.max` 文件直接导入。本仓库记录了经过实际验证的插件安装流程、Blender 5.x 兼容性修复、双插件结合工作流，以及常见问题的排查方法。

## 特性

- ✅ 直接导入 `.skp`，组件 → Blender 实例（共享网格数据，编辑一个全部同步）
- ✅ 保留材质颜色与纹理贴图（实测 83 个纹理材质全部正确提取）
- ✅ 中文界面正常导入 UV（修复上游按英文名判 UV 的 bug，非英文界面此前会静默丢全部 UV，v1.2.0）
- ✅ 直接导入 `.max`（官方扩展，无需安装 3ds Max），支持 V-Ray/Corona/Physical 材质部分转换
- ✅ 兼容 Blender 5.1 / 5.2（Python 3.13）
- ✅ 兼容 SketchUp 2026.1 及以下版本文件

## 仓库结构

```
blender-skp-importer/
├── README.md                # 本文档：安装与快速开始
├── AGENTS.md                # AI / 协作规则与关键坑
├── DEVELOPMENT.md           # 架构说明与问题记录
├── CHANGELOG.md             # 版本变更记录
├── docs/
│   └── 插件结合指南.md        # 双插件结合使用，让导入更稳定
└── plugins/                 # 打包好的插件本体（可直接安装）
    ├── sketchup_importer.zip# SKP 导入插件 zip（解压即得完整插件，已含兼容修复）
    └── io_scene_max/        # .max 导入扩展（官方 v1.9.2）
```

## 快速开始 · SketchUp（.skp）

### 1. 安装插件

解压本仓库 `plugins/sketchup_importer.zip` 得到 `sketchup_importer` 文件夹（已含兼容修复），也可从 [RedHaloStudio/Sketchup_Importer](https://github.com/RedHaloStudio/Sketchup_Importer/releases) 下载原版。

> ⚠️ 注意：release 页面同时提供 Mac 版，Windows 用户务必认准包含 `sketchup.cpXXX-win_amd64.pyd` 和 `SketchUpAPI.dll` 的包。

将 `sketchup_importer` 文件夹复制到 Blender 的插件目录：

```
%APPDATA%\Blender Foundation\Blender\<版本>\scripts\addons\
```

如果你装了多个 Blender（如 5.1 和 5.2 并存），**每个版本的 addons 目录都要复制一份**。

### 2. 启用并导入

- 打开 Blender → 编辑（Edit）→ 偏好设置（Preferences）→ 插件（Add-ons）
- 搜索 `sketchup` → 勾选 `Import-Export: SketchUp Importer`
- 文件（File）→ 导入（Import）→ SketchUp → 选择 `.skp` 文件

## 快速开始 · 3ds Max（.max）

### 1. 安装扩展

将 `plugins/io_scene_max/` 复制到 Blender 的扩展目录（每个版本一份）：

```
%APPDATA%\Blender Foundation\Blender\<版本>\extensions\user_default\
```

### 2. 启用并导入

- 偏好设置 → 插件（Add-ons）→ 搜索 `Autodesk MAX` → 勾选 `Import-Export: Import Autodesk MAX (.max)`
- 文件（File）→ 导入（Import）→ Autodesk MAX → 选择 `.max` 文件

> 💡 **最稳定方案(实测)**:在 3ds Max 里用 [vray-material-replacer](https://github.com/Simiely/vray-material-replacer) 处理材质(**顺序:先烘焙程序贴图 → 再转 Physical**,见指南)→ **导出 FBX 并勾选「实例化 Instancing + 嵌入媒体」** → Blender 导入 FBX——实例、材质、贴图都能保留。完整流程与备选路线(io_scene_max 直读 .max)见 [docs/插件结合指南.md](docs/插件结合指南.md)。

## 已知限制

| 项目 | 说明 |
|---|---|
| 大场景性能（SKP） | 数百组件的模型导入时内存峰值可达 10GB+，耗时较长，属正常现象 |
| 手动调整过的贴图（SKP） | SketchUp 里用「纹理 → 位置」手动移动/旋转过的贴图，导入后位置可能对不上（插件仅处理缩放比例） |
| 透明贴图 | v1.2.0 已修复：此前中文界面 UV 静默丢失 → 贴图退化为单点采样（alpha=0 处整片全透明）；UV 修复后贴图 Alpha 自动生效 |
| `.max` 贴图路径 | 只认"链接纹理"，贴图必须与 .max 同目录或子目录，否则变紫 |
| `.max` 灯光/代理 | VRayLight 灯光对象不会导入,需重打灯;VRayProxy 代理网格(`.vrmesh`)会丢,需先在 3ds Max 里转成真实网格(Convert to Editable Poly)再导入,排查与解决见 [DEVELOPMENT.md](DEVELOPMENT.md) |
| `.max` 版本 | 需 3ds Max 2015+ 保存；压缩格式仅支持 .zip 且 Max ≤2023 |

## 文档索引

- [AGENTS.md](AGENTS.md) — AI/协作者规则与关键坑（自动加载）
- [DEVELOPMENT.md](DEVELOPMENT.md) — 架构说明与问题记录（一坑一篇）
- [CHANGELOG.md](CHANGELOG.md) — 版本变更记录
- [docs/插件结合指南.md](docs/插件结合指南.md) — 双插件结合使用，让 .max 导入更稳定
