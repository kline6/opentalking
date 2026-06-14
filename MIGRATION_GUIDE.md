# OpenTalking 换机迁移使用指南

> 本文档帮助你将 OpenTalking 项目从一台电脑迁移到另一台电脑并正常运行。

---

## 一、GitHub 仓库信息

- **仓库地址**：https://github.com/kline6/opentalking2（私有仓库）
- **克隆命令**：
  `ash
  git clone https://github.com/kline6/opentalking2.git
  cd opentalking2
  `

---

## 二、新电脑环境要求

| 依赖 | 版本要求 | 说明 |
|------|---------|------|
| Python | 3.10+（推荐 3.11） | 核心运行时 |
| Node.js | 18+ | 前端开发服务器 |
| Git | 任意 | 版本管理 |
| ffmpeg | 最新版 | 视频处理 |
| NVIDIA 驱动 + CUDA | 仅 GPU 推理需要 | 本地模型推理 |

---

## 三、安装步骤

### 1. 克隆仓库
`ash
git clone https://github.com/kline6/opentalking2.git
cd opentalking2
`

### 2. 安装 Python 依赖

方式 A（推荐，使用 uv）：
`ash
uv sync --extra dev --python 3.11
`

方式 B（使用 pip）：
`ash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install --index-url https://pypi.tuna.tsinghua.edu.cn/simple -e ".[dev]"
`

### 3. 安装前端依赖
`ash
cd apps/web
npm ci
cd ../..
`

### 4. 配置环境变量（最关键）
`ash
cp .env.example .env
`

编辑 .env 文件，填入你的 API Key。最小必填配置：

`nv
# LLM 对话（必须）
OPENTALKING_LLM_API_KEY=你的LLM_API_Key

# 语音识别 STT（必须）
OPENTALKING_STT_DEFAULT_PROVIDER=dashscope
OPENTALKING_STT_DASHSCOPE_API_KEY=你的STT_API_Key

# 语音合成 TTS（默认使用免费的 Edge TTS，无需 key）
OPENTALKING_TTS_DEFAULT_PROVIDER=edge
`

### 5. 验证安装
`ash
opentalking-unified --help
`

### 6. 启动服务
`ash
# Mock 模式（无需 GPU，快速验证全链路）
bash scripts/start_unified.sh --mock

# 或本地 GPU 模式
bash scripts/start_unified.sh --backend local --model quicktalk
`

启动后访问：
- 前端 WebUI：http://localhost:5173
- 后端 API：http://localhost:8000

---

## 四、需要从旧电脑额外迁移的文件

以下文件 **不在** Git 仓库中，需要手动从旧电脑传输：

| 文件/目录 | 说明 | 是否必须 |
|-----------|------|---------|
| .env | 你的 API Key 和个性化配置 | 必须（或在新电脑重新配置） |
| models/ | 本地模型权重 | 仅本地推理需要 |
| models/quicktalk/ | QuickTalk 模型 | 用 quicktalk 时需要 |
| models/wav2lip/ | Wav2Lip 模型 | 用 wav2lip 时需要 |
| models/musetalk/ | MuseTalk 模型 | 用 musetalk 时需要 |
| models/local-audio/ | 本地 STT/TTS 模型 | 用本地语音模型时需要 |

### 迁移方式

1. **配置文件**：直接复制旧电脑的 .env 文件到新电脑项目根目录
2. **模型权重**（如需要）：
   - 推荐使用移动硬盘、网盘或局域网传输
   - 或者在新电脑上重新下载：
     `ash
     opentalking-download-models quicktalk
     `

---

## 五、快速开始命令速查

`ash
# 首次使用 - Mock 模式验证
bash scripts/start_unified.sh --mock

# 消费级 GPU + QuickTalk
bash scripts/start_unified.sh --backend local --model quicktalk

# 消费级 GPU + Wav2Lip
bash scripts/start_unified.sh --backend local --model wav2lip

# 远端 OmniRT 服务
bash scripts/start_unified.sh --backend omnirt --model flashtalk --omnirt http://<gpu-server>:9000

# 停止所有服务
bash scripts/quickstart/stop_all.sh
`

---

## 六、常见问题

### Q: 没有 GPU 可以用吗？
可以。使用 --mock 模式可以跑通 API / WebUI / LLM / TTS / WebRTC 全链路，只是没有视频推理渲染。

### Q: Edge TTS 需要 API Key 吗？
不需要。Edge TTS 是免费的微软语音合成服务，开箱即用。

### Q: 需要 Redis 吗？
本地开发默认使用内存模拟，不需要安装 Redis。生产部署时才需要。

### Q: Windows 用户怎么部署？
推荐使用 WSL2 Ubuntu 部署，详细步骤见 docs/zh/quick-start/windows-deployment.md。

### Q: 大文件（模型权重/视频）怎么传？
仓库中的 examples 大文件（约225MB）因网络原因未能推送。可通过以下方式获取：
- 旧电脑直接拷贝
- 或参考 README 从源下载示例素材

---

## 七、有用的文档

- README.md - 项目总览和快速开始
- docs/zh/tutorials/install-from-source.md - 源码安装详细教程
- docs/zh/quick-start/windows-deployment.md - Windows 部署指南
- docs/zh/tutorials/configuration.md - 配置详解
- AGENT.md - 项目架构和协作规范

---

*文档生成时间：2026-06-14*
