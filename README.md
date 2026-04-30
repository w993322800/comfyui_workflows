 短视频生成流水线
更新: 2026-04-19 Skills 系统已整合 (skills_loader.py)

本地 ComfyUI + Fish Speech 短视频合成方案。

目录结构
comfyui_workflows/
├── main_pipeline.py          # ⭐ 主流水线（交互式一键生成）
├── pipeline_generator.py     # 旧版流水线脚本
├── prompt_generator.py       # 提示词生成器
├── 文生图.json               # ComfyUI 文生图工作流 (z-image-turbo)
├── shot1_i2v.json           # ComfyUI 图生视频工作流 (Wan2.2 I2V)
├── fish_speech_tts.py       # Fish Speech TTS 合成脚本
├── merge_video_audio.sh     # FFmpeg 音视频合并脚本
└── projects/                # 生成的项目输出目录
快速开始
方式一：交互模式（推荐）
cd /home/evynlau/桌面/comfyui_workflows
python main_pipeline.py --interactive
系统会引导你完成：

输入场景描述
自动生成多种子定妆照
交互式选择满意版本
自动生成分镜图
选择分镜制作视频
可选添加 TTS 配音
方式二：一键生成
python main_pipeline.py -d "一个小男孩在森林里遇到一只白狐" -t "从前有座山..."
流水线架构
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
主要功能
1. 自动提示词丰富
输入简单描述，系统自动补充：
角色锚定（外貌、服装、表情）
时代背景
场景环境
镜头运动
动作描述
2. 定妆照生成
支持多种子生成（默认3种）
可调整种子多次尝试
交互式选择满意版本
自动保存选中种子用于后续一致性
3. 分镜图生成
基于选定定妆照的种子保持角色一致性
自动生成分镜描述（远景/中景/近景）
序列种子递增确保风格统一
4. 分镜视频生成
从分镜图生成视频
自动生成动作提示词
镜头运动自动补充
支持种子控制便于复现
5. 短片合成
视频拼接
TTS 配音合成（需启动 Fish Speech）
音视频合并
使用示例
生成定妆照（仅）
python main_pipeline.py -d "一个小男孩" --costume-only
指定分镜数量
python main_pipeline.py -d "场景描述" -s 5
指定定妆照变体数
python main_pipeline.py -d "场景描述" -c 5
使用已有定妆照
python main_pipeline.py -d "场景描述" --use-costume projects/xxx/costume.png
启动服务
ComfyUI
cd /home/evynlau/work/ComfyUI
python main.py
# 访问 http://127.0.0.1:8188
Fish Speech API Server（可选，用于TTS）
source /home/evynlau/work/comfyui-env/bin/activate
cd /home/evynlau/桌面/prompts_project/fish-speech
python3 tools/api_server.py \
  --llama-checkpoint-path /home/evynlau/桌面/prompts_project/fish-speech/checkpoints/FishAudio/s2-pro \
  --decoder-checkpoint-path /home/evynlau/桌面/prompts_project/fish-speech/checkpoints/FishAudio/s2-pro/codec.pth \
  --device cuda

# 验证
curl http://localhost:8080/v1/health
# 返回 {"status":"ok"} 表示正常
输出目录
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
├── audio/                     # 音频
│   └── narration.wav
├── final_short.mp4           # 最终成品
└── report.json               # 生成报告
工作流参数
文生图 (z-image-turbo)
参数	值
模型	z_image_turbo_bf16.safetensors
CLIP	qwen_3_4b.safetensors
分辨率	1024 × 1024
步数	8
图生视频 (wan2.2 I2V)
参数	值
模型	wan2.2_i2v_low/high_noise_14B_fp8_scaled
分辨率	640 × 640
帧数	81 帧 @ 16fps ≈ 5 秒
LoRA	wan2.2_i2v_lightx2v_4steps_lora_v1
故障排查
ComfyUI 连接失败
❌ 无法连接到 ComfyUI
解决：确保 ComfyUI 已启动在 http://127.0.0.1:8188

Fish Speech 连接失败
⚠️ Fish Speech 未运行（TTS将跳过）
解决：

检查 API Server：curl http://localhost:8080/v1/health
如未运行，按上文步骤启动
或直接回车跳过 TTS
定妆照不满意
系统会生成多种子版本，如都不满意：

选择 [r] 重新生成更多变体
可用 -c 5 生成更多版本
分镜图不满意
生成后可以选择只制作部分镜头的视频，其他可以跳过。

项目结构
comfyui_workflows/
├── main_pipeline.py          # 主流水线脚本
├── pipeline_generator.py     # 旧版脚本（保留兼容）
├── prompt_generator.py       # 提示词工具
├── 文生图.json               # z-image-turbo 工作流
├── shot1_i2v.json           # Wan2.2 I2V 工作流
├── fish_speech_tts.py       # TTS 工具
└── projects/                # 项目输出
