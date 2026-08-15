# AGENTS.md · 项目规则

> 📌 **文档基线**：2026-08-16（commit `6bd98e7b`）v1.1.1：VRay 代理丢失问题记录
> **更新文档/代码后，请更新此行**（日期 + 新 commit hash），并在 CHANGELOG 追加版本

## 技术栈
- Blender 5.1.2（`C:\Program Files\Blender Foundation\Blender\`）与 5.2.0 LTS（`C:\Program Files\Blender Foundation\Blender 5.2\`）双安装，均内置 Python 3.13.13
- 插件 sketchup_importer 0.27.0（RedHaloStudio，仓库 `plugins/sketchup_importer.zip` 内含 2 个兼容修复，需解压安装），自带 `sketchup.cp313-win_amd64.pyd` 匹配 Python 3.13
- 扩展 io_scene_max 1.9.2（Blender 官方扩展仓库，.max 导入），仓库 `plugins/io_scene_max/` 内含本体
- SketchUp 文件支持到 2026.1.185；.max 需 Max 2015+ 保存

## 关键坑（务必先读）
- **Blender 5.x 节点名跟随界面语言本地化**：中文界面下 `nodes.new("ShaderNodeBsdfPrincipled")` 创建的节点 name 是「原理化 BSDF」，不要按英文名 `nodes["Principled BSDF"]` 查找——改用创建时返回的变量。socket 名（Base Color / Alpha / Surface）不受影响
- **Blender 传入的 filepath 是正斜杠**：不能用 `os.path.sep`（反斜杠）split 路径，会切不开导致整个路径被当文件名（WinError 123）
- **本机双 Blender**：插件/扩展需同步复制到两个版本的对应目录（addons 与 extensions 各一份）
- **Blender 4.2+ 扩展不是普通 addon**：安装目录是 `extensions\user_default\`（不是 `user\default`），启用模块名带命名空间前缀 `bl_ext.user_default.io_scene_max`（裸名 `io_scene_max` 会报 No module named）
- **io_scene_max 只认"链接纹理"**：贴图必须与 .max 同目录/子目录；VRayLight 灯光对象、VRayProxy 代理不会导入
- **VRay 代理(.vrmesh)导入即空**：io_scene_max 不支持代理网格,导入后是空 EMPTY 占位(实测 12 个 vrmesh → 63 个空物体)。排查:MAXFILES.TXT grep vrmesh;解决:Max 里转真实网格(Convert to Editable Poly)再导入
- **大场景导入**：数百组件模型内存峰值 10GB+、耗时极长；后台 headless 导入时 mesh 阶段无日志输出，易误判卡死——用 `tasklist | grep blender` 看真实进程，不要用 `tasklist //FI`（Git Bash 转义问题）
- **下载别拿 Mac 版**：release 里 Mac 资产是 `sketchup.cpython-311-darwin.so` + `SketchUpAPI.framework`，Windows 加载不了

## 约定
- 注释与提交信息用中文
- GitHub token 只走环境变量（`GH_TOKEN`），不写入任何文件
- 文档四件套（README/AGENTS/DEVELOPMENT/CHANGELOG）随改动同步更新
- 插件本体更新时同步替换 `plugins/` 下文件（保持仓库内插件可用）

## 常用命令
- 后台冒烟测试（Blender 5.1.2，SKP 插件）：
  `"C:\Program Files\Blender Foundation\Blender\blender.exe" --background --python-expr "import bpy; bpy.ops.preferences.addon_enable(module='sketchup_importer'); print('OK')"`
- 启用 .max 扩展（注意带命名空间前缀）：
  `"C:\Program Files\Blender Foundation\Blender\blender.exe" --background --python-expr "import bpy; bpy.ops.preferences.addon_enable(module='bl_ext.user_default.io_scene_max'); print('OK')"`
- 后台导入测试（SKP）：`--background --factory-startup --python-expr "import bpy; bpy.ops.preferences.addon_enable(module='sketchup_importer'); bpy.ops.import_scene.skp(filepath=r'<绝对路径>')"`

## 详细规则（按需 @引用）
- @rules/常见坑.md（内容超 150 词时拆出）
