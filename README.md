# README\.md

\# 🎬 短视频生成流水线

更新: 2026\-04\-19 Skills 系统已整合 \(skills\_loader\.py\)

本地 ComfyUI \+ Fish Speech 短视频合成 AI 短剧完整方案，开源免费、本地离线部署，可全自动完成**定妆照生成、分镜制作、视频生成、AI配音、音视频合成**，一站式产出完整AI短剧。

## 📁 完整项目目录结构

```tree
comfyui_workflows/
├── main_pipeline.py          # ⭐ 主流水线（交互式一键生成）
├── pipeline_generator.py     # 旧版流水线脚本（兼容保留）
├── prompt_generator.py       # 智能提示词自动生成工具
├── skills_loader.py          # Skills 技能系统加载器
├── fish_speech_tts.py        # Fish Speech 本地TTS配音脚本
├── merge_video_audio.sh      # FFmpeg 音视频自动合并脚本
├── monitor.py                # 项目运行监控脚本
├── monitor_videos.py         # 视频生成监控脚本
├── image_viewer.py           # 图片预览查看工具
├── view_gallery.py           # 作品画廊查看脚本
├── 文生图.json               # ComfyUI 文生图工作流 (z-image-turbo)
├── wan2.2图文生视频.json      # Wan2.2 通用图文转视频工作流
├── shot1_i2v.json            # 1号镜头图生视频工作流
├── shot1_i2v_reference.json  # 1号镜头参考工作流
├── shot1_i2v_reference_test.json # 1号镜头测试工作流
├── shot2_i2v.json            # 2号镜头图生视频工作流
├── shot3_i2v.json            # 3号镜头图生视频工作流
├── video_results.json        # 视频生成结果配置文件
├── 分镜规划指南.md           # 项目分镜制作官方指南
├── 分镜规划表.md             # 标准化分镜参数对照表
├── 视频一致性保障方案.md     # 角色/画风统一解决方案
├── 白狐与樵夫_分镜规划.md    # 案例短剧分镜脚本
├── PROGRESS.md               # 项目迭代更新日志
├── README.md                 # 项目说明文档
├── .gitignore                # Git 忽略配置
├── backup/                   # 项目备份目录
├── output/test/              # 测试输出目录
├── skills/                   # 技能系统配置目录
├── web_ui/                   # 简易网页前端目录
└── projects/                 # 正式项目成品输出目录
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

||
|---|

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

## 📝 零基础新手完整搭建 \&amp; 运行教程

本教程适配 **RTX2060 6G 显存** 及以上英伟达显卡，全程**开源免费、无付费、无水印、本地离线**，新手只需复制命令，无需代码基础。

### 一、前置硬件\&amp;软件要求

**硬件要求**

- 显卡：NVIDIA RTX2060 6G 及以上（AMD显卡无法使用）

- 内存：16G 及以上

- 系统：Linux / Windows10\+

**必须预装软件（全部免费）**

- Python 3\.10（最稳定版本，务必匹配）

- Git

- CUDA 11\.8

- FFmpeg（音视频合成必需）

### 二、完整环境搭建步骤

#### 1\. 项目部署

将本项目下载放置路径：`桌面/comfyui\_workflows`（路径**禁止中文、空格**）

同级目录必备两个主程序：

- ComfyUI 主程序文件夹

- fish\-speech 配音程序文件夹

#### 2\. 安装项目依赖

进入项目根目录，打开终端，执行安装命令

```bash
# 创建虚拟环境
python -m venv comfyui-env

# 激活虚拟环境（Linux）
source comfyui-env/bin/activate

# 激活虚拟环境（Windows）
# comfyui-env\Scripts\activate

# 安装全部依赖
pip install torch torchvision torchaudio requests pillow ffmpeg-python opencv-python

```

#### 3\. 模型文件放置（核心必需）

所有模型全部免费开源，放入对应目录，缺一不可：

- **文生图模型**：`z\_image\_turbo\_bf16\.safetensors` →`ComfyUI/models/checkpoints/`

- **CLIP模型**：`qwen\_3\_4b\.safetensors` → `ComfyUI/models/clip/`

- **视频模型**：`wan2\.2\_i2v 量化版` → `ComfyUI/models/checkpoints/`

- **视频LoRA**：`wan2\.2\_i2v\_lightx2v\_4steps\_lora\_v1` → `ComfyUI/models/loras/`

- **TTS配音模型**：放入 `fish\-speech/checkpoints/FishAudio/s2\-pro/`

### 三、项目标准启动流程（固定顺序）

**⚠️ 必须严格按顺序启动，否则连接报错**

#### 第一步：启动 ComfyUI 服务

```bash
cd /home/evynlau/work/ComfyUI
python main.py

```

启动成功标识：浏览器可正常访问 `http://127\.0\.0\.1:8188`，页面无报错，保持终端后台运行，**不要关闭**。

#### 第二步：启动 Fish Speech 配音服务（可选）

```bash
source /home/evynlau/work/comfyui-env/bin/activate
cd /home/evynlau/桌面/prompts_project/fish-speech
python3 tools/api_server.py \
  --llama-checkpoint-path /home/evynlau/桌面/prompts_project/fish-speech/checkpoints/FishAudio/s2-pro \
  --decoder-checkpoint-path /home/evynlau/桌面/prompts_project/fish-speech/checkpoints/FishAudio/s2-pro/codec.pth \
  --device cuda

```

新开终端验证服务：

```bash
curl http://localhost:8080/v1/health

```

返回 `\{\&\#34;status\&\#34;:\&\#34;ok\&\#34;\}` 即为启动成功。

#### 第三步：启动短剧生成流水线

新开独立终端，进入项目目录

```bash
cd /home/evynlau/桌面/comfyui_workflows
# 新手推荐交互模式（傻瓜式引导）
python main_pipeline.py --interactive

```

### 四、完整生成操作流程

1. 运行交互命令后，输入短剧场景描述

2. 系统自动生成3组角色定妆照，预览后选择满意版本

3. 系统锁定角色种子，保证全片人物、画风统一

4. 自动生成多镜头分镜图（远景/中景/近景）

5. 自选需要生成动态视频的分镜镜头

6. 输入短剧旁白/台词，自动生成AI配音

7. 系统自动完成视频拼接、音视频合并

8. 最终成品 `final\_short\.mp4` 输出至 projects 项目文件夹

### 五、RTX2060 6G 专属优化设置（必看）

2060显存有限，默认参数会爆显存、闪退，必须修改参数：

- 视频分辨率：修改为 **576×576**

- 视频帧数：下调至 **48帧**（单镜头3秒左右）

- 优先使用 **8bit量化模型**，降低显存占用

- 禁止多服务同时运行，跑完图再跑视频

### 六、常见新手报错\&amp;解决方案

- **ComfyUI连接失败**：服务未启动、端口8188未监听，重启ComfyUI，检查网址无误

- **Fish Speech跳过**：配音服务未启动，可手动启动或直接跳过配音

- **爆显存/OOM闪退**：使用量化模型、降低分辨率和帧数、关闭多余后台程序

- **角色画风错乱**：未锁定定妆照种子，务必交互式手动选定角色

- **无FFmpeg合并**：确认系统已安装FFmpeg并配置环境变量

### 七、项目优势说明

- ✅ 100% 开源免费，无会员、无次数限制、无水印

- ✅ 全部本地离线运行，无需联网、无需付费API

- ✅ 支持个人自用、商用变现，无版权分成

- ✅ 自动保障角色一致性，适合批量制作连载短剧

- ✅ 零基础可上手，全程交互式引导，无需剪辑基础

## 🛠️ 故障排查

- **ComfyUI 连接失败**：确保已启动并监听`http://127\.0\.0\.1:8188`

- **Fish Speech 未运行**：执行健康检测，未启动则重新开服务，也可直接跳过配音

- **定妆照不满意**：输入 `r` 重新生成，或用 `\-c 5` 增加变体数量

- **分镜不满意**：可单独选择部分镜头生成视频，跳过不满意镜头

## 💡 GitHub 仓库侧边项目树开启教程

很多新手打开 GitHub 项目，只有 README 页面，**没有左侧文件目录树**，无法浏览源码、查看工作流文件，属于GitHub设置问题，非项目报错，下面是一键开启方法。

### 开启方法（浏览器端，永久生效）

1. 打开你的 GitHub 项目仓库主页

2. 点击页面右上角 **Settings（设置）**

3. 下滑找到 **Features**功能板块

4. **开启：Repository navigation tree**（仓库导航树）

5. 刷新页面，左侧即可出现**完整项目文件树侧边栏**

### 功能作用

- 侧边栏展示项目**全部源码、配置文件、教程文档**

- 可直接点击预览、在线编辑、下载单个文件

- 快速切换工作流json、脚本、说明文档，不用返回首页

### 常见问题

- **找不到设置选项**：仅仓库所有者可开启，游客无法开启

- **开启后不显示**：刷新浏览器、清除缓存即可恢复
