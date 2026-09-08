<div align="center">

# ComfyUI 中文翻译版

**[中文版] ComfyUI — 最强大且模块化的 AI 创作引擎，用图形节点界面生成图像/视频/音频/3D**

[![原项目](https://img.shields.io/badge/原项目-Comfy-Org--ComfyUI-blue?style=flat-square&logo=github)](https://github.com/Comfy-Org/ComfyUI)
[![中文文档](https://img.shields.io/badge/中文文档-README.zh--CN.md-orange?style=flat-square)](README.zh-CN.md)
[![GitHub Stars](https://img.shields.io/github/stars/Comfy-Org/ComfyUI?style=flat-square&label=原项目Stars)](https://github.com/Comfy-Org/ComfyUI/stargazers)
[![微信联系](https://img.shields.io/badge/微信-uaycar-brightgreen?style=flat-square&logo=wechat)](#)

</div>

---

> 这是 [Comfy-Org/ComfyUI](https://github.com/Comfy-Org/ComfyUI) 的中文翻译版本。
> 完整源代码请访问原项目：https://github.com/Comfy-Org/ComfyUI

**代部署 / 定制服务 / 技术咨询 请添加微信：uaycar**

---

## 📖 项目简介

ComfyUI 是目前最强大、模块化程度最高的扩散模型 / AI 创作引擎，采用图形化节点（graph/nodes）界面。创作者可以通过拖拽节点搭建工作流，生成图像、视频、3D 模型、音频等内容，并对每一个模型、每一个参数、每一个输出拥有完全的控制力。它原生支持最新的开源 SOTA 模型，也能通过合作伙伴节点调用 Nano Banana、Seedance、Hunyuan3D 等闭源模型，可运行于 Windows、Linux、macOS 本地环境或官方云端。

## ✨ 主要特性

- 可视化节点图：无需写代码即可搭建并复用图像、视频、音频、3D、文本工作流。
- 可复用子图、工作流模板、App Mode 与本地 API，方便集成进生产流水线。
- 高效本地执行：异步队列、局部重执行、智能显存/内存管理、模型卸载、量化模型支持。
- 原生支持海量模型：Stable Diffusion 1.5/SDXL/SD3.5、Flux、Qwen Image、Wan 2.x、HunyuanVideo、CogVideoX、ACE-Step、Hunyuan3D、SAM 3、Qwen3 等。
- 支持加载完整 checkpoint 或分离的扩散模型、VAE、文本编码器、LoRA、ControlNet、放大模型等。
- 内置局部重绘、扩图、蒙版合成、模型合并、放大、补帧、分割、深度估计等工具。
- 工作流可保存为 JSON，也能从生成的图片/视频中还原完整工作流与种子。
- 核心完全离线可用，不请求就不下载任何内容；可用 `--disable-api-nodes` 强制离线。
- 丰富的自定义节点生态，可通过 `extra_model_paths.yaml` 灵活配置模型目录。
- 支持 16bit PNG、32bit EXR、10bit AVIF 等高色深图像与多种 HDR 视频格式的读写。

## 📁 文件说明

| 文件 | 说明 |
|:-----|:-----|
| README.md | 本文件（中文简介） |
| README.zh-CN.md | 详细中文文档（完整汉化） |

## 🚀 快速开始

1. 新手最简单的方式：下载官方桌面版（Windows / macOS）——https://www.comfy.org/download
2. Windows 免安装：下载便携整合包，用 7-Zip 解压后直接运行；checkpoint 大模型放入 `ComfyUI\models\checkpoints`。
3. 命令行安装（comfy-cli）：

```bash
pip install comfy-cli
comfy install
```

4. 手动安装（Windows / Linux）：Git clone 仓库 → 按显卡安装 PyTorch → 安装依赖：

```bash
pip install -r requirements.txt
```

5. 启动：

```
python main.py
```

6. 浏览器打开 `http://127.0.0.1:8188`，拖入或新建节点图，把模型文件放进 `models/` 对应子目录即可出图。

完整源代码与最新版本请访问原项目：https://github.com/Comfy-Org/ComfyUI

## 📞 联系方式

**代部署 / 定制服务 / 技术咨询 请添加微信：uaycar**

---

本项目为 [Comfy-Org/ComfyUI](https://github.com/Comfy-Org/ComfyUI) 的中文翻译版本，所有代码版权归原项目作者所有，遵循其原始许可证。

**如果觉得有用，请给原项目点个 Star！** ⭐
