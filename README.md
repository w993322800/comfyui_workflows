# README\.md

\# 🎬 短视频生成流水线

更新: 2026\-04\-19 Skills 系统已整合 \(skills\_loader\.py\)

本地 ComfyUI \+ Fish Speech 短视频合成 AI 短剧完整方案。

## 📁 目录结构

```tree
comfyui_workflows/
├── main_pipeline.py          # ⭐ 主流水线（交互式一键生成）
├── pipeline_generator.py     # 旧版流水线脚本
├── prompt_generator.py       # 提示词生成器
├── 文生图.json               # ComfyUI 文生图工作流 (z-image-turbo)
├── shot1_i2v.json           # ComfyUI 图生视频工作流 (Wan2.2 I2V)
├── fish_speech_tts.py       # Fish Speech TTS 合成脚本
├── merge_video_audio.sh     # FFmpeg 音视频合并脚本
└── projects/                # 生成的项目输出目录
```

## 🚀 快速开始

### 方式一：交互模式（推荐新手）

```bash
cd /home/evynlau/桌面/comfyui_workflows
python main_pipeline.py --interactive
```

交互引导流程：

- 输入场景描述

- 自动生成多种子定妆照

- 交互式选择满意版本

- 自动生成分镜图

- 选择分镜制作视频

- 可选添加 TTS 专业配音

### 方式二：命令行一键生成

```bash
python main_pipeline.py -d "一个小男孩在森林里遇到一只白狐" -t "从前有座山..."
```

## 🔧 流水线架构

```Plain Text
┌─────────────────────────────────────────────────────────────────┐
│                     短视频生成流水线                                │
│                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐        │
│  │   描述输入     │ →  │   定妆照生成   │ →  │   交互选择    │        │
│  │  (简单描述)    │    │ (多种子尝试)   │    │ (pick fav)   │        │
│  └──────────────┘    └──────────────┘    └──────┬───────┘        │
│                                                  ↓                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐        │
│  │   最终短片    │ ←  │   音视频合并   │ ←  │   视频拼接    │        │
│  └──────────────┘    └──────────────┘    └──────┬───────┘        │
│                                                  ↓                 │
│              ┌──────────────┐    ┌──────────────┐                  │
│              │   分镜视频    │ ←  │   分镜图生成   │                  │
│              │  (Wan2.2)    │    │ (seed一致性)  │                  │
│              └──────────────┘    └──────────────┘                  │
└─────────────────────────────────────────────────────────────────┘
```

## ✨ 主要功能

1\. **自动提示词丰富**
输入简单描述，自动补充：角色外貌/服装/表情、时代背景、场景环境、镜头运动、动作描述。

2\. **定妆照生成**
支持多种子生成、可调整种子重试、交互式选版、锁定种子保证全片角色一致。

3\. **分镜图生成**
基于选定定妆照种子保持人物不变；自动远景/中景/近景分镜；序列种子递增统一画风。

4\. **分镜视频生成**
分镜图一键转 Wan2\.2 动态视频；自动补动作与镜头运动；支持种子可复现。

5\. **短片自动合成**
多镜头视频自动拼接、Fish Speech 本地 TTS 配音、FFmpeg 音视频自动合并。

## 💻 使用示例

仅生成定妆照

```bash
python main_pipeline.py -d "一个小男孩" --costume-only
```

指定分镜镜头数量

```bash
python main_pipeline.py -d "场景描述" -s 5
```

指定定妆照生成变体数量

```bash
python main_pipeline.py -d "场景描述" -c 5
```

复用已有定妆照继续制作

```bash
python main_pipeline.py -d "场景描述" --use-costume projects/xxx/costume.png
```

## 🖥️ 本地服务启动

### 启动 ComfyUI

```bash
cd /home/evynlau/work/ComfyUI
python main.py
```

访问地址：http://127\.0\.0\.1:8188

### 启动 Fish Speech 配音服务

```bash
source /home/evynlau/work/comfyui-env/bin/activate
cd /home/evynlau/桌面/prompts_project/fish-speech
python3 tools/api_server.py \
  --llama-checkpoint-path /home/evynlau/桌面/prompts_project/fish-speech/checkpoints/FishAudio/s2-pro \
  --decoder-checkpoint-path /home/evynlau/桌面/prompts_project/fish-speech/checkpoints/FishAudio/s2-pro/codec.pth \
  --device cuda
```

服务健康检测

```bash
curl http://localhost:8080/v1/health
```

返回 `\{\&\#34;status\&\#34;:\&\#34;ok\&\#34;\}` 即正常可用。

## 📂 输出目录结构

```tree
projects/project_YYYYMMDD_HHMMSS/
├── costume_photos/           # 定妆照
│   └── costume_001.png
├── storyboards/              # 分镜图
│   ├── shot_01_*.png
│   ├── shot_02_*.png
│   └── shot_03_*.png
├── videos/                    # 分镜视频
│   ├── shot_01_*.mp4
│   ├── shot_02_*.mp4
│   └── shot_03_*.mp4
├── audio/                     # 配音音频
│   └── narration.wav
├── final_short.mp4           # 最终成片短剧
└── report.json               # 项目参数记录
```

## ⚙️ 工作流参数配置

### 文生图 \(z\-image\-turbo\)

|参数|配置值|
|---|---|
|基础模型|z\_image\_turbo\_bf16\.safetensors|
|CLIP 模型|qwen\_3\_4b\.safetensors|
|生成分辨率|1024 × 1024|
|采样步数|8|

### 图生视频 \(Wan2\.2 I2V\)

|参数|配置值|
|---|---|
|视频模型|wan2\.2\_i2v\_low/high\_noise\_14B\_fp8\_scaled|
|分辨率|640 × 640|
|帧率/时长|81帧 @ 16fps ≈ 5秒|
|辅助LoRA|wan2\.2\_i2v\_lightx2v\_4steps\_lora\_v1|

## 🛠️ 故障排查

- **ComfyUI 连接失败**：确保已启动并监听 `http://127\.0\.0\.1:8188`

- **Fish Speech 未运行**：执行健康检测，未启动则重新开服务，也可直接跳过配音

- **定妆照不满意**：输入 `r` 重新生成，或用 `\-c 5` 增加变体数量

- **分镜不满意**：可单独选择部分镜头生成视频，跳过不满意镜头
