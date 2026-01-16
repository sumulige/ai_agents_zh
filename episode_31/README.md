# Episode 31: Google Veo 3.1 with n8n - Generate Videos for Free

> AI Agents A-Z - 用 n8n 构建实用的 AI Agent
> 难度: ⭐⭐⭐ | 预估时间: 40 分钟

---

## [Level 1] 这一集在做什么？

本集教你如何在 n8n 中使用 Google Vertex AI 调用 Veo 3.1 视频生成模型，完全通过 Google Cloud 的免费额度实现零成本视频生成。

**一句话**：像"视频工作室"一样，用 Google Veo 3.1 从文字或图像生成高质量视频。

**适用场景**：
- Text to Video - 从文字描述生成视频
- Image to Video - 从图像生成视频
- 社交媒体视频创作
- 营销视频自动化

> **快速判断**：如果你需要**使用 Google Veo 3.1 生成视频**，这一集适合你。
> 想了解更多？继续阅读 [Level 2]。

---

## [Level 2] 核心概念

### 你会学到什么

| 序号 | 概念 | 说明 |
|------|------|------|
| 1 | **Google Vertex AI** | Google 的 AI 模型托管平台 |
| 2 | **Veo 3.1** | Google 的视频生成模型 |
| 3 | **长轮询 (Long Running Operations)** | 处理异步视频生成任务 |
| 4 | **Google Cloud 免费额度** | 新用户 $300 额度 |

### 涉及的 n8n 节点

| 节点类型 | 用途 | 新手友好度 |
|----------|------|------------|
| Form Trigger | 收集用户输入 | ⭐ 简单 |
| Code | 转换图像格式 | ⭐⭐ 中等 |
| HTTP Request | 调用 Vertex AI API | ⭐⭐⭐ 高级 |
| Wait | 等待生成完成 | ⭐ 简单 |
| If | 条件判断状态 | ⭐ 简单 |
| Convert to File | 转换为文件 | ⭐ 简单 |

### 涉及的外部服务

| 服务 | 免费额度 | 难度 | 官网 |
|------|----------|------|------|
| **Google Cloud Vertex AI** | $300 新用户额度 | ⭐⭐⭐ | [cloud.google.com](https://cloud.google.com/vertex-ai) |
| **Google OAuth2** | 免费认证 | ⭐⭐ | [console.cloud.google.com](https://console.cloud.google.com) |

### Veo 3.1 参数说明

| 参数 | 选项 | 说明 |
|------|------|------|
| **Resolution** | 720p, 1080p | 视频分辨率 |
| **Aspect Ratio** | 16:9, 9:16 | 宽高比 |
| **Duration** | 4s, 6s, 8s | 视频长度 |
| **Generate Audio** | true/false | 是否生成音频 |

> **了解够了？** 知道学什么就可以开始。继续阅读 [Level 3] 了解工作流结构。

---

## [Level 3] 工作流结构

### 工作流概览图

```
┌─────────────────────────────────────────────────────────────────────┐
│                      "Veo 3.1 Video Generation" 工作流                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  [Form Trigger] ──► [Convert images to base64]                     │
│  用户表单              转换图像文件                                    │
│       │                      │                                       │
│       │                      ▼                                       │
│       │              [Configure me] ──► [Setup workflow]             │
│       │              配置参数              设置 API URL               │
│       │                      │                    │                  │
│       │                      └────────────────────┘                  │
│       │                             │                                │
│       ▼                             ▼                                │
│  [Start generating the video] ──────────────────────────────┐      │
│  POST /predictLongRunning                                      │      │
│       │                                                         │      │
│       ▼                                                         │      │
│  [Initial wait] ──────────────────┐                            │      │
│  等待 15 秒                        │                            │      │
│       │                            │                            │      │
│       ▼                            ▼                            │      │
│  [Get the status of the generation] ◄───────────────────────────┘      │
│  GET /fetchPredictOperation                                          │
│       │                                                             │
│       ▼                                                             │
│  [If - done?]                                                      │
│  检查是否完成                                                        │
│       │                                                             │
│       ├─── YES ──► [Convert to File]                               │
│       │                   转换为文件                                  │
│       │                                                             │
│       └─── NO ──► [Wait before polling again]                      │
│                      等待后继续轮询                                  │
│                           │                                        │
│                           └────────────────┐                        │
│                                            │                        │
│                                            ▼                        │
│                                     [Loop back to status]           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 数据流

```
用户输入
    │
    ├──► 提示词 (必需)
    │
    ├──► 分辨率 (720p/1080p)
    │
    ├──► 宽高比 (16:9/9:16)
    │
    ├──► 视频长度 (4s/6s/8s)
    │
    ├──► 起始图像 (可选)
    │
    ├──► 结束图像 (可选)
    │
    └──► 生成音频 (可选)
         │
         ▼
    POST 到 Vertex AI /predictLongRunning
         │
         ▼
    轮询状态检查
         │
         ▼
    下载生成的视频
```

### 节点说明

| 节点 | 类型 | 配置要点 | 数据输出 |
|------|------|----------|----------|
| **On form submission** | Form Trigger | 收集所有用户输入 | 表单数据 |
| **Convert images to base64** | Code | 将图像转为 base64 | {files: {...}} |
| **Configure me** | Set | 设置 PROJECT_ID, LOCATION_ID | 配置数据 |
| **Setup workflow** | Set | 构建 API endpoint URL | {URL} |
| **Start generating the video** | HTTP Request | POST predictLongRunning | {name: operation} |
| **Initial wait** | Wait | 等待 15 秒 | 延迟 |
| **Get the status of the generation** | HTTP Request | GET fetchPredictOperation | {done, response} |
| **If** | If | 检查 done === true | 路由 |
| **Wait before polling again** | Wait | 等待后重新检查 | 延迟 |
| **Convert to File** | Convert to File | 转换 base64 为文件 | 视频文件 |

> **准备就绪？** 理解工作流结构后，继续阅读 [Level 4] 开始构建。

---

## [Level 4] 构建步骤

### 前置准备

在开始之前，请确保：

- [ ] n8n 已安装并运行（访问 http://localhost:5678）
- [ ] Google Cloud 账号已创建
- [ ] Google Cloud Project 已创建
- [ ] Vertex AI API 已启用
- [ ] OAuth2 凭证已配置

### 步骤 1: 创建 Google Cloud 项目

**目标**: 设置 Google Cloud 环境

**操作**:

1. 访问 [Google Cloud Console](https://console.cloud.google.com)
2. 创建新项目或选择现有项目
3. 记录 **Project ID**
4. 启用 Vertex AI API:
   - 导航到 APIs & Services > Library
   - 搜索 "Vertex AI API"
   - 点击启用

**验证**: Vertex AI API 已在项目中启用

---

### 步骤 2: 配置 OAuth2 凭证

**目标**: 设置 n8n 与 Google 的认证

**操作**:

1. 在 Google Cloud Console 中:
   - 导航到 APIs & Services > Credentials
   - 创建 OAuth 2.0 客户端 ID
   - 应用类型选择 "Desktop app"
   - 下载客户端配置

2. 在 n8n 中:
   - 添加新凭证
   - 选择 "Google OAuth2 API"
   - 配置客户端 ID 和密钥
   - 授权范围: `https://www.googleapis.com/auth/cloud-platform`

**验证**: 凭证已连接

---

### 步骤 3: 导入工作流

**目标**: 导入 Veo 3.1 工作流

**操作**:

1. 在 n8n 中点击右上角 **"..."** 菜单
2. 选择 **"Import from File"**
3. 选择 `veo_31.json`

**验证**: 工作流显示在画布上

---

### 步骤 4: 配置项目参数

**目标**: 设置 Google Cloud 项目信息

**操作**:

1. 点击 **"Configure me"** 节点
2. 设置以下参数:
   - `PROJECT_ID`: 你的 Google Cloud Project ID
   - `LOCATION_ID`: us-central1 (或其他可用区域)
   - `API_ENDPOINT`: us-central1-aiplatform.googleapis.com
   - `MODEL_ID`: veo-3.1-generate-preview

**验证**: 参数格式正确

---

### 步骤 5: 测试视频生成

**目标**: 生成第一个视频

**操作**:

1. 激活工作流
2. 打开表单 URL
3. 填写表单:
   - **prompt**: "A cat playing with a ball in a garden"
   - **Resolution**: 720p
   - **Aspect ratio**: 16:9
   - **Video duration**: 4 seconds
   - **Generate audio**: 勾选
4. 提交表单
5. 等待生成（约 2-5 分钟）

**预期结果**: 获得一个 MP4 视频文件

> **需要帮助？** 如果遇到问题，查看 [Level 5] 故障排除。

---

## [Level 5] 进阶内容

### Veo 3.1 提示词技巧

| 提示词类型 | 示例 |
|-----------|------|
| 基础动作 | "A cat jumping over a fence" |
| 摄像机运动 | "Slow pan from left to right, cinematic" |
| 环境描述 | "Sunset at the beach, golden hour lighting" |
| 风格指定 | "Anime style, vibrant colors" |
| 质量提升 | "High quality, detailed, 4K" |

### 视频参数组合建议

| 用途 | 分辨率 | 宽高比 | 长度 |
|------|--------|--------|------|
| YouTube | 1080p | 16:9 | 6-8s |
| TikTok/Shorts | 720p | 9:16 | 4-6s |
| Instagram | 1080p | 1:1 | 4-8s |

### Google Cloud 免费额度

| 项目 | 额度 |
|------|------|
| **新用户** | $300 赠金 |
| **每月免费** | 取决于服务 |
| **Vertex AI** | 按使用计费 |

### 使用起始和结束图像

起始图像 (Start Image):
- 定义视频的第一帧
- 用于从静态图像生成视频

结束图像 (End Image):
- 定义视频的最后一帧
- 控制视频的结束状态

组合使用可以创建:
- 图像之间的过渡
- 故事板式视频
- 产品展示视频

### 批量视频生成

使用 Loop 节点处理多个提示词：

```
[Prompt List] ──► [Loop Over Items] ──► [Generate Video]
                                              │
                                              ▼
                                  [Save to Google Drive/YouTube]
```

### 故障排除

| 问题 | 症状 | 可能原因 | 解决方案 |
|------|------|----------|----------|
| OAuth 认证失败 | HTTP 401 | 凭证过期或错误 | 重新配置 OAuth2 |
| API 未启用 | HTTP 403 | Vertex AI 未启用 | 在 Console 中启用 API |
| 轮询超时 | processing 状态 | 生成时间过长 | 增加 Wait 时间 |
| 图像转换失败 | base64 错误 | 图像太大 | 压缩图像到 5MB 以下 |
| 配额不足 | HTTP 429 | 超出免费额度 | 等待重置或升级账户 |

### 生产部署注意事项

**成本优化**:
- 使用 720p 降低成本
- 限制视频长度到 4 秒
- 监控 API 使用量
- 设置预算警报

**性能优化**:
- 使用异步处理
- 实现请求队列
- 缓存常用结果

**安全建议**:
- 使用服务账号而非 OAuth
- 限制 API 密钥权限
- 实现内容审核

---

## 资源下载

### n8n 工作流文件

下载并导入到 n8n：

- [veo_31.json](./veo_31.json)

**导入方法**:
1. 在 n8n 中点击右上角 "..." 菜单
2. 选择 "Import from File"
3. 选择下载的 JSON 文件

---

## 观看视频

[![Veo 3.1 is now in n8n - how to use it for FREE](https://img.youtube.com/vi/bNzkWBOE37Y/0.jpg)](https://www.youtube.com/watch?v=bNzkWBOE37Y)

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

**Episode**: 31 | **版本**: v2.0 (分层解释版) | **最后更新**: 2025-01-17

**标签**: n8n, Google Veo 3.1, Vertex AI, video generation, text to video, image to video
