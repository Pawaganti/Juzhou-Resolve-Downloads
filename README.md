<p align="center"><img src="resources/brand.svg" width="72" alt="句轴"></p>

# 句轴 Juzhou · DaVinci Resolve 字幕插件与独立版

**句轴是一款面向 DaVinci Resolve 的 AI 字幕助手，利用本地语音模型完成转录与打轴，将生成或导入的 SRT 字幕写入达芬奇时间线，并支持文稿校对、波形调整及 SRT 导出。** 可独立运行，也可从达芬奇菜单启动。

Windows x64 · 当前版本 **v1.2.0** · 构建 `1.2.0-stable.20261030`。

## 功能

- **字幕写入达芬奇**：导入 SRT → 编辑校对 → 写入原生字幕轨道；也可读取时间线配音生成字幕，保留写入回执供核验与撤销。
- **本地 AI 转录与打轴**：支持千问 Qwen3-ASR、Qwen3-ForcedAligner、Zipformer 和 SenseVoice，多模型切换；有文稿直接对齐，无文稿先转录。
- **文稿整理**：常见文档导入、格式清理、分句、标点规则，原稿保护与红删蓝增对照。
- **字幕校对**：双向编辑、多选、拆分合并、查找替换、撤销重做、波形试听与时间调整。
- **输出交付**：SRT 自定义命名、日期与版本编号、重叠分轨、导出前问题提示、繁体中文导出。

## 下载与开始

1. 在 [Releases](https://github.com/Pawaganti/Juzhou-Resolve-Downloads/releases/latest) 下载 `Juzhou-v1.2.0-win32-x64.zip`，**完整解压到可写目录**，双击 `00-Start.cmd` 独立启动，无需安装开发环境。
2. 按 [模型配置教程](docs/MODEL_SETUP.zh-CN.md) 下载或关联模型。**运行包不含模型权重；第一次转录前必须配置模型。**
3. 打开句轴，选择配音 → 转录或用已有文稿打轴 → 校对并导出 SRT／写入达芬奇。

已有 SRT 可直接导入、校对并写入达芬奇，无需语音模型。需要达芬奇菜单和桌面快捷方式时，再运行 `01-Install-Resolve.cmd`；安装与便携启动的区别见 [接入说明](docs/RESOLVE_SETUP.zh-CN.md)。不要只复制 EXE，旁边的 `_internal` 和 `integration` 也必须保留。

[模型下载与配置](docs/MODEL_SETUP.zh-CN.md) · [达芬奇安装与检查](docs/RESOLVE_SETUP.zh-CN.md) · [使用说明](docs/USAGE.zh-CN.md) · [兼容性说明](docs/COMPATIBILITY.md) · [更新记录](CHANGELOG.md)

## 界面

下图为当前构建的**合成演示字幕**，不含个人项目；显示繁体 SRT 导出选项。

![字幕编辑与繁体导出](docs/screenshots/traditional-export.png)

## 使用范围

这里提供 Windows x64 运行包、使用说明与模型配置教程，不包含模型权重、个人资料或完整开发源码。首次语音处理需配置模型；已有 SRT 可直接校对。应用为独立窗口，达芬奇菜单负责启动，非原生停靠面板。

API 当前仅保存配置；中文、英文与混排是主要使用场景，其他语言未逐一实测。详见 [兼容性说明](docs/COMPATIBILITY.md)。

同系列：[声匣 SoundBox](https://github.com/Pawaganti/SoundBox-Resolve-Downloads)。
