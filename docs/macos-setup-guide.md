# macOS 环境搭建与部署完整指南

本指南专为 macOS 用户（特别针对 **Apple Silicon M系列芯片 M1 / M2 / M3 / M4** 及 Intel Mac）编写，包含两大部分：
1. **HyperFrames 短视频自动化生成工作流（macOS 部署实操）** —— 本项目视频流水线在 Mac 上的完整复现指南。
2. **本地声音克隆在 macOS 上的搭建运行（GPT-SoVITS / MPS 加速）** —— 利用 Mac 统一内存与 Metal GPU 跑本地声音克隆。

---

# 第一部分：HyperFrames 短视频生成工作流（macOS 搭建）

本项目在 macOS (Apple Silicon M1 Pro) 上实测验证，硬件 GPU 渲染速度超过 **300+ fps**，60 秒 1080x1920 高清视频可在 50 秒内极速导出。

### 1. 基础环境依赖安装

推荐使用 macOS 包管理工具 **Homebrew** 进行一键安装：

#### 1.1 安装 Homebrew（若未安装）
打开终端（Terminal）执行：
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### 1.2 安装 Node.js 与 FFmpeg
```bash
brew install node ffmpeg
```
验证安装：
```bash
node -v      # 推荐 v20+ 或 v22+
npm -v
ffmpeg -version
ffprobe -version
```

#### 1.3 安装 Python 与 edge-tts 配音库
本项目默认采用微软开源的高保真音质配音库 `edge-tts`（无需申请 API Key，极度稳定）：
```bash
brew install python
pip3 install edge-tts
```
测试配音功能：
```bash
edge-tts --voice zh-CN-YunyangNeural --text "测试Mac环境配音" --write-media /tmp/test.mp3
```

#### 1.4 安装 Google Chrome
HyperFrames 渲染依赖官方 Chromium 内核：
- 请确保系统“访达 -> 应用程序”中已安装 Google Chrome。
- 默认路径位于：`/Applications/Google Chrome.app/Contents/MacOS/Google Chrome`。

---

### 2. 全局安装与配置 HyperFrames

在终端运行：
```bash
npm install -g hyperframes
```

验证安装：
```bash
hyperframes --version
```

---

### 3. 运行项目与极速渲染

#### 3.1 进入项目工程目录
```bash
cd videos/voice-clone-explainer
```

#### 3.2 关键配置：指定 macOS 系统 Chrome 路径
在 macOS 下，建议将渲染器指向系统安装的原生 Google Chrome，即可自动激活 Apple Silicon 的 Metal/WebGL 硬件 GPU 加速：

```bash
# 设置 Chrome 浏览器路径
export HYPERFRAMES_BROWSER_PATH="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
```

#### 3.3 检查、预览与渲染命令

```bash
# 1. 代码规范与字体安全检查
hyperframes lint

# 2. 动效、布局安全区与时间轴健康度检查（WCAG AA 对比度与溢出分析）
hyperframes check

# 3. 启动本地浏览器实时交互预览
hyperframes preview

# 4. 硬件加速渲染导出 1080x1920 MP4 成片
hyperframes render . --output renders/video.mp4
```

---
---

# 第二部分：本地声音克隆在 macOS 上的搭建运行（Apple Silicon MPS）

Mac 电脑（M1/M2/M3/M4）拥有独特的**统一内存（Unified Memory）架构**，8G / 16G / 24G / 36G 统一内存均可被 GPU 直接调用，非常适合本地离线运行语音模型。

### 1. GPT-SoVITS 在 macOS 上的部署流程

#### 1.1 安装依赖工具
```bash
brew install git cmake ffmpeg
```

#### 1.2 克隆代码与创建 Python 环境
建议使用 Python 3.10 环境（通过 conda 或 venv）：

```bash
git clone https://github.com/RVC-Boss/GPT-SoVITS.git
cd GPT-SoVITS

# 创建并激活 Python 3.10 虚拟环境
python3.10 -m venv venv
source venv/bin/activate
```

#### 1.3 安装 macOS 适配版 PyTorch（启用 MPS Metal 加速）
```bash
pip install --upgrade pip
pip install torch torchvision torchaudio
```
验证 Apple Silicon MPS 加速可用：
```bash
python -c "import torch; print('MPS available:', torch.backends.mps.is_available())"
# 正常应输出: MPS available: True
```

#### 1.4 安装依赖项
```bash
pip install -r requirements.txt
```

#### 1.5 下载预训练模型权重
从 HuggingFace 下载底模权重并放入 `GPT_SoVITS/pretrained_models` 目录：
- 仓库地址：[HuggingFace: lj1995/GPT-SoVITS](https://huggingface.co/lj1995/GPT-SoVITS)
- 所需核心权重：`s1bert25hz-275hr.ckpt`、`s2G488k.pth` 等。

#### 1.6 启动 WebUI
```bash
python webui.py
```
终端输出后，在 Safari 或 Chrome 浏览器打开 `http://127.0.0.1:9874` 即可使用。

---

### 2. macOS 运行声音克隆的技巧与排坑

1. **设备选择**：
   - 在 WebUI 设置中，设备选择 `mps`（Metal Performance Shaders）或 `cpu`。
   - M系列芯片使用 `mps` 推理速度可达近实时。
2. **统一内存使用**：
   - 8G 内存 Mac：建议进行 **5 秒零样本（Zero-Shot）极速克隆**，或将微调 Batch Size 设为 2~4；
   - 16G 及以上 Mac：可轻松进行完整微调训练与批量高质量推理。
3. **断网离线使用**：
   - 模型权重与底模一旦下载完成，运行 `python webui.py` 时全程断开 Wi-Fi 也可以稳定合成音频，100% 保护个人声音资产隐私。
