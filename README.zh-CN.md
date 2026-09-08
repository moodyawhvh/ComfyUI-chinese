# ComfyUI 中文文档

<div align="center">

[![原项目](https://img.shields.io/badge/原项目-Comfy--Org--ComfyUI-blue?style=flat-square&logo=github)](https://github.com/Comfy-Org/ComfyUI)
[![微信联系](https://img.shields.io/badge/微信-uaycar-brightgreen?style=flat-square&logo=wechat)](#)

</div>

> 本文档是 [Comfy-Org/ComfyUI](https://github.com/Comfy-Org/ComfyUI) 官方 README 的中文翻译整理版，长清单有精简，最新信息以原项目为准。
>
> **代部署 / 定制服务 / 技术咨询 请添加微信：uaycar**

## 📖 项目简介

ComfyUI 是面向视觉创作者的 AI 创作引擎：模块化的节点图界面让你对每个模型、每个参数、每个输出都有完全的控制力，可以生成图像、视频、3D 模型、音频等内容。

- 原生支持最新开源 SOTA 模型；[合作伙伴节点](https://docs.comfy.org/tutorials/partner-nodes/overview#partner-nodes)可调用 Nano Banana、Seedance、Hunyuan3D 等闭源模型。
- 支持 Windows、Linux、macOS，可本地（[桌面版](https://www.comfy.org/download)/便携包）或[云端](https://www.comfy.org/cloud)运行。
- 复杂工作流可通过 App Mode 以简洁界面呈现；API 端点可无缝集成进生产流水线。

## ✨ 主要特性

- 可视化节点图，无需代码即可搭建、复用图像/视频/音频/3D/文本工作流。
- 可复用子图、工作流模板、App Mode 与本地 API。
- 高效本地执行：异步队列、局部重执行、智能 VRAM/RAM 管理、模型卸载、量化模型支持。
- 原生模型支持面极广（代表性条目，完整模板见[工作流库](https://comfy.org/workflows/)）：
  - [文生图](https://comfy.org/workflows/tag/text-to-image/)：Stable Diffusion 1.5、SDXL、SD3.5、Flux.1、Flux.2、Qwen Image、Z-Image、Hunyuan Image 2.1、HiDream 等。
  - [图像编辑](https://comfy.org/workflows/tag/image-edit/)：Flux Kontext、Flux.2 Klein、Qwen Image Edit、HiDream E1.1、OmniGen2 等。
  - [视频生成](https://comfy.org/workflows/tag/video-generation/)：Wan 2.1/2.2、LTX-Video、HunyuanVideo 1.5、CogVideoX、Cosmos Predict2、Mochi 等。
  - [音频生成](https://comfy.org/workflows/tag/text-to-audio/)：ACE-Step 1.5、Stable Audio 3、MiniMax Music 3。
  - [3D 与视觉](https://comfy.org/workflows/)：Hunyuan3D 2.1、TripoSplat、SUPIR、Depth Anything 3、SAM 3 等。
  - [文本生成](https://comfy.org/workflows/tag/text-generation/)：Gemma 3/4、Qwen3 系列，支持多模态输入。
- 可加载完整 checkpoint 或分离的扩散模型、VAE、文本编码器、LoRA、ControlNet、适配器与放大模型。
- 内置局部重绘、扩图、参考条件、蒙版合成、模型合并、放大、补帧、分割、深度估计等工具。
- 工作流可存为 JSON，也能从生成的媒体文件中还原完整工作流与种子。
- 完全离线可用：核心不经请求不下载任何内容，`--disable-api-nodes` 可禁用可选付费 API 节点。
- 支持自定义节点扩展；用 [`extra_model_paths.yaml`](extra_model_paths.yaml.example) 配置额外模型路径。
- 支持 16bit PNG、32bit EXR、10bit AVIF 等高色深图像与多种 HDR 视频格式的读写。

## 🚀 安装

### Windows / macOS（推荐）

新手强烈推荐[桌面应用](https://comfy.org/download)，下载即用，是最简单的方式。

### Windows 便携版（整合包）

官方提供免安装整合包，下载后用 [7-Zip](https://7-zip.org) 解压直接运行；自带 Python 3.13 与 PyTorch CUDA 13.0，启动失败先更新 NVIDIA 驱动。把 checkpoint（巨大的 ckpt/safetensors 文件）放入 `ComfyUI\models\checkpoints`，其他大模型按说明放入 `ComfyUI\models\` 对应子目录。想与其他 UI 共享模型，把 `extra_model_paths.yaml.example` 重命名为 `extra_model_paths.yaml` 后编辑搜索路径即可。

- [NVIDIA GPU（20 系及以上）](https://github.com/comfyanonymous/ComfyUI/releases/latest/download/ComfyUI_windows_portable_nvidia.7z)
- [AMD GPU](https://github.com/comfyanonymous/ComfyUI/releases/latest/download/ComfyUI_windows_portable_amd.7z) ｜ [Intel GPU](https://github.com/comfyanonymous/ComfyUI/releases/latest/download/ComfyUI_windows_portable_intel.7z)
- [NVIDIA 旧卡版（CUDA 12.6 / Python 3.12，10 系及更早，勿用于 20 系以上）](https://github.com/comfyanonymous/ComfyUI/releases/latest/download/ComfyUI_windows_portable_nvidia_cu126.7z)

### comfy-cli

```bash
pip install comfy-cli
comfy install
```

### 手动安装（Windows / Linux）

1. Python 3.13 支持最好（3.14 可用但部分自定义节点可能有兼容问题）；NVIDIA 20 系及以上必须使用 cu130 及以上 PyTorch，建议始终使用较新的稳定版 torch。
2. Git clone 原仓库（https://github.com/Comfy-Org/ComfyUI）。
3. checkpoint 放入 `models/checkpoints`，VAE 放入 `models/vae`。
4. 按显卡安装 PyTorch：

```bash
# NVIDIA
pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu130
# AMD GPU (Linux ROCm 稳定版)
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/rocm7.2
# Intel Arc
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/xpu
```

5. AMD RDNA3（RX 7000 系）Windows/Linux 实验性支持：`pip install --pre torch torchvision torchaudio --index-url https://rocm.nightlies.amd.com/v2/gfx110X-all/`；RDNA4（RX 9000 系）把地址中的 `gfx110X-all` 换成 `gfx120X-all`。
6. 在 ComfyUI 目录内安装依赖：

```bash
pip install -r requirements.txt
```

### 昇腾 NPU / 寒武纪 MLU / 天数智芯

安装对应 torch_npu / torch_mlu（含 CNToolkit）/ Iluvatar Corex Toolkit 工具链后，按 Linux 手动安装流程操作，`python main.py` 启动。

## ▶️ 运行

```
python main.py
```

- 非官方支持的 AMD 卡：6700/6600 等 RDNA2 尝试 `HSA_OVERRIDE_GFX_VERSION=10.3.0 python main.py`；7600 等 RDNA3 用 `11.0.0`。
- ROCm 可设 `PYTORCH_TUNABLEOP_ENABLED=1` 提速（首次运行很慢）。
- 预览：`--preview-method auto`；要高质量预览，下载 [TAESD](https://github.com/madebyollin/taesd) 的 `taesd_decoder.pth` 等解码器放入 `models/vae_approx`，再用 `--preview-method taesd`。

## 🧩 ComfyUI-Manager

用于安装、更新、管理自定义节点的官方扩展：

```bash
pip install -r manager_requirements.txt
python main.py --enable-manager
```

## ⌨️ 常用快捷键（节选）

| 快捷键 | 功能 |
|:-------|:-----|
| `Ctrl` + `Enter` | 当前图加入生成队列 |
| `Ctrl` + `Shift` + `Enter` | 插队到队列最前 |
| `Ctrl` + `Alt` + `Enter` | 取消当前生成 |
| `Ctrl` + `Z` / `Ctrl` + `Y` | 撤销 / 重做 |
| `Ctrl` + `S` / `Ctrl` + `O` | 保存 / 加载工作流 |
| `Ctrl` + `B` | 旁路选中节点（等效临时移除并直连） |
| 按住 `Space` 拖动 | 平移画布 |
| 双击左键 | 打开节点快速搜索 |
| `Alt` + `+` / `Alt` + `-` | 画布放大 / 缩小 |
| `Q` / `H` | 显示/隐藏队列 / 历史 |
| `.` | 视图适配选中节点（未选中则适配全图） |

macOS 上 `Ctrl` 可换为 `Cmd`，完整列表见原项目 README。

## 📝 使用要点

- 只有输入全部就绪的节点才会执行；重复提交相同的图只执行第一次，改动只会重跑受影响的部分。
- 把生成的 PNG 拖回页面即可还原完整工作流（含种子）。
- 提示词可用 `(good code:1.2)` 调整权重（默认 1.1），字面括号写作 `\(`。
- `{day|night}` 语法实现通配/动态提示词，每次排队随机替换，支持 `//` 与 `/* */` 注释。
- 文本反演 embeddings 放入 `models/embeddings`，在 CLIPTextEncode 节点用 `embedding:文件名` 引用（`.pt` 可省略）。

## 💬 支持与社区

- [Discord](https://comfy.org/discord)：#help、#feedback 频道。
- [Matrix](https://app.element.io/#/room/%23comfyui_space%3Amatrix.org)：`#comfyui_space:matrix.org`（开源版 Discord）。
- 官网：https://www.comfy.org/

---

> 本文档为 [Comfy-Org/ComfyUI](https://github.com/Comfy-Org/ComfyUI) 的中文翻译整理版，所有代码与原始文档版权归原项目作者所有，遵循其原始许可证（GPL-3.0）。
>
> **代部署 / 定制服务 / 技术咨询 请添加微信：uaycar**
>
> **如果觉得有用，请给原项目 [Comfy-Org/ComfyUI](https://github.com/Comfy-Org/ComfyUI) 点个 Star！** ⭐
