# DEVELOPMENT.md · 开发说明

## 项目概览

目标：在 Windows + Blender 5.x 上，把 SketchUp `.skp` 模型（含组件实例、材质纹理）无损导入 Blender。

技术路线：选用 RedHaloStudio/Sketchup_Importer 0.27.0 —— 基于 SketchUp C API 的 Cython 绑定（`sketchup.cp313-win_amd64.pyd`），在 Blender 内置 Python 内直接读取 SKP 文件，不经过任何中间格式。

## 架构说明

```
.skp 文件
   ↓ sketchup.Model.from_file()   （SketchUp C API）
sketchup.cp313-win_amd64.pyd      （Cython 绑定，匹配 Blender 5.x 的 Python 3.13）
   ↓
SceneImporter.load() 分四阶段：
   1. write_camera()   场景/相机
   2. write_materials() 材质 + 纹理（tex.write → 临时目录 → bpy.data.images.load → pack）
   3. write_components() 组件定义遍历（proxy_dict 处理嵌套）
   4. write_entities()  网格对象 + UV（f.st_scale 缩放 + tessfaces UV 坐标）
   ↓
Blender 场景（组件 → 共享网格数据的实例对象，层级保留）
```

关键点：
- 纹理处理：SketchUp 纹理 `tex.write(path)` 写出原始位图 → 临时目录 → `bpy.data.images.load()` → `img.pack()` 打包进 blend 后删除临时目录
- UV 处理：`f.st_scale = materials_scales[材质名]` 设置面纹理缩放，`f.tessfaces` 取三角面 UV，写入 `me.uv_layers[0]`
- 实例处理：组件定义写一次，多次出现的实例通过共享 mesh 数据块实现

## 关键问题与方案

### 问题：Blender 5.x 导入报 KeyError「Principled BSDF not found」

**TL;DR**：Blender 5.x 节点默认 name 跟随界面语言，中文界面下为「原理化 BSDF」，插件写死英文名查找失败。

- 问题：`nodes["Principled BSDF"]` 抛 `KeyError: key "Principled BSDF" not found`
- 根因：Blender 5.x 起节点名（name）按界面语言本地化；实测 `nodes.new("ShaderNodeBsdfPrincipled")` 返回的节点 name 是「原理化 BSDF」（中文界面）
- 解决：删除按名字查找，直接用创建时返回的变量 `default_shader = principled_bsdf`（socket 名如 Base Color/Alpha 实测不受本地化影响）
- 预防：插件代码内不要用 `nodes["<英文节点名>"]` 反查刚创建的节点

### 问题：导入报 OSError WinError 123 文件名语法错误

**TL;DR**：Blender 传入的 filepath 用正斜杠 `/`，插件用 `os.path.sep`（`\`）split 切不开，整个路径被当文件名拼进临时目录。

- 问题：`os.mkdir(temp_dir)` 抛 `OSError: [WinError 123]`，temp_dir 形如 `C:\Temp\E:/Desktop/...`
- 根因：`self.filepath.split(os.path.sep)[-1]` 在 Windows 上按 `\` 切分正斜杠路径无效；`tex.name.split(os.path.sep)` 同理
- 解决：统一先 `replace('\\', '/')` 再按 `/` 切分；取 basename 用 `os.path.basename(self.filepath.replace('\\','/'))`
- 预防：Windows 上处理 Blender 路径一律先归一化为 `/`

### 问题：下载到 Mac 版插件无法加载

- 问题：插件在 Blender 里不显示 / 加载报错
- 根因：release 资产误取 Mac 版（`sketchup.cpython-311-darwin.so` + `SketchUpAPI.framework`），Windows 无法加载；且其绑定 Python 3.11 与 Blender 5.x 的 3.13 不匹配
- 解决：Windows 用 RedHaloStudio release 的 zip（内含 `sketchup.cp313-win_amd64.pyd` + `SketchUpAPI.dll`）
- 预防：下载前核对资产内是否含 `win_amd64.pyd` 与 `SketchUpAPI.dll`

### 问题：纹理"错误"排查（最终排除提取环节）

- 问题：用户反馈导入后"很多贴图错误"
- 排查：用插件的 sketchup API 提取全部 83 个带纹理材质的图片做 md5 哈希对比——内容全部正确、无串图（重名纹理哈希一致属正常复用）；355 个纯色材质无纹理属正常
- 结论：提取环节无 bug；若个别贴图显示异常，指向 SketchUp 内手动调整过位置/旋转的贴图（插件未处理每面纹理 origin/rotation），或视图模式/透明 Alpha 问题
- 验证方法：遍历 `Model.materials` → `mat.texture.write(文件)` → 按文件名分组比较 md5

### 问题：本机双 Blender 只装了一边

- 问题：5.1.2 里能看到插件，5.2 里看不到
- 根因：Program Files 下 `Blender\`（5.1.2）与 `Blender 5.2\` 是两个独立安装，插件只复制到了 5.1 的 APPDATA addons 目录
- 解决：两个版本的 `%APPDATA%\Blender Foundation\Blender\<v>\scripts\addons\` 都要复制插件
- 预防：安装插件前先确认本机所有 Blender 版本（`ls "C:\Program Files\Blender Foundation"`）

### 问题：VRay 代理（VRayProxy）模型导入 Blender 后丢失（空物体）

**TL;DR**：`io_scene_max` 不支持 VRay 代理，代理物体（`.vrmesh`）导入后是空 EMPTY 占位，网格内容丢失。

- 问题：用户反馈 `.max` 导入 Blender 后"有模型掉了"；实测 `.blend` 里 561 个 EMPTY 中 63 个无任何子对象、另有 7 个 0 顶点 0 面网格，贴图路径全部完好（排除贴图问题）
- 根因：场景含 **12 个 VRay 代理文件（`.vrmesh`）**，代理网格数据存在代理文件里（渲染时加载），扩展不加载代理网格，只创建空占位
- 排查方法（可复现）：① `MAXFILES.TXT`（Max 资源清单）grep `vrmesh` 数代理 ② 打开 .blend 统计 `EMPTY` 与 `MESH_NO_GEO`（0 polygons）③ 核对贴图路径存在性排除贴图丢失
- 解决：在 3ds Max 里把 VRay 代理**转成真实网格**（右键 → 转换为可编辑多边形/网格，转换会读取 .vrmesh 加载网格）→ 另存/归档 → 重新导入 Blender
- 预防：导入前在 Max 检查场景是否含代理（`MAXFILES.TXT` 搜 vrmesh）；含代理先转换再走导入流程

### 方案决策:.max 进 Blender 用「FBX(勾选实例化)」最稳定

- 结论：Max 转材质 → 导出 FBX（勾选**实例化 Instancing + 嵌入媒体**）→ Blender 导入 FBX，**优于 io_scene_max 直读 .max**
- 原因：① 实例关系保留（Blender 导入后为共享网格数据，编辑一个全部同步）② Physical 材质映射 Principled BSDF 更完整 ③ 规避直读限制（VRay 代理丢失、压缩格式、V-Ray 材质部分转换）
- 适用：本机有 3ds Max 时首选；无 Max（只有别人给的 .max 文件）时走 io_scene_max 直读（指南路线 B）
- 注意：代理（.vrmesh）导出 FBX 前仍需在 Max 转真实网格；灯光仍不导出

## 待办 / 已知边界

- [ ] 每面纹理的位置偏移（origin）与旋转：SketchUp「纹理 → 位置」手动调整过的贴图目前无法对齐
- [ ] 贴图 Alpha 自动接透明材质（镂空栏杆等）：当前仅材质颜色带透明时才设 BLEND
- [ ] 后台 headless 导入大场景稳定性（内存 17GB 级场景耗时极长）
