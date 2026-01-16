# Episode 16: Create Short Poem Videos for TikTok

> AI Agents A-Z - 用 n8n 构建实用的 AI Agent
> 难度: ⭐⭐⭐ | 预估时间: 50 分钟

---

## [Level 1] 这一集在做什么？

本集教你用 n8n 自动化创建诗歌短视频，适用于 TikTok、YouTube Shorts 和 Facebook Reels。

**一句话**：像"视频工厂"一样，输入一首诗，自动生成包含 AI 图像、语音朗读和字幕的完整视频。

**适用场景**：
- TikTok/YouTube Shorts 内容批量创作
- 诗歌朗读频道自动化
- 社交媒体视频自动发布

> **快速判断**：如果你需要**自动化生成诗歌视频**，这一集适合你。
> 想了解更多？继续阅读 [Level 2]。

---

## [Level 2] 核心概念

### 你会学到什么

| 序号 | 概念 | 说明 |
|------|------|------|
| 1 | **LLM 脚本生成** | 将诗歌拆分为多个场景，生成画面描述 |
| 2 | **Together AI 图像生成** | 使用 FLUX.1 生成诗歌插图 |
| 3 | **Chatterbox TTS** | 高质量文字转语音引擎 |
| 4 | **视频合成与合并** | 将图像、音频、字幕合成为视频 |

### 涉及的 n8n 节点

| 节点类型 | 用途 | 新手友好度 |
|----------|------|------------|
| Form Trigger | 收集诗歌和素材 | ⭐ 简单 |
| Information Extractor | 生成场景脚本 | ⭐⭐ 中等 |
| Split Out | 拆分场景数组 | ⭐⭐ 中等 |
| HTTP Request | 调用 API | ⭐⭐ 中等 |
| Switch | 状态轮询 | ⭐⭐ 中等 |
| Loop Over Items | 循环处理场景 | ⭐⭐ 中等 |

### 涉及的外部服务

| 服务 | 免费额度 | 难度 | 官网 |
|------|----------|------|------|
| **Together AI** | 有免费额度 | ⭐⭐ | [together.ai](https://together.ai/) |
| **AI Agents No-Code Tools** | 自建服务器 | ⭐⭐⭐ | [github](https://github.com) |
| **Postiz** | 开源自建 | ⭐⭐ | [postiz.com](https://postiz.com/) |

> **了解够了？** 知道学什么就可以开始。继续阅读 [Level 3] 了解工作流结构。

---

## [Level 3] 工作流结构

### 工作流概览图

```
┌─────────────────────────────────────────────────────────────────────┐
│              "AI Poem Video Creator" 工作流                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  [Form Trigger] ──► [Configure me] ──► [Setup defaults]             │
│  用户输入表单      服务器URL配置           URL格式化                  │
│       │                 │                     │                      │
│       │                 └─────────────────────┼──────────┐          │
│       │                                       │          │          │
│       │                    ┌──────────────────┘          │          │
│       │                    │                                     │
│       │     ┌──────────────▼──────────────┐                   │
│       │     │     上传背景音乐/声音样本       │                   │
│       │     │    (可选，通过 If 判断)       │                   │
│       │     └───────────────────────────────┘                   │
│       │                    │                                     │
│       ▼                    ▼                                     │
│  [Create the script] ──► [Split Out] ──► [Loop Over Items]       │
│  生成场景脚本          拆分场景          循环处理每个场景           │
│       │                                       │                  │
│       │                  ┌──────────────────┴───────┐            │
│       │                  │                              │          │
│       │                  ▼                              ▼          │
│       │         [Generate AI image]          [Upload image]        │
│       │         使用 FLUX.1 生成图像          上传到服务器           │
│       │                  │                              │          │
│       │                  └──────────────┬───────────────┘          │
│       │                                 │                          │
│       │                                 ▼                          │
│       │                        [Start generating TTS]             │
│       │                        使用 Chatterbox 生成语音             │
│       │                                 │                          │
│       │                    ┌────────────┴─────────┐               │
│       │                    │                      │               │
│       │                    ▼                      ▼               │
│       │              [Get TTS status]        [TTS status switch]   │
│       │              轮询TTS状态             ready/processing      │
│       │                    │                      │               │
│       │                    └──────────────┬───────┘               │
│       │                                   │                       │
│       │                                   ▼                       │
│       │                        [Start generating video]            │
│       │                        生成带字幕的视频                     │
│       │                                   │                       │
│       │                    ┌──────────────┴─────────┐               │
│       │                    │                      │               │
│       │                    ▼                      ▼               │
│       │              [Get video status]      [Video switch]        │
│       │              轮询视频状态            ready/processing       │
│       │                                   │                       │
│       │                    ┌──────────────┴─────────┐               │
│       │                    │                      │               │
│       ▼                    ▼                      ▼               │
│  [Combine loop items] ──► [Start merging videos] ──► [Download]      │
│  合并所有场景            添加背景音乐并合并         下载视频           │
│       │                                   │                         │
│       └───────────────────────────────────┼─────────────────────┐   │
│                                           │                     │   │
│                                           ▼                     ▼   │
│                                    [Schedule with Postiz]    [Cleanup]│
│                                    发布到社交媒体              清理临时文件│
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 数据流

```
用户输入诗歌
    │
    ├──► LLM 拆分为场景
    │     └─── 每个场景包含：text, image_prompt
    │
    ├──► 循环处理每个场景
    │     ├──► 生成 AI 图像 (FLUX.1)
    │     ├──► 生成 TTS 语音 (Chatterbox)
    │     └──► 合成带字幕的视频
    │
    ├──► 合并所有场景
    │     └─── 添加背景音乐
    │
    └──► 发布到社交媒体
```

### 节点说明

| 节点 | 类型 | 配置要点 | 数据输出 |
|------|------|----------|----------|
| **On form submission** | Form Trigger | 收集诗歌、背景音乐、声音样本 | 表单数据 |
| **Configure me** | Set | 配置服务器 URL 和参数 | 配置数据 |
| **HTTP Request** | HTTP Request | 测试服务器健康状态 | 200 OK |
| **Upload background music** | HTTP Request | POST 到 `/api/v1/media/storage` | {file_id} |
| **Upload voice sample** | HTTP Request | POST 到 `/api/v1/media/storage` | {file_id} |
| **Create the script** | Information Extractor | 生成场景 JSON Schema | {scenes, title, tags} |
| **Split Out** | Split Out | 拆分 scenes 数组 | 单个场景 |
| **Generate AI image** | HTTP Request | POST 到 Together AI | 图像 URL |
| **Upload image** | HTTP Request | 上传到媒体服务器 | {file_id} |
| **Start generating TTS** | HTTP Request | POST 到 `/api/v1/media/audio-tools/tts/chatterbox` | {file_id} |
| **Get TTS status** | HTTP Request | GET 到 `/status` | {status} |
| **TTS status switch** | Switch | ready/processing/failed | 路由 |
| **Start generating video** | HTTP Request | POST 到 `/api/v1/media/video-tools/generate/tts-captioned-video` | {file_id} |
| **Get video status** | HTTP Request | GET 到 `/status` | {status} |
| **Video generation switch** | Switch | ready/processing/failed | 路由 |
| **Combine loop items** | Aggregate | 聚合所有场景数据 | [{video_id, tts_id, image_id}] |
| **Start merging videos** | HTTP Request | POST 到 `/api/v1/media/video-tools/merge` | {file_id} |
| **Get merge status** | HTTP Request | GET 到 `/status` | {status} |
| **Download the video** | HTTP Request | GET 下载最终视频 | 视频文件 |
| **Schedule YouTube video** | HTTP Request | POST 到 Postiz API | 发布确认 |

> **准备就绪？** 理解工作流结构后，继续阅读 [Level 4] 开始构建。

---

## [Level 4] 构建步骤

### 前置准备

在开始之前，请确保：

- [ ] n8n 已安装并运行（访问 http://localhost:5678）
- [ ] AI Agents No-Code Tools 服务器已部署
- [ ] Together AI API Key 已配置
- [ ] (可选) Postiz 已安装用于社交媒体发布

### 步骤 1: 部署 AI Agents No-Code Tools 服务器

**目标**: 部署视频生成后端服务

**操作**:

按照 `guide-start-server.md` 指南部署服务器。服务器需要包含：

- `/api/v1/media/storage` - 媒体文件存储
- `/api/v1/media/audio-tools/tts/chatterbox` - TTS 生成
- `/api/v1/media/video-tools/generate/tts-captioned-video` - 视频生成
- `/api/v1/media/video-tools/merge` - 视频合并
- `/health` - 健康检查

**验证**: 访问服务器 URL + `/health`，应返回 200 OK

---

### 步骤 2: 导入工作流

**目标**: 导入诗歌视频生成工作流

**操作**:

1. 在 n8n 中点击右上角 **"..."** 菜单
2. 选择 **"Import from File"**
3. 选择 `workflow_poem_videos.json`

**验证**: 工作流应该显示在画布上

---

### 步骤 3: 配置服务器连接

**目标**: 连接到 AI Agents No-Code Tools 服务器

**操作**:

1. 点击 **"Configure me"** 节点
2. 设置 `AI_AGENTS_NO_CODE_TOOLS_URL` 为你的服务器地址
3. 设置 `postiz_api_url` (如果使用 Postiz)
4. 配置 TTS 参数：
   - `chatterbox_exaggeration`: 0.5
   - `chatterbox_cfg_weight`: 0.5
   - `chatterbox_temperature`: 0.8

**验证**: HTTP Request 节点测试连接成功

---

### 步骤 4: 配置 Together AI

**目标**: 配置图像生成服务

**操作**:

1. 点击 **"Generate AI image"** 节点
2. 添加 Together AI Bearer Token 凭证
3. 确认模型为 `black-forest-labs/FLUX.1-schnell-Free`

**验证**: 凭证已连接

---

### 步骤 5: 配置 LLM

**目标**: 选择并连接 LLM

**操作**:

1. 找到 "Pick your choice of LLM" 区域
2. 选择一个 LLM 节点 (推荐 OpenRouter)
3. 配置 API 凭证
4. **重要**: 删除或禁用其他未使用的 LLM 节点

**验证**: LLM 节点凭证已连接

---

### 步骤 6: 测试完整工作流

**目标**: 端到端验证

**操作**:

1. 激活工作流
2. 打开表单 URL
3. 填写表单：
   - **The poem**: 输入一首公共领域的诗歌
   - **Background music**: (可选) 上传 MP3 文件
   - **Sample voice**: (可选) 上传声音样本文件
4. 提交表单
5. 等待处理（可能需要 5-10 分钟）

**预期结果**: 生成一个完整的诗歌视频，包含：
- AI 生成的插图
- TTS 语音朗读
- 字幕显示
- 背景音乐

> **需要帮助？** 如果遇到问题，查看 [Level 5] 故障排除。

---

## [Level 5] 进阶内容

### 艺术风格自定义

在 **Create the script** 节点中，修改 `ImagePromptTemplate` 可以自定义图像风格：

```
Art style: Soft, poetic, and slightly surreal art style
- Soft painterly texture
- Dreamy twilight
- Cottagecore
- Surreal elements
- Vintage atmosphere
- Whimsical
```

### TTS 参数调整

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `exaggeration` | 0.5 | 语音表现力 (0-1) |
| `cfg_weight` | 0.5 | 引导权重 |
| `temperature` | 0.8 | 随机性 (0-1) |

### 背景音乐控制

在 **Configure me** 节点中设置：

```json
"background_music_volume": 0.1  // 音量 (0-1)
```

### Postiz 社交媒体发布

工作流包含自动发布到 TikTok/YouTube 的功能：

1. 配置 **Postiz** 节点凭证
2. 确保服务器已部署（本地无法发布 TikTok）
3. 启用相关节点

### 故障排除

| 问题 | 症状 | 可能原因 | 解决方案 |
|------|------|----------|----------|
| 服务器连接失败 | "Server isn't ready" | 服务器未部署或 URL 错误 | 检查服务器 URL 格式 |
| 图像生成失败 | HTTP 错误 | Together API 问题 | 检查 API Key 和额度 |
| TTS 超时 | processing 状态 | 服务器负载过高 | 增加 Wait 时间 |
| 视频合并失败 | 错误返回 | 背景音乐格式问题 | 确保音乐是 MP3 格式 |
| LLM 输出格式错误 | 工作流中断 | JSON Schema 不匹配 | 检查 n8n 版本 >= 1.97.1 |

### 生产部署注意事项

**成本优化**:
- Together AI 有免费额度，用完后可以移除 `-Free` 后缀
- 使用更快的付费模型加速生成
- 监控服务器 GPU 使用量

**性能优化**:
- 调整场景数量减少处理时间
- 使用队列处理大批量请求
- 并行处理图像和 TTS 生成

**安全建议**:
- 使用环境变量存储 API Keys
- 限制表单公开访问
- 定期清理临时文件

### 相关资源

**相关 Episode**:
- [Episode 42](../episode_42/) - 说明视频生成
- [Episode 22](../episode_22/) - 长视频生成
- [Episode 13](../episode_13/) - MiniMax Hailuo TTS

**外部资源**:
- [Together AI 文档](https://docs.together.ai/)
- [Postiz 文档](https://docs.postiz.com/)
- [Chatterbox TTS GitHub](https://github.com/remsky/Kokoro-Fast-API)

---

## 资源下载

### n8n 工作流文件

下载并导入到 n8n：

- [workflow_poem_videos.json](./workflow_poem_videos.json)

**导入方法**:
1. 在 n8n 中点击右上角 "..." 菜单
2. 选择 "Import from File"
3. 选择下载的 JSON 文件

### 服务器指南

- [guide-start-server.md](./guide-start-server.md) - 服务器部署指南

### 示例素材

- [background_music.mp3](./background_music.mp3) - 示例背景音乐
- [voice_sample.mp3](./voice_sample.mp3) - 示例声音样本
- [overlay_black.mp4](./overlay_black.mp4) - 示例叠加层

---

## 观看视频

[![This $0 AI system creates TikTok videos on autopilot](https://img.youtube.com/vi/EbaErWRc7H4/0.jpg)](https://www.youtube.com/watch?v=EbaErWRc7H4)

---

## 社区支持

遇到问题？加入社区获取帮助：

- [Skool 社区](https://www.skool.com/ai-agents-az/about?gw11)
- 获取 Premium 版本工作流（包含更多功能）

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

**Episode**: 16 | **版本**: v2.0 (分层解释版) | **最后更新**: 2025-01-17

**标签**: n8n, TikTok, poem videos, FLUX.1, TTS, video generation, automation
