# AGENTS.md · 项目规则

> 📌 **文档基线**：2026-08-15（commit 待推送后回填）完成四件套创建
> **更新文档/代码后，请更新此行**（日期 + 新 commit hash），并在 CHANGELOG 追加版本

## 技术栈
- Blender 5.1.2（`C:\Program Files\Blender Foundation\Blender\`）与 5.2.0 LTS（`C:\Program Files\Blender Foundation\Blender 5.2\`）双安装，均内置 Python 3.13.13
- 插件 sketchup_importer 0.27.0（RedHaloStudio），自带 `sketchup.cp313-win_amd64.pyd` 匹配 Python 3.13
- SketchUp 文件支持到 2026.1.185

## 关键坑（务必先读）
- **Blender 5.x 节点名跟随界面语言本地化**：中文界面下 `nodes.new("ShaderNodeBsdfPrincipled")` 创建的节点 name 是「原理化 BSDF」，不要按英文名 `nodes["Principled BSDF"]` 查找——改用创建时返回的变量。socket 名（Base Color / Alpha / Surface）不受影响
- **Blender 传入的 filepath 是正斜杠**：不能用 `os.path.sep`（反斜杠）split 路径，会切不开导致整个路径被当文件名（WinError 123）
- **本机双 Blender**：插件需同步复制到两个版本的 `%APPDATA%\Blender Foundation\Blender\<v>\scripts\addons\`
- **大场景导入**：数百组件模型内存峰值 10GB+、耗时极长；后台 headless 导入时 mesh 阶段无日志输出，易误判卡死——用 `tasklist | grep blender` 看真实进程，不要用 `tasklist //FI`（Git Bash 转义问题）
- **下载别拿 Mac 版**：release 里 Mac 资产是 `sketchup.cpython-311-darwin.so` + `SketchUpAPI.framework`，Windows 加载不了

## 约定
- 注释与提交信息用中文
- GitHub token 只走环境变量（`GH_TOKEN`），不写入任何文件
- 文档四件套（README/AGENTS/DEVELOPMENT/CHANGELOG）随改动同步更新

## 常用命令
- 后台冒烟测试（Blender 5.1.2）：
  `"C:\Program Files\Blender Foundation\Blender\blender.exe" --background --python-expr "import bpy; bpy.ops.preferences.addon_enable(module='sketchup_importer'); print('OK')"`
- 后台导入测试：`--background --factory-startup --python-expr "import bpy; bpy.ops.preferences.addon_enable(module='sketchup_importer'); bpy.ops.import_scene.skp(filepath=r'<绝对路径>')"`

## 详细规则（按需 @引用）
- @rules/常见坑.md（内容超 150 词时拆出）
