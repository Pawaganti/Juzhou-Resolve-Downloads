# 模型下载与配置：从零跑通句轴

适用：句轴 **1.2.0 / 1.2.0-stable.20261030**，Windows x64。模型不随软件上传，使用前请遵守各模型原站的许可。

## 先选一条路线

| 你的需求 | 建议配置 | 需要另外安装什么 |
| --- | --- | --- |
| 先试普通话转录和打轴，配置尽量简单 | Zipformer 中文 INT8；生成方案选「快速模式」 | 模型文件即可；CPU 可用 |
| 中文、英文或中英混排，日常使用 | Qwen3-ASR-0.6B + Qwen3-ForcedAligner-0.6B；均衡方案 | 两个模型 + 本文的千问 Python 环境 |
| 复杂口音，希望与小模型对照 | Qwen3-ASR-1.7B + 同一个 ForcedAligner | 更大的转录模型；复用千问环境 |
| 轻量口语转录备选 | SenseVoice Small INT8 | 模型文件；打轴另选 Zipformer 或千问对齐 |
| 只编辑、导出已有 SRT | 不需要语音模型 | 无 |

**ASR 是语音识别：把声音变成文字。ForcedAligner 是对齐：把已有文字定位到声音。两者不是同一个模型。** 这里的 Qwen3-ASR 也不是普通千问聊天／翻译模型，GGUF、Ollama 和其他模型架构不能直接关联使用。

## 1. 启动软件，找到模型目录

1. 下载 Windows ZIP，右键「全部解压」到可写目录，不要在压缩包里运行。
2. 双击 `00-Start.cmd` 独立启动；需要固定安装及达芬奇菜单时，才运行 `01-Install-Resolve.cmd`。详见 [接入说明](RESOLVE_SETUP.zh-CN.md)。
3. 设置 → **模型** → **本地模型** → **打开模型目录**。便携版是当前 EXE 旁的 `models`；安装版是 `%LOCALAPPDATA%\Programs\Juzhou\models`。没有该目录时可以新建。

后面的千问命令使用你自行选定的 `JUZHOU_MODELS` 路径变量。请先确定使用便携版还是安装版，不要把权重下载到另一套程序目录。工作数据与模型分开保存，模型不要放进缓存。

已有模型不用再下载：设置 → 模型 →「关联模型文件夹」，选择实际包含权重、配置和词表的文件夹，再勾选启用、保存设置。无法自动识别时展开「手动关联模型位置」，先选正确型号再填写路径。

## 2. 最简单：配置 Zipformer

模型来自 sherpa-onnx 维护者的 [Zipformer 中文 INT8 文件页](https://huggingface.co/csukuangfj/sherpa-onnx-zipformer-ctc-zh-int8-2025-07-03/tree/main)。只下载下列三个文件：

| 文件 | 下载链接 | 用途 |
| --- | --- | --- |
| `model.int8.onnx` | [下载模型](https://huggingface.co/csukuangfj/sherpa-onnx-zipformer-ctc-zh-int8-2025-07-03/resolve/main/model.int8.onnx?download=true) | 约 367 MB 的模型权重 |
| `tokens.txt` | [下载词表](https://huggingface.co/csukuangfj/sherpa-onnx-zipformer-ctc-zh-int8-2025-07-03/resolve/main/tokens.txt?download=true) | 词表 |
| `bbpe.model` | [下载分词文件](https://huggingface.co/csukuangfj/sherpa-onnx-zipformer-ctc-zh-int8-2025-07-03/resolve/main/bbpe.model?download=true) | 分词配置 |

把三个文件放成这样，不要多套一层同名文件夹：

```text
Juzhou/
  句轴.exe
  models/
    zipformer-zh-int8/
      model.int8.onnx
      tokens.txt
      bbpe.model
```

然后：设置 → 模型 → **重新扫描** → 确认 Zipformer 已安装并启用 → 保存。回到「文稿／转录」，生成方案选 **快速模式**。选一段 10～30 秒普通话音频，点击底部转录按钮；拿到文稿后按当前按钮继续生成字幕，再试听核对。

这条路线不需要额外 Python、CUDA 或 API Key。含较多英文时请用后面的千问路线，不要把中文模型的输出当作英文识别结果。

## 3. 可选：SenseVoice 和人声检测

### SenseVoice Small INT8

打开 [SenseVoice ONNX 文件页](https://huggingface.co/csukuangfj/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-2024-07-17/tree/main)，下载 `model.int8.onnx`（约 239 MB）和 `tokens.txt`，放进 `models/sensevoice-int8/`。

选择 **INT8** 文件即可，不需要同时下载较大的 `model.onnx`。重新扫描、启用后，在「识别设置」中把转录模型改为 SenseVoice；它只负责转录，不替代打轴模型。

### Silero VAD（辅助入出点精修）

打开 [Silero 官方文件目录](https://github.com/snakers4/silero-vad/tree/master/src/silero_vad/data)，下载 `silero_vad.onnx`，放进 `models/silero-vad/`。可用 [原始文件下载](https://raw.githubusercontent.com/snakers4/silero-vad/master/src/silero_vad/data/silero_vad.onnx)。

浏览器显示文件页时选择下载原始文件，不能把网页另存为 ONNX。下载到的权重应是 MB 量级，而非几百字节的链接文本。Silero 用于人声活动检测，不能独立识别字幕，呼吸和轻声仍可能需要人工判断。

## 4. 千问：先准备运行环境

**本节只安装运行环境，不下载模型。** 本版本使用 Transformers 后端，不要求安装 vLLM 或 FlashAttention。Qwen 官方也建议使用独立的 Python 3.12 环境；参见 [官方安装说明](https://github.com/QwenLM/Qwen3-ASR#environment-setup)。

为避免影响电脑原有 Python，使用独立的 Miniforge 环境：

1. 从 [Miniforge 官方发布页](https://github.com/conda-forge/miniforge/releases/latest) 下载 `Miniforge3-Windows-x86_64.exe`，安装到当前用户。不要选择 ARM 版，也不必修改系统 PATH。
2. 开始菜单搜索并打开 **Miniforge Prompt**。下列命令在这个窗口中执行，**不是普通 PowerShell**；每次粘贴一行，等上一行成功再执行下一行。
3. 先设置模型目录，下面两个命令**只选一个**。便携路径是示例，替换成你解压目录里 `models` 的实际位置：

```bat
REM 便携版（把示例路径换成自己的目录）
set "JUZHOU_MODELS=D:\Apps\Juzhou\models"
REM 或：安装版
set "JUZHOU_MODELS=%LOCALAPPDATA%\Programs\Juzhou\models"
```

4. 确认下面的目标文件夹里没有其他现成 Python 环境。若已有可用千问环境，先直接用后面的检查命令验证，不要覆盖安装。

```bat
conda create --prefix "%JUZHOU_MODELS%\qwen-runtime\python" python=3.12 pip -y
conda activate "%JUZHOU_MODELS%\qwen-runtime\python"
python -c "import sys; print(sys.executable)"
```

最后应显示以所选 `models\qwen-runtime\python\python.exe` 结尾的路径。必须在这个环境中继续安装，不能装进其他聊天工具或系统 Python。文件位置是本版本的查找约定；只在设置中关联权重，不会自动补齐环境。

### 选择 CPU 或 NVIDIA GPU

**没有 NVIDIA 显卡，或先确认能运行：**

```bat
python -m pip install torch torchaudio --index-url https://download.pytorch.org/whl/cpu
```

**使用 NVIDIA 显卡加速：**

打开 [PyTorch 官方安装选择页](https://pytorch.org/get-started/locally/)，选择 Windows / Pip / Python，以及与你显卡驱动兼容的 CUDA 版本。把页面给出的 `pip3 install ...` 改为 `python -m pip install ...`，在同一个 Miniforge Prompt 运行。使用其官方 CUDA 下载源；不要先装 CPU 版再误以为选中 CUDA 就能加速。

CUDA 加速需要兼容的 NVIDIA 显卡、驱动与 PyTorch，是否可用按下面的检测确认。PyTorch、下载缓存和运行时也占磁盘，除权重外请另留数 GB，GPU 版通常更多；模型文件大小不等于显存需求。

### 安装语音依赖并检查

```bat
python -m pip install "qwen-asr==0.0.6" "transformers==4.57.6" "qwen-omni-utils==0.0.9" soundfile
python -m pip check
python -c "from qwen_asr import Qwen3ASRModel,Qwen3ForcedAligner; import torch,soundfile; print('Qwen import OK'); print('Torch:',torch.__version__); print('CUDA:',torch.cuda.is_available())"
```

期待看到 `Qwen import OK`；CPU 路线的 `CUDA: False` 正常。GPU 路线若显示 False，先检查是否安装 CPU wheel、驱动是否过旧；未解决前用软件「基础设置 → 本地模型加速 → CPU」或「自动」，不要强制 CUDA。红色安装错误／依赖冲突未解决时不要继续推理。

软件启动环境时不会打开 Miniforge Prompt。再新开一个普通 **PowerShell**，用实际完整路径检查一次（下方为便携示例，安装版应换成安装目录），确认脱离激活窗口也能导入：

```powershell
& 'D:\Apps\Juzhou\models\qwen-runtime\python\python.exe' -c "from qwen_asr import Qwen3ASRModel,Qwen3ForcedAligner; import torch,soundfile; print('Qwen import OK'); print('CUDA:',torch.cuda.is_available())"
```

这一步报 DLL 或导入错误时，环境仍未就绪；保留完整错误，先解决缺依赖，再进入模型加载。不能只凭“模型已扫描到”判断千问已经能用。

**验证边界：**以上固定语音包版本取自当前可运行环境；Miniforge 从零安装路线依据官方环境方式及本软件的 Python 查找路径编写，尚未在一台全新 Windows 电脑上完整安装验收。这里不会把“文档写好”称作“所有电脑即装即用”。

## 5. 千问：下载模型文件

| 模型原名 | 作用 | 权重下载量约值 | 文件页 |
| --- | --- | --- | --- |
| Qwen3-ASR-0.6B | 日常转录 | 1.88 GB | [Hugging Face](https://huggingface.co/Qwen/Qwen3-ASR-0.6B/tree/main) / [ModelScope](https://modelscope.cn/models/Qwen/Qwen3-ASR-0.6B) |
| Qwen3-ASR-1.7B | 较大转录备选 | 4.70 GB | [Hugging Face](https://huggingface.co/Qwen/Qwen3-ASR-1.7B/tree/main) / [ModelScope](https://modelscope.cn/models/Qwen/Qwen3-ASR-1.7B) |
| Qwen3-ForcedAligner-0.6B | 文稿与配音打轴 | 1.84 GB | [Hugging Face](https://huggingface.co/Qwen/Qwen3-ForcedAligner-0.6B/tree/main) / [ModelScope](https://modelscope.cn/models/Qwen/Qwen3-ForcedAligner-0.6B) |

只用均衡方案时，下载 **0.6B ASR + ForcedAligner** 即可，约 3.72 GB 权重；1.7B 可以后再加。表中是磁盘下载量，非显存承诺。

### 方式 A：命令下载整个模型（推荐，少漏文件）

继续在已激活的 Miniforge Prompt 中执行。国内网络可使用 Qwen 官方列出的 ModelScope 来源：

```bat
python -m pip install modelscope
modelscope download --model Qwen/Qwen3-ASR-0.6B --local_dir "%JUZHOU_MODELS%\qwen-asr-0.6b"
modelscope download --model Qwen/Qwen3-ForcedAligner-0.6B --local_dir "%JUZHOU_MODELS%\qwen-aligner-0.6b"
```

可选大模型，再单独执行：

```bat
modelscope download --model Qwen/Qwen3-ASR-1.7B --local_dir "%JUZHOU_MODELS%\qwen-asr-1.7b"
```

Hugging Face 也可使用其 Python 下载接口；选择一种来源即可，不必重复下载：

```bat
python -c "import os; from huggingface_hub import snapshot_download; snapshot_download('Qwen/Qwen3-ASR-0.6B',local_dir=os.path.join(os.environ['JUZHOU_MODELS'],'qwen-asr-0.6b'))"
python -c "import os; from huggingface_hub import snapshot_download; snapshot_download('Qwen/Qwen3-ForcedAligner-0.6B',local_dir=os.path.join(os.environ['JUZHOU_MODELS'],'qwen-aligner-0.6b'))"
```

网络中断后使用相同命令、相同目标目录重新下载，工具会检查已有文件。下载到一半不算模型可用。

### 方式 B：网页逐个下载

进入上表文件页，把模型根目录中的**所有权重分片、配置、分词器与词表**下载到对应文件夹。不能只拿 `model.safetensors`；`config.json`、`tokenizer_config.json`、`preprocessor_config.json`、`vocab.json`、`merges.txt` 等也要保留。若有 `model.safetensors.index.json`，必须同时下载它列出的全部分片。不下载示例音频也可以。

最终目录示意：

```text
models/
  qwen-runtime/
    python/
      python.exe
      Lib/...
      DLLs/...
      ...环境的其他文件（完整保留）
  qwen-asr-0.6b/
    config.json
    model.safetensors
    tokenizer_config.json
    ...其余分词器和配置
  qwen-aligner-0.6b/
    config.json
    model.safetensors
    ...其余分词器和配置
```

**不要移动已经创建的 Conda 环境目录**；环境包含安装前缀，移动后应在新位置重新建立环境。权重则可放到其他磁盘，再在软件中关联。

## 6. 在软件中启用和试用

1. 关闭再打开句轴，设置 → 模型 → 重新扫描，确认对应模型已启用；保存设置。
2. 在「文稿／转录」生成方案中选择均衡方案。在「识别设置」确认：**转录为千问 0.6B，打轴为千问对齐**。大模型方案只替换 ASR，不需要另找“1.7B 对齐”。手动改组合后显示自定义属于正常状态。
3. 语言选普通话、英语或自动识别；中英混排可以先自动识别。专有名词可提前填写提示，但不保证纠正所有同音字。
4. 先用 10～30 秒音频验证：无文稿先转录，有文稿直接校对打轴。生成后试听检查入出点，再导出。

软件内模型旁的下载入口是**打开模型网页**，不是自动下载并安装全部依赖。已在其他位置保存的模型直接关联即可；不必复制大权重。

Qwen 的模型、依赖准备完整后在本机推理；不需要 API Key。当前版本的 API 面板仅保存配置，不能用填 Key 的方式替代本地模型。

## 7. 常见问题

| 现象 | 检查与操作 |
| --- | --- |
| 尚未安装模型 | 先看是否多套一层文件夹、下载文件是否完整，再重新扫描或手动关联 |
| 已下载但型号不符 | 选准确的 ASR／ForcedAligner 型号；聊天模型、GGUF 和其他 ONNX 变体不兼容 |
| 缺少千问运行组件 | 检查 `models/qwen-runtime/python/python.exe`，只下载权重不够；按第 4 节准备环境 |
| 引擎启动失败 | 在已激活环境运行第 4 节导入检查；在用户数据目录 `logs/qwen-worker.log` 查看具体错误 |
| CUDA 不可用 | 检查 NVIDIA 驱动与 PyTorch CUDA wheel；先改 CPU 或自动，不靠手填设备强行启用 |
| 显存不足 | 关闭占用显存的软件；改 0.6B、CPU 或自动，减少同时进行的模型任务 |
| 首次加载较慢 | 模型首次读取和初始化需要时间；先用短音频确认运行，再处理长文件 |
| 下载的模型只有几 KB | 可能保存了网页或 Git LFS 指针；从文件页的 Download 或命令重新下载原始文件 |
| 断网无法下载 | 换用同模型的官方 ModelScope 来源；已完整下载的本地推理不依赖在线下载 |
| 识别有错、时间点不准 | 先核对语言、配音来源和模型组合，再试听修正；模型可用不等于准确率 100% |

便携入口的用户数据在解压目录 `data`；安装版默认在 `%LOCALAPPDATA%\Juzhou`，若曾自定义则以 `data-location.json` 记录为准。缓存清理不会代替模型管理。卸载或换电脑前备份工作数据；分享问题日志前检查是否含路径、文稿或其他私人信息。

参考来源：[Qwen3-ASR 官方说明](https://github.com/QwenLM/Qwen3-ASR)、[Miniforge](https://github.com/conda-forge/miniforge)、[PyTorch](https://pytorch.org/get-started/locally/)。下载页面与目录结构核对于本次归档；上游发布可能变化，优先保留已验证环境，不要随意更新整套依赖。
