# 达芬奇接入、检查与卸载

## Windows 安装向导

长期使用请下载 `Juzhou-v1.2.0-windows-x64-setup.exe`，双击中文向导安装。更新保留原配置、文稿和模型关联，卸载从 Windows「已安装的应用」进行。详见 [安装说明](INSTALL.zh-CN.md)。

以下 CMD 与手动配置步骤属于 **便携版的辅助方式**，不等同于 EXE 安装版。

## 便携包的启动入口

| 入口 | 作用 | 是否需要安装 |
| --- | --- | --- |
| `00-Start.cmd` | 从当前解压目录启动，工作数据放在旁边的 `data` | 不需要；建议先用这个 |
| `01-Install-Resolve.cmd` | 旧式 CMD 安装辅助，复制程序到当前用户并添加达芬奇菜单、桌面快捷方式 | 仅在需要固定安装和菜单入口时使用 |
| `02-Check-Setup.cmd` | 只读检查文件、安装目录和菜单配置的目标路径 | 不修改工程、设置或模型 |
| `句轴.exe` | 原始程序入口，使用安装版的默认／已配置数据位置 | 可直接运行，但不是 `00-Start.cmd` 的便携数据方式 |

便携版需解压到可写目录。基础程序不需要安装 Python、Node.js 或 npm；语音模型另按 [模型教程](MODEL_SETUP.zh-CN.md) 配置。首次使用需 WebView2 Runtime，通常随 Edge／Windows 已安装；缺失时从 [微软官网](https://developer.microsoft.com/microsoft-edge/webview2/) 安装。

## 核心流程：把 SRT 写到达芬奇时间线

1. 打开达芬奇项目及目标时间线；在句轴导入 SRT，进入字幕页校对文字和时间。
2. 点击「写入达芬奇」，核对目标时间线、起始时间码、新字幕轨道、条数，以及未复核／重叠提示。
3. 确认后检查达芬奇中的字幕文字、入出点和轨道。时间重叠可能需要多条字幕轨道。

已有时间码的 SRT 不需要重新转录或下载语音模型。由音频生成的字幕也走同一写入流程。写入失败时保留编辑结果，导出 SRT 后可从达芬奇媒体池导入，再拖到时间线字幕轨道并核对位置。

## 自动安装菜单入口

1. 保存工程并退出 Resolve，关闭句轴。完整解压 ZIP。
2. 双击 `01-Install-Resolve.cmd`，看清提示后按键继续；以软件弹出的安装结果为准。旧入口 `安装到本机.cmd` 调用同一个安装器，保留供已熟悉它的用户使用。
3. 程序复制到 `%LOCALAPPDATA%\Programs\Juzhou`，创建桌面与开始菜单的「句轴」快捷方式。工作流插件写入 ProgramData 的专属子目录；权限不足可能只完成脚本菜单和桌面入口，弹窗会说明。
4. 重新打开 Resolve，从 **工作区（Workspace）→ 工作流整合（Workflow Integrations）→ 句轴** 启动；也可尝试 **工作区 → 脚本（Scripts）→ Utility → Juzhou**。

Resolve 官方的工作流整合文档面向 **Studio**。菜单和外部脚本能力随版本、版本类型和系统权限不同，不能保证免费版或所有旧版均可用；独立的 SRT 编辑和导出不受菜单入口限制。

安装版默认数据在 `%LOCALAPPDATA%\Juzhou`，已有 `data-location.json` 指针会沿用。**便携 `data` 与安装版数据不是同一份工作稿，不会自动合并迁移。** 切换方式前导出工程／字幕或备份原数据，避免把打开空会话误认为丢稿。建议长期固定使用一种启动方式。

## 连接设置与验证

在提供该设置的 Resolve Studio 中，打开 **偏好设置（Preferences）→ 系统（System）→ 常规（General）→ 外部脚本使用（External scripting using）**，选择 **Local／本地** 并保存。只做本机连接不必选 Network。完整加载工程和时间线后再连接。

本版通过 Resolve 安装目录内的 `fuscript.exe` 调用脚本，按 `%PROGRAMFILES%\Blackmagic Design\DaVinci Resolve` 查找。若 Resolve 安装在其他位置，当前程序不会自动适配所有自定义路径；先通过 SRT 导出使用，反馈安装位置以便后续适配。

运行 `02-Check-Setup.cmd` 可以检查程序文件与菜单目标路径；它**不实际连接 Resolve、不证明写入成功，也不测试 CUDA 或模型识别**。真正连接检查：打开句轴的配音来源，读取达芬奇当前时间线，确认名称与轨道一致；写入操作先在测试工程中用 2～3 条字幕验收。

官方依据为 Resolve 安装目录 `Support/Developer/Scripting/README.md` 和 `Workflow Integrations/README.txt`；界面翻译随版本可能略有变化。建议保存工程后操作，不修改键盘预设。

## 自动安装失败时：手动配置

先确保固定程序目录内已有完整程序。可将解压包内的 `句轴.exe`、`_internal`、`integration`、第三方许可及说明复制到 `%LOCALAPPDATA%\Programs\Juzhou`；模型可单独关联，不复制其他人的 `data`、日志、会话或 `data-location.json`。

### 工作流整合入口

1. 打开 `%PROGRAMDATA%\Blackmagic Design\DaVinci Resolve\Support\Workflow Integration Plugins`。
2. 新建且只使用 `com.juzhou.subtitle` 子文件夹，把程序包 **integration 文件夹里的内容**复制进去，不能再多套一层 integration。
3. 在这个子文件夹中新建 `application.json`。用 PowerShell 按当前用户实际路径生成：

```powershell
$exe = Join-Path $env:LOCALAPPDATA 'Programs\Juzhou\句轴.exe'
$plugin = Join-Path $env:PROGRAMDATA 'Blackmagic Design\DaVinci Resolve\Support\Workflow Integration Plugins\com.juzhou.subtitle'
if (-not (Test-Path -LiteralPath $exe)) { throw '先复制完整程序，再配置菜单。' }
$json = @{executable=$exe} | ConvertTo-Json
[IO.File]::WriteAllText((Join-Path $plugin 'application.json'), $json, (New-Object Text.UTF8Encoding($false)))
```

写 ProgramData 被拒绝时，完成复制／配置这一步可能需要管理员权限；独立运行不用管理员。不要改其他插件目录。重启 Resolve 后检查菜单。

### 脚本菜单入口

在当前用户的 `%APPDATA%\Blackmagic Design\DaVinci Resolve\Support\Fusion\Scripts\Utility` 放置 `Juzhou.lua`。以下 PowerShell 会创建该专属脚本及启动文件。已有自行修改的同名文件时，先备份：

```powershell
$program = Join-Path $env:LOCALAPPDATA 'Programs\Juzhou'
$exe = Join-Path $program '句轴.exe'
if (-not (Test-Path -LiteralPath $exe)) { throw '先复制完整程序。' }
$vbs = Join-Path $program 'LaunchJuzhou.vbs'
$line = 'CreateObject("WScript.Shell").Run """' + $exe + '""", 1, False'
[IO.File]::WriteAllText($vbs, $line, [Text.Encoding]::Unicode)
$scripts = Join-Path $env:APPDATA 'Blackmagic Design\DaVinci Resolve\Support\Fusion\Scripts\Utility'
New-Item -ItemType Directory -Path $scripts -Force | Out-Null
$lua = 'local launcher = ' + (ConvertTo-Json $vbs -Compress) + "`n" + 'os.execute(''wscript.exe "''..launcher..''"'')'
[IO.File]::WriteAllText((Join-Path $scripts 'Juzhou.lua'), $lua, (New-Object Text.UTF8Encoding($false)))
```

以上仅添加句轴入口，不安装语音模型。菜单打开后是独立桌面窗口，非 Resolve 原生停靠面板。

## 移动目录、更新与卸载

- 便携版移动：关闭句轴后移动整个目录；模型关联和配音路径若已改变，重新选择。Conda 千问环境移动后应按新路径重建；工作稿里旧素材路径也需重新关联。
- 旧 CMD 安装副本：菜单指向 `%LOCALAPPDATA%\Programs\Juzhou` 的副本；移动下载目录不会改变菜单目标。重新运行安装器会更新固定副本。
- 手动入口：移动它指向的程序后，更新 `application.json` 和 VBS 中的路径，然后重启 Resolve。
- 更新前备份工作数据、模型关联及自定义规则。不要删除模型来修复启动问题。
- 卸载菜单：先关闭句轴和 Resolve，删除 **仅属于句轴的** `com.juzhou.subtitle` 文件夹、`Utility/Juzhou.lua`、桌面与开始菜单的句轴快捷方式，重启 Resolve。
- 移除程序：确认数据和模型已备份，再删除固定安装目录或便携目录。`%LOCALAPPDATA%\Juzhou`、便携 `data` 和独立存放的模型默认保留；需要清理时由你另行决定。
