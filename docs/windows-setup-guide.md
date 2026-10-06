# Windows 环境搭建与部署完整指南

本指南包含两大部分：
1. **HyperFrames 短视频自动化生成工作流（Windows 部署指南）** —— 在 Windows 下复现本项目，一键生成 AI 知识短视频。
2. **GPT-SoVITS 本地声音克隆环境搭建（8G 显卡 Windows 实操）** —— 在 Windows 下使用本地 NVIDIA 显卡断网运行声音克隆。

---

# 第一部分：HyperFrames 短视频生成工作流搭建

### 1. 基础环境依赖安装

在 Windows 10/11 系统上，需要安装以下基础工具：

#### 1.1 安装 Node.js
- 推荐版本：**Node.js v20 LTS 或 v22**
- 官网下载：[https://nodejs.org/](https://nodejs.org/)（下载 Windows x64 `.msi` 安装包，默认一路 Next 即可）。
- 验证安装：
  ```powershell
  node -v
  npm -v
  ```

#### 1.2 安装 FFmpeg（必须加入系统环境变量 PATH）
HyperFrames 视频抽帧、音频切分与合成依赖 FFmpeg。
- **推荐便捷安装方式（通过 winget 或 scoop）**：
  在以管理员身份运行的 PowerShell 中执行：
  ```powershell
  winget install Gyan.FFmpeg
  ```
- **或手动安装**：
  1. 前往 [Gyan.dev FFmpeg Builds](https://www.gyan.dev/ffmpeg/builds/) 下载 `ffmpeg-release-essentials.zip`。
  2. 解压到 `C:\ffmpeg`。
  3. 将 `C:\ffmpeg\bin` 添加到系统的 **环境变量 PATH** 中。
  4. 验证安装：
     ```powershell
     ffmpeg -version
     ffprobe -version
     ```

#### 1.3 安装 Python 与 edge-tts 配音库
本项目默认采用微软开源的高保真音质配音库 `edge-tts`（无需 API Key，免费稳定）。
- 官网下载安装 **Python 3.10+**（安装时务必勾选 **"Add python.exe to PATH"**）。
- 打开终端安装 `edge-tts`：
  ```powershell
  pip install edge-tts
  ```
- 测试配音命令：
  ```powershell
  edge-tts --voice zh-CN-YunyangNeural --text "测试配音" --write-media test.mp3
  ```

#### 1.4 安装 Google Chrome
HyperFrames 渲染依赖 Chromium 内核渲染页面动画：
- 请确保电脑已安装官方 Google Chrome。
- 默认安装路径通常为：`C:\Program Files\Google\Chrome\Application\chrome.exe`。

---

### 2. 全局安装与配置 HyperFrames

打开 PowerShell 或 Windows Terminal：

```powershell
npm install -g hyperframes
```

- 验证安装：
  ```powershell
  hyperframes --version
  ```

> **Windows 权限提示**：  
> 若提示 `无法加载文件 ... 因为在此系统上禁止运行脚本`，以管理员身份打开 PowerShell 运行：
> ```powershell
> Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
> ```

---

### 3. 克隆/初始化本项目与生成视频

#### 3.1 切换到项目工程目录
```powershell
cd /path/to/autoChatCut/videos/voice-clone-explainer
```

#### 3.2 指定 Windows 上的 Chrome 路径并渲染视频
在 PowerShell 中运行：

```powershell
# 设置 Chrome 浏览器路径
$env:HYPERFRAMES_BROWSER_PATH = "C:\Program Files\Google\Chrome\Application\chrome.exe"

# 1. 代码格式与规范检查
hyperframes lint

# 2. 动效、布局溢出与时间轴健康度检查
hyperframes check

# 3. 实时本地浏览器预览（按需开启）
hyperframes preview

# 4. 渲染导出 60 秒完整成片 MP4
hyperframes render . --output renders/video.mp4
```

---
---

# 第二部分：GPT-SoVITS 本地声音克隆环境搭建（8G 显卡 Windows 实操）

视频案例中使用的声音克隆技术以 **GPT-SoVITS** 最为成熟，支持 5 秒极速克隆与本地断网运行。

### 1. 硬件配置要求
- **系统**：Windows 10 / 11 64位
- **显卡**：NVIDIA 显卡（显存 6GB 以上，推荐 **RTX 2060 / 2080 / 3060 / 4060 等 8GB 显存**）
- **内存**：16GB 及以上
- **硬盘**：建议至少预留 20GB 空闲 SSD 空间

---

### 2. 最简单方式：官方整合包（解压即用，推荐小白）

对于 Windows 用户，强烈建议使用官方打包的预编译整合包（内置 Python、PyTorch、CUDA 环境与基础模型权重，无需手动配置环境）：

1. **下载地址**：
   - 官方 GitHub Release：[GPT-SoVITS Releases](https://github.com/RVC-Boss/GPT-SoVITS/releases)
   - 或从官方国内网盘镜像下载 `GPT-SoVITS-beta-整合包.7z`。
2. **解压安装**：
   - 解压到非中文、无空格路径，例如 `D:\AI\GPT-SoVITS\`。
3. **一键启动**：
   - 双击根目录下的 `go-webui.bat`。
   - 程序将自动加载环境并启动浏览器前端界面（默认地址：`http://127.0.0.1:9874`）。

---

### 3. 开发者源码搭建方式（纯净环境）

如果你希望通过 Git 源码在自有 Python 环境下运行：

#### 3.1 克隆仓库
```powershell
git clone https://github.com/RVC-Boss/GPT-SoVITS.git
cd GPT-SoVITS
```

#### 3.2 创建虚拟环境并安装 PyTorch CUDA
```powershell
python -m venv venv
.\venv\Scripts\activate

# 安装对应 CUDA 11.8 或 12.1 的 PyTorch
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

#### 3.3 安装依赖
```powershell
pip install -r requirements.txt
```

#### 3.4 下载基础预训练模型
从 HuggingFace 下载底模权重文件并放入对应文件夹：
- [GPT-SoVITS Pretrained Models](https://huggingface.co/lj1995/GPT-SoVITS)
- 将 `s1bert25hz-275hr.ckpt`、`s2G488k.pth` 等底模放入 `GPT_SoVITS/pretrained_models` 目录。

#### 3.5 启动 WebUI
```powershell
python webui.py
```

---

### 4. 8G 显卡实操：12秒本地声音克隆 4 步走

1. **音频录制与切片**：
   - 准备一段 10~30 秒左右自己清晰、无底噪的录音文件（`.wav` 格式）。
   - 在 WebUI 的 **"0-语音切分"** 面板中，填入音频路径，点击“开启语音切分”，自动按静音分句。
2. **自动语音识别打标（ASR）**：
   - 在 **"1-ASR"** 面板中，选择 `Faster-Whisper`，点击“开启ASR”，几秒内自动完成文字打标。
3. **一键格式化与微调训练（可选）**：
   - **零样本推理（Zero-Shot）**：如果只需快速克隆，直接在“1B-微调推理”中上传 5 秒音频作为 Prompt 音频，无需训练直接输入文字即可生成声音！
   - **微调训练（更拟真）**：在 8G 显卡上，Batch Size 设为 4~8，训练 10~15 个 Epoch（约只需 5~10 分钟）。
4. **断网推理生成**：
   - 进入 **"1C-推理"** 界面，输入任何你想说的文本，点击合成，即可导出完全属于你自己的声音音频！全程无需联网，完全离线运行。
