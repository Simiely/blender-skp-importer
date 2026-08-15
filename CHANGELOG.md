# CHANGELOG.md · 变更记录

## [v1.1.3] - 2026-08-16

### 变更
- 明确材质处理**顺序:先烘焙程序贴图 → 再转材质**(实测更稳)——脚本的 vrMtlToPhysical 只把 VRayBitmap/VRayHDRI 转 Bitmap,VRayColor/VRayDirt/falloff 等 VRay 程序贴图原样保留;先烘焙(V-Ray 渲染器下)烘成位图后转换迁移干净,漏烘也有兜底
- docs/插件结合指南.md 路线 A/B 步骤调整为先烘焙再转换,并加「顺序要点」说明
- README 提示语、DEVELOPMENT 方案决策记录同步补充顺序说明

## [v1.1.2] - 2026-08-16

### 变更
- 明确 .max 进 Blender 的**最稳定方案**:Max 转材质(vray-material-replacer)→ 导出 FBX(勾选实例化 + 嵌入媒体)→ Blender 导入 FBX;io_scene_max 直读 .max 降为备选(无 Max 场景)
- docs/插件结合指南.md 重写第二节:路线总览(两条路)+ 路线 A FBX 详细步骤与注意事项 + 路线 B 直读 .max;第三节表格 MAX 行指向 FBX 路线
- README 快速开始提示改为最稳定方案(FBX 实例化)
- DEVELOPMENT.md 新增方案决策记录「FBX(勾选实例化)最稳定」

## [v1.1.1] - 2026-08-16

### 新增
- DEVELOPMENT.md 新增问题记录「VRay 代理(VRayProxy)模型导入丢失」:io_scene_max 不支持 `.vrmesh` 代理网格,导入后是空占位;含排查方法(MAXFILES.TXT grep vrmesh + .blend 统计空物体)与解决(3ds Max 转真实网格后重新导入)
- README 已知限制表补充 VRayProxy 解决指引
- docs/插件结合指南.md 补充实测案例(活力之丘点位模型:12 个 .vrmesh → 63 个空占位)

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
