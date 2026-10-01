# Radius UI - Mission Control Fixes

RimWorld 1.6 的独立 Harmony 补丁 Mod，修复 Radius UI - Mission Control 的信件入口。不修改、替换或打包 RadiusUI 原始源码和程序集，不新增存档数据。

## 修复范围

- 自定义信件的文字标签和信封左键都调用原信件的虚方法 `Letter.OpenLetter()`，不再因有效定位目标而只跳镜头。保留原版 ChoiceLetter 选项和成长信件等专用窗口入口。
- 原 Mod 的右键可否移除判定保持不变。
- 面板关闭时，恢复原版信件按钮、悬浮说明和布局，不再被自定义信件开关拦截。
- 面板开启、自定义信件关闭时，调用原版 `LetterStack.LettersOnGUI()`，信件从底部主按钮栏上方向上排列。
- 两项都开启时保留 RadiusUI 的信件外观、折叠布局和堆叠逻辑。

| Bottom-right panel | Styled letters | 修复后路径 |
| --- | --- | --- |
| 开 | 开 | RadiusUI 自定义信件；点击打开原信件 |
| 开 | 关 | RadiusUI 面板 + 原版信件绘制 |
| 关 | 开 | 原版面板 + 原版信件绘制 |
| 关 | 关 | 原版面板 + 原版信件绘制 |

不处理信件业务选项本身，也不修改其他 HUD 模块。启用 classic Modern Notifications 导致 Mission Control 自行停用时，本补丁同样不安装。

## 安装与加载顺序

1. Harmony (`brrainz.harmony`)
2. Radius UI Framework (`astryl.RadiusUI.Framework`)
3. Radius UI - Mission Control (`astryl.RadiusUI.MissionControl`)
4. Radius UI - Mission Control Fixes (`nanaloveyuki.radiusui.missioncontrol.fixes`)

将补丁文件夹放到游戏 `Mods/RadiusUI-MissionControl-Fixes` 并在 Mod 列表启用。更新程序集后重启游戏。停用本补丁会恢复原 Mod 的行为，不会修改其设置。

验证对象：提供的 Mission Control 3.0.0 DLL，SHA-256：

```text
303c25e90ce708e79142181bff7ed9147981d2575071dd711c81a1ae3b9f5c7f
```

启动成功日志：`[Mission Control Fixes] Letter click and settings routing patches installed.`

补丁通过运行时解析 Mission Control 的方法与字段，构建不需要其 DLL。启动时检查目标 IL 结构；上游版本改变导致不匹配时会记录错误并撤销本补丁已安装的修改，不伪装成修复成功。

## 构建与部署

需要 .NET SDK、Python 3.9+ 和已安装的 RimWorld 1.6。WSL 部署还需要 `powershell.exe` 和 `wslpath`，用于检查 Windows 游戏进程和目录联接。编译引用游戏程序集；NuGet 的 Harmony 只供编译使用，不随 Mod 分发。

目录沿用参考项目：`Source/`、`1.6/Assemblies/`、`About/`、`LoadFolders.xml`、`scripts/`。

### WSL / Linux

```bash
./scripts/deploy.sh --build-only
./scripts/deploy.sh
# 自定义游戏位置
RIMWORLD_DIR=/path/to/RimWorld ./scripts/deploy.sh
```

默认路径：`/mnt/e/Apps/Steam/steamapps/common/RimWorld`。

### Windows PowerShell

```powershell
./scripts/build-and-deploy.ps1 -BuildOnly
./scripts/build-and-deploy.ps1
./scripts/build-and-deploy.ps1 -RimWorldDir 'E:\Apps\Steam\steamapps\common\RimWorld'
```

支持 `Debug` / `Release`，默认 `Release`。直接构建：

```bash
dotnet build Source/RadiusUI.MissionControl.Fixes.csproj -c Release -p:RimWorldDir=/path/to/RimWorld
```

部署只复制本补丁 DLL、About.xml、LoadFolders.xml、README 和 LICENSE。不会复制 Harmony、RadiusUI 或游戏 DLL，也不复制源码。游戏运行、目标路径是符号链接/联接、已有目标目录缺少或不匹配本补丁 packageId 时拒绝部署。逐文件验证 SHA-256，不清理目标里的其他文件。

## 验证与限制

- Release 构建通过。
- 使用真实 Mission Control DLL 安装 Harmony 补丁，执行实际点击处理 IL：修复前展开状态的有效目标信封 `opens=0, jumps=1`；修复后文字/信封、折叠/展开、有效/无效目标以及 ChoiceLetter/其他 Letter 共 16 个分支均 `opens=1, jumps=0`。
- 实际开关拦截方法、补丁绘制入口及布局后处理的四种设置组合验证通过；验证时先安装上游 Harmony 调用链，再安装本补丁。
- 上述无界面验证隔离了 Unity 原生 GUI、镜头和绘制边界，不等于游戏内完整鼠标操作验证。
- WSL build-only 脚本执行通过；Windows PowerShell 脚本语法解析通过，未完整运行其构建部署路径。
- 隔离目录部署恰好复制五个允许文件并逐项校验 SHA-256；缺失身份、其他 Mod 身份、符号链接目标均拒绝。
- 实际游戏目录部署预检发现 RimWorld 正在运行，已拒绝部署；未绕过检查、未关闭游戏。

游戏内验收：打开有定位目标的接受/拒绝信件和成长信件，分别点击文字和信封；切换四种设置组合并在地图/世界界面打开信件；检查不可右键移除的信件仍保留原限制。

## 更新记录

### 1.0.0

- 恢复自定义信封的原信件打开行为。
- 修复面板与自定义信件开关组合造成的原版入口丢失。
- 添加独立构建和安全部署脚本。
