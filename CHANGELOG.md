# CHANGELOG.md · 变更记录

## [v1.1.0] - 2026-08-16

### 新增
- `plugins/` 目录:打包两个插件本体,开箱即用
  - `plugins/sketchup_importer.zip`(SKP 导入,解压即得完整插件,含本仓库 2 个 Blender 5.x 兼容修复;因 GitHub Contents API 大文件请求被网络层拦截,以 zip 形式存储)
  - `plugins/io_scene_max/`(.max 导入,Blender 官方扩展 v1.9.2)
- `docs/插件结合指南.md`:io_scene_max 与 vray-material-replacer 结合,让 .max 导入更稳定;SKP + MAX 双格式统一工作流
- README 增加 3ds Max(.max)导入快速开始、仓库结构、.max 相关已知限制

### 变更
- README 从单 SKP 方案扩展为 SKP + MAX 双格式方案

## [v1.0.0] - 2026-08-15

初始发布，四件套文档就位。

### 新增
- 仓库建立，文档四件套（README / AGENTS / DEVELOPMENT / CHANGELOG）
- 记录 Windows + Blender 5.x 安装 sketchup_importer 0.27.0 的完整流程（含双 Blender 版本处理）
- 记录并修复 2 个 Blender 5.x 兼容 bug：
  - Principled BSDF 节点名本地化（中文界面「原理化 BSDF」）导致 KeyError
  - Blender filepath 正斜杠与 `os.path.sep` 不匹配导致 WinError 123
- 纹理提取验证：83 个纹理材质全部正确提取（md5 校验无串图）
- 明确已知边界：手动调整过的贴图位置/旋转、透明 Alpha 自动接线
