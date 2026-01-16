# Episode 20: Use Wan 2.2, ComfyUI and n8n to Generate Videos for Free

> AI Agents A-Z - 用 n8n 构建实用的 AI Agent
> 难度: ⭐⭐⭐ | 预估时间: 45 分钟

---

## [Level 1] 这一集在做什么？

本集教你用 Modal.com 的免费 GPU 运行 Wan 2.2 视频生成模型，通过 ComfyUI 和 n8n 实现零成本视频生成。

**一句话**：像"视频工厂"一样，用文字生成视频，或用图像生成视频，完全免费。

**适用场景**：
- 文字生成视频 (Text to Video)
- 图像生成视频 (Image to Video)
- 无需本地 GPU 的 AI 视频创作

> **快速判断**：如果你需要**零成本生成高质量 AI 视频**，这一集适合你。
> 想了解更多？继续阅读 [Level 2]。

---

## [Level 2] 核心概念

### 你会学到什么

| 序号 | 概念 | 说明 |
|------|------|------|
| 1 | **Wan 2.2** | 阿里巴巴开源的强大视频生成模型 |
| 2 | **ComfyUI** | 节点式 UI，用于构建视频生成工作流 |
| 3 | **Modal.com** | 无服务器 GPU 平台，有 $30/月免费额度 |
| 4 | **ComfyUI API** | 将 ComfyUI 工作流转为 API 调用 |

### 涉及的 n8n 节点

| 节点类型 | 用途 | 新手友好度 |
|----------|------|------------|
| Form Trigger | 收集提示词/图像 | ⭐ 简单 |
| Set | 配置 ComfyUI URL | ⭐ 简单 |
| HTTP Request | 调用 ComfyUI API | ⭐⭐ 中等 |
| If | 状态判断 | ⭐ 简单 |
| Wait | 等待生成完成 | ⭐ 简单 |
| Code | 转换 base64 | ⭐⭐ 中等 |

### 涉及的外部服务

| 服务 | 免费额度 | 难度 | 官网 |
|------|----------|------|------|
| **Modal** | $30/月 | ⭐⭐ | [modal.com](https://modal.com/) |
| **Wan 2.2** | 开源 | ⭐⭐ | [huggingface.co](https://huggingface.co/Wan-Video/Wan2.2) |
| **ComfyUI** | 开源 | ⭐⭐⭐ | [comfy.org](https://docs.comfy.org/) |

> **了解够了？** 知道学什么就可以开始。继续阅读 [Level 3] 了解工作流结构。

---

## [Level 3] 工作流结构

### 工作流概览图 (Text to Video)

```
┌─────────────────────────────────────────────────────────────┐
│              "Wan 2.2 Text to Video" 工作流                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [Form Trigger] ──► [Setup defaults]                        │
│  用户输入表单      配置 ComfyUI URL                          │
│       │                  │                                   │
│       │                  └───► [Start the workflow]           │
│       │                        POST /prompt                   │
│       │                             │                        │
│       │                             ▼                        │
│       │                        [Wait until generated]         │
│       │                        等待视频生成                    │
│       │                             │                        │
│       │                             ▼                        │
│       │                        [Get video status]            │
│       │                        GET /history/{prompt_id}      │
│       │                             │                        │
│       │                             ▼                        │
│       │                        [Completed?]                  │
│       │                        检查是否完成                    │
│       │                             │                        │
│       │                    ┌──────┴──────┐                   │
│       │                    │             │                   │
│       │                    ▼             ▼                   │
│       │               [Success?]    [Loop back]             │
│       │               检查成功      继续等待                   │
│       │                    │             │                   │
│       │                    ▼             │                   │
│       │               [Download]       │                   │
│       │               下载视频         │                   │
│       │                    │             │                   │
│       └────────────────────┴─────────────┘                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 工作流概览图 (Image to Video)

```
┌─────────────────────────────────────────────────────────────┐
│             "Wan 2.2 Image to Video" 工作流                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [Form Trigger] ──► [Convert image to base64]              │
│  用户输入+图像      转换图像格式                             │
│       │                  │                                   │
│       ▼                  ▼                                   │
│  [Setup defaults] ──► [Start the workflow]                 │
│  配置参数           POST /prompt (含图像)                     │
│       │                  │                                   │
│       ▼                  ▼                                   │
│  [Wait loop] ──────────► [Get video status]                │
│  等待轮询            检查生成状态                             │
│       │                  │                                   │
│       └──────┬───────────┘                                   │
│              │                                               │
│              ▼                                               │
│         [Download]                                          │
│         下载视频                                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Modal + ComfyUI 架构

```
┌─────────────────────────────────────────────────────────────┐
│                 Modal 服务器架构                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  n8n ──► ComfyUI Server ──► Wan 2.2 Model                  │
│                              │                              │
│                              └──► 视频生成                   │
│                                                             │
│  ComfyUI 工作流 JSON 定义处理流程                            │
│  Modal 提供无服务器 GPU 执行环境                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 数据流

```
Text to Video:
  提示词 ──► ComfyUI ──► Wan 2.2 ──► 视频

Image to Video:
  图像 + 提示词 ──► ComfyUI ──► Wan 2.2 ──► 视频
```

### 可用模型

| 模型 | 大小 | 类型 | 特点 |
|------|------|------|------|
| **Wan 2.2 5B** | 5B 参数 | T2V/I2V | 更快，适合测试 |
| **Wan 2.2 14B** | 14B 参数 | T2V/I2V | 更高质量，需要更多资源 |

### 节点说明

| 节点 | 类型 | 配置要点 | 数据输出 |
|------|------|----------|----------|
| **On form submission** | Form Trigger | 收集 prompt/图像 | 表单数据 |
| **Setup defaults** | Set | 配置 ComfyUI URL | {COMFY_URL, PROMPT} |
| **Start the workflow with ComfyUI** | HTTP Request | POST /prompt | {prompt_id} |
| **Wait until the video gets generated** | Wait | 等待 10 秒 | 延迟 |
| **Get video generation status** | HTTP Request | GET /history/{id} | 状态数据 |
| **Completed?** | If | 检查完成状态 | 路由 |
| **Success?** | If | 检查成功状态 | 路由 |
| **Download the video** | HTTP Request | GET /view | 视频文件 |

> **准备就绪？** 理解工作流结构后，继续阅读 [Level 4] 开始构建。

---

## [Level 4] 构建步骤

### 前置准备

在开始之前，请确保：

- [ ] n8n 已安装并运行（访问 http://localhost:5678）
- [ ] Modal 账号已创建 ([modal.com](https://modal.com/)）
- [ ] Modal CLI 已安装 (`pip install modal`)
- [ ] Python 3.10+ 已安装

### 步骤 1: 创建虚拟环境并安装 Modal

**目标**: 设置 Python 环境

**操作**:

1. 创建虚拟环境:
```bash
python3 -m venv .venv
source .venv/bin/activate  # Linux/Mac
# 或
.venv\Scripts\activate  # Windows
```

2. 安装 Modal CLI:
```bash
pip install modal
```

3. 登录 Modal:
```bash
modal token new
```

**验证**: 运行 `modal --version` 显示版本号

---

### 步骤 2: 部署 Modal ComfyUI 服务器

**目标**: 在 Modal 上部署 Wan 2.2 + ComfyUI 服务

**操作**:

1. 选择你要部署的模型版本：
   - `modal_wan_comfyui_5b.py` - Wan 2.2 5B (Text/Image to Video)
   - `modal_wan_comfyui_14b_t2v.py` - Wan 2.2 14B (Text to Video)
   - `modal_wan_comfyui_14b_i2v.py` - Wan 2.2 14B (Image to Video)

2. 部署服务器:
```bash
modal serve modal_wan_comfyui_5b.py  # 测试运行
# 或
modal deploy modal_wan_comfyui_5b.py  # 正式部署
```

3. 复制输出的服务器 URL（格式：`https://xxx.modal.run`）

**验证**: 访问服务器 URL + `/system_stats`，应看到服务器状态

---

### 步骤 3: 导入 n8n 工作流

**目标**: 导入视频生成工作流

**操作**:

1. 在 n8n 中点击右上角 **"..."** 菜单
2. 选择 **"Import from File"**
3. 选择对应的工作流文件：
   - `n8n_wan_2.2_5b_t2v.json` - Text to Video
   - `n8n_wan_2.2_5b_i2v.json` - Image to Video
   - `n8n_wan_2.2_14b_t2v.json` - 14B Text to Video
   - `n8n_wan_2.2_14b_i2v.json` - 14B Image to Video

**验证**: 工作流应该显示在画布上

---

### 步骤 4: 配置 ComfyUI URL

**目标**: 连接到 Modal ComfyUI 服务器

**操作**:

1. 点击 **"Configure me"** 节点
2. 将 `comfyui_url` 替换为你的 Modal 服务器 URL

**示例**: `https://your-server.modal.run`

**验证**: URL 格式正确

---

### 步骤 5: 测试 Text to Video

**目标**: 生成第一个视频

**操作**:

1. 激活工作流
2. 打开表单 URL
3. 输入提示词：
   - Prompt: "A cat playing with a ball, cinematic lighting"
   - Negative prompt (可选): "low quality, blurry"
4. 提交表单
5. 等待生成（约 1-3 分钟）

**预期结果**: 获得一个 MP4 视频文件

---

### 步骤 6: 测试 Image to Video

**目标**: 从图像生成视频

**操作**:

1. 切换到 Image to Video 工作流
2. 打开表单 URL
3. 上传一张图片
4. 输入提示词描述运动
5. 提交表单
6. 等待生成

**预期结果**: 图像转换为动态视频

> **需要帮助？** 如果遇到问题，查看 [Level 5] 故障排除。

---

## [Level 5] 进阶内容

### Wan 2.2 提示词技巧

| 提示词类型 | 示例 |
|-----------|------|
| 基础动作 | "a cat walking through grass" |
| 摄像机运动 | "slow zoom in, cinematic" |
| 环境描述 | "sunset at the beach, golden hour" |
| 风格指定 | "anime style, vibrant colors" |
| 质量提升 | "high quality, detailed, 4K" |

### 负面提示词

```
low quality, blurry, distorted, watermark,
text, bad anatomy, ugly, poorly drawn
```

### ComfyUI 工作流自定义

可以修改 ComfyUI JSON 工作流文件来自定义：

- 调整视频分辨率
- 修改视频长度 (帧数)
- 添加后处理效果
- 使用不同的采样器

### Modal 定价

| 资源类型 | 价格 |
|----------|------|
| **免费额度** | $30/月 |
| **L40s GPU** | ~$0.57/小时 |
| **T4 GPU** | ~$0.22/小时 |

### 批量视频生成

使用 Loop 节点处理多个提示词：

```
[List of Prompts] ──► [Loop Over Items] ──► [Generate Video]
                                                │
                                                ▼
                                    [Save to Google Drive]
```

### 故障排除

| 问题 | 症状 | 可能原因 | 解决方案 |
|------|------|----------|----------|
| 服务器连接失败 | HTTP 错误 | Modal 服务器未部署 | 检查服务器 URL，重新部署 |
| 视频生成超时 | 请求超时 | 冷启动或负载高 | 增加等待时间或使用更小的模型 |
| 内存不足 | OOM 错误 | 14B 模型需要更多资源 | 切换到 5B 模型 |
| 视频质量差 | 输出不理想 | 提示词不够详细 | 改进提示词描述 |
| ComfyUI 错误 | workflow 执行失败 | JSON 格式问题 | 使用官方 ComfyUI 工作流 |

### 生产部署注意事项

**成本优化**:
- 使用 5B 模型进行测试
- 批量处理时监控 Modal 使用量
- 设置预算警报

**性能优化**:
- 使用 `modal serve` 持续运行避免冷启动
- 调整视频分辨率平衡速度和质量
- 实现请求队列

**安全建议**:
- 使用环境变量存储敏感信息
- 限制工作流的公开访问
- 实现速率限制

### 相关资源

**相关 Episode**:
- [Episode 22](../episode_22/) - 长视频生成
- [Episode 13](../episode_13/) - MiniMax Hailuo 视频
- [Episode 31](../episode_31/) - Veo 3.1 集成

**外部资源**:
- [Modal 文档](https://modal.com/docs)
- [ComfyUI 文档](https://docs.comfy.org/)
- [Wan 2.2 官方页面](https://huggingface.co/Wan-Video/Wan2.2)
- [ComfyUI Wan 工作流](https://docs.comfy.org/tutorials/video/wan/wan2_2)

---

## 资源下载

### n8n 工作流文件

下载并导入到 n8n：

- [n8n_wan_2.2_5b_t2v.json](./n8n_wan_2.2_5b_t2v.json) - 5B Text to Video
- [n8n_wan_2.2_5b_i2v.json](./n8n_wan_2.2_5b_i2v.json) - 5B Image to Video
- [n8n_wan_2.2_14b_t2v.json](./n8n_wan_2.2_14b_t2v.json) - 14B Text to Video
- [n8n_wan_2.2_14b_i2v.json](./n8n_wan_2.2_14b_i2v.json) - 14B Image to Video

**导入方法**:
1. 在 n8n 中点击右上角 "..." 菜单
2. 选择 "Import from File"
3. 选择下载的 JSON 文件

### Modal 服务器代码

- [modal_wan_comfyui_5b.py](./modal_wan_comfyui_5b.py) - 5B 模型
- [modal_wan_comfyui_14b_t2v.py](./modal_wan_comfyui_14b_t2v.py) - 14B Text to Video
- [modal_wan_comfyui_14b_i2v.py](./modal_wan_comfyui_14b_i2v.py) - 14B Image to Video

**部署方法**:
```bash
modal serve modal_wan_comfyui_5b.py  # 测试
modal deploy modal_wan_comfyui_5b.py  # 部署
```

### ComfyUI 工作流文件

- [comfyui_wan2_2_5B_ti2v.json](./comfyui_wan2_2_5B_ti2v.json)
- [comfyui_api_wan2_2_5B_t2v.json](./comfyui_api_wan2_2_5B_t2v.json)
- [comfyui_api_wan2_2_14B_t2v.json](./comfyui_api_wan2_2_14B_t2v.json)
- [comfyui_api_wan2_2_14B_i2v.json](./comfyui_api_wan2_2_14B_i2v.json)

---

## 观看视频

[![Wan 2.2 for FREE (NO GPU NEEDED) - Best VEO3 alternative](https://img.youtube.com/vi/rZ45_IhojLY/0.jpg)](https://www.youtube.com/watch?v=rZ45_IhojLY)

---

## 社区支持

遇到问题？加入社区获取帮助：

- [Skool 社区](https://www.skool.com/ai-agents-az/about)
- 获取 Premium 版本工作流

---

## 导航

| 你的需求 | 建议阅读 |
|----------|----------|
| 快速了解本集内容 | Level 1 |
| 决定是否学习本集 | Level 1-2 |
| 理解工作流原理 | Level 3 |
| 跟随教程构建 | Level 4 |
| 排查问题/生产部署 | Level 5 |

---

**Episode**: 20 | **版本**: v2.0 (分层解释版) | **最后更新**: 2025-01-17

**标签**: n8n, Wan 2.2, ComfyUI, Modal, video generation, text to video, image to video
