# Episode 25: China is Winning the Open AI Model Race - Qwen and Wan 2.2

> AI Agents A-Z - 用 n8n 构建实用的 AI Agent
> 难度: ⭐⭐ | 预估时间: 35 分钟

---

## [Level 1] 这一集在做什么？

本集介绍阿里巴巴等中国公司发布的开源 AI 模型，包括 Qwen Image Edit Plus 和 Wan 2.2 系列视频模型，展示如何通过 Fal.ai 在 n8n 中使用这些模型。

**一句话**：像"模型探索者"一样，了解中国开源 AI 模型的强大功能，并将其集成到 n8n 工作流中。

**适用场景**：
- 使用 Qwen 进行图像编辑
- 使用 Wan 2.2 生成视频
- 通过 Fal.ai 简化模型调用
- 探索开源模型替代方案

> **快速判断**：如果你需要**使用中国开源 AI 模型**，这一集适合你。
> 想了解更多？继续阅读 [Level 2]。

---

## [Level 2] 核心概念

### 你会学到什么

| 序号 | 概念 | 说明 |
|------|------|------|
| 1 | **Qwen Image Edit Plus** | 阿里巴巴的图像编辑模型 |
| 2 | **Wan 2.2 系列** | 阿里巴巴的视频生成模型 |
| 3 | **Fal.ai 子工作流** | 创建可复用的模型调用模块 |
| 4 | **中国开源 AI 生态** | 了解 Qwen、Wan、Flux 等模型 |

### 涉及的 n8n 节点

| 节点类型 | 用途 | 新手友好度 |
|----------|------|------------|
| Execute Workflow Trigger | 子工作流触发器 | ⭐ 简单 |
| HTTP Request | 调用 Fal.ai API | ⭐⭐ 中等 |
| Form Trigger | 收集用户输入 | ⭐ 简单 |
| Code | 转换文件格式 | ⭐⭐ 中等 |
| Merge | 合并结果 | ⭐ 简单 |

### 涉及的外部服务

| 服务 | 免费额度 | 难度 | 官网 |
|------|----------|------|------|
| **Fal.ai** | 按使用付费 | ⭐⭐ | [fal.ai](https://fal.ai/) |
| **Qwen** | 开源 | ⭐ | [qwenlm.github.io](https://qwenlm.github.io/) |
| **Wan 2.2** | 开源 | ⭐ | [huggingface.co](https://huggingface.co/Wan-Video) |

### 模型对比

| 模型 | 公司 | 类型 | 特点 |
|------|------|------|------|
| **Qwen 2.5** | 阿里巴巴 | LLM | 强大的中文和编程能力 |
| **Qwen Image** | 阿里巴巴 | 图像生成 | 高质量图像 |
| **Qwen Image Edit Plus** | 阿里巴巴 | 图像编辑 | 精细编辑控制 |
| **Wan 2.2** | 阿里巴巴 | 视频生成 | Text/Image to Video |

> **了解够了？** 知道学什么就可以开始。继续阅读 [Level 3] 了解工作流结构。

---

## [Level 3] 工作流结构

### Qwen Image Edit Plus 子工作流

```
┌─────────────────────────────────────────────────────────────┐
│            "Qwen Image Edit Plus" 子工作流                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [Execute Workflow Trigger] ──► [HTTP Request]              │
│  子工作流触发器              POST 到 Fal.ai                 │
│       │                            │                        │
│       │                            ▼                        │
│       │                   [Convert to File]                 │
│       │                   转换响应为文件                     │
│       │                            │                        │
│       ▼                            ▼                        │
│  [Return to Main Workflow]                                   │
│  返回编辑后的图像                                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Wan 2.2 Animate Move 工作流

```
┌─────────────────────────────────────────────────────────────┐
│              "Wan 2.2 Animate Move" 工作流                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [Form Trigger] ──► [Convert to base64] ──► [HTTP Request]   │
│  用户表单            转换图像               调用 Fal.ai       │
│       │                  │                     │             │
│       │                  ▼                     ▼             │
│       │            [Prepare data] ──────────► [Poll status]   │
│       │            准备请求数据              轮询完成状态        │
│       │                  │                     │             │
│       │                  └─────────────────────┘             │
│       │                                                │       │
│       ▼                                                ▼       │
│  [Download video] ◄───────────────── [Complete?]              │
│  下载最终视频                      检查是否完成                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 数据流

```
Qwen Image Edit:
  图像 + 编辑指令 ──► Fal.ai ──► 编辑后的图像

Wan 2.2 Animate Move:
  视频 + 图像 ──► Fal.ai ──► 动画视频

Wan 2.2 Animate Replace:
  视频 + 替换图像 ──► Fal.ai ──► 替换动画视频
```

### 节点说明

| 节点 | 类型 | 配置要点 | 数据输出 |
|------|------|----------|----------|
| **Execute Workflow Trigger** | Trigger | 子工作流入口 | 输入参数 |
| **HTTP Request** | HTTP Request | POST 到 Fal.ai 端点 | 模型响应 |
| **Form Trigger** | Form | 收集图像和参数 | 表单数据 |
| **Convert to base64** | Code | 转换图像为 base64 | base64 数据 |
| **Poll request status** | HTTP Request | GET 检查任务状态 | {status, result} |
| **Convert to File** | Convert to File | 转换为二进制 | 文件数据 |

> **准备就绪？** 理解工作流结构后，继续阅读 [Level 4] 开始构建。

---

## [Level 4] 构建步骤

### 前置准备

在开始之前，请确保：

- [ ] n8n 已安装并运行（访问 http://localhost:5678）
- [ ] Fal.ai 账号已创建 ([fal.ai](https://fal.ai/)）
- [ ] Fal.ai API Key 已获取

### 步骤 1: 获取 Fal.ai API Key

**目标**: 配置 Fal.ai 认证

**操作**:

1. 访问 [fal.ai](https://fal.ai/)
2. 注册/登录账号
3. 进入 Settings > API Keys
4. 创建新的 API Key
5. 复制 Key

**验证**: API Key 格式正确

---

### 步骤 2: 导入 Qwen Image Edit Plus 子工作流

**目标**: 导入图像编辑子工作流

**操作**:

1. 在 n8n 中点击右上角 **"..."** 菜单
2. 选择 **"Import from File"**
3. 选择 `n8n_qwen_image_edit_2509.json`

**验证**: 工作流显示在画布上

---

### 步骤 3: 配置 Fal.ai 凭证

**目标**: 设置 API 认证

**操作**:

1. 在子工作流中找到 **HTTP Request** 节点
2. 配置 Header Auth 或 Key Auth 凭证
3. 输入 Fal.ai API Key

**验证**: 凭证已连接

---

### 步骤 4: 导入 Wan 2.2 工作流

**目标**: 导入视频生成工作流

**操作**:

1. 导入 `wan_22_animate_move.json`
2. 导入 `wan_22_animate_replace.json`

**验证**: 两个工作流都显示在画布上

---

### 步骤 5: 测试 Qwen Image Edit

**目标**: 编辑第一张图像

**操作**:

1. 从主工作流调用 Qwen 子工作流
2. 提供测试图像
3. 设置编辑提示词：
   - "Make the sky more dramatic"
   - "Add a sunset to the background"
4. 执行工作流
5. 查看编辑结果

**预期结果**: 获得编辑后的图像

---

### 步骤 6: 测试 Wan 2.2 Animate Move

**目标**: 生成第一个动画视频

**操作**:

1. 激活 Wan 2.2 工作流
2. 打开表单 URL
3. 上传视频和图像
4. 选择分辨率 (480p/580p/720p)
5. 提交表单
6. 等待生成（约 1-3 分钟）

**预期结果**: 获得动画视频

> **需要帮助？** 如果遇到问题，查看 [Level 5] 故障排除。

---

## [Level 5] 进阶内容

### 中国开源 AI 模型全景

| 公司 | 模型 | 类型 | 特点 |
|------|------|------|------|
| 阿里巴巴 | Qwen 2.5 | LLM | 强大中文能力 |
| 阿里巴巴 | Qwen Image | 图像 | 高质量生成 |
| 阿里巴巴 | Wan 2.2 | 视频 | 多功能视频模型 |
| 阶跃星辰 | Step-1-X | 图像 | 超高分辨率 |
| 智谱 AI | GLM-4 | LLM | 多模态能力 |
| 月之暗面 | Kimi | LLM | 长文本处理 |

### Fal.ai 支持的模型

**Qwen 系列**:
- `qwen-2.5-coder-32b-instruct` - 代码生成
- `qwen-2-vl-7b-instruct` - 视觉语言
- `qwen-image-edit` - 图像编辑

**Wan 2.2 系列**:
- `fal-ai/wan/v2.2-14b/animate/move` - 运动动画
- `fal-ai/wan/v2.2-14b/animate/replace` - 替换动画

### API 调用示例

**Qwen Image Edit**:
```json
{
  "image_url": "https://...",
  "prompt": "Add a sunset to the background",
  "negative_prompt": "low quality"
}
```

**Wan 2.2 Animate Move**:
```json
{
  "video_url": "data:video/mp4;base64,...",
  "image_url": "data:image/png;base64,...",
  "resolution": "720p"
}
```

### 成本对比

| 服务 | 大约成本 |
|------|----------|
| Fal.ai Qwen Edit | ~$0.01/张 |
| Fal.ai Wan 2.2 | ~$0.05/视频 |
| Modal 自部署 | $30/月无限 |

### 批量处理策略

```
[Image List] ──► [Loop Over Items] ──► [Qwen Edit Subworkflow]
                                             │
                                             ▼
                                [Save Edited Images]
```

### 故障排除

| 问题 | 症状 | 可能原因 | 解决方案 |
|------|------|----------|----------|
| Fal.ai 认证失败 | HTTP 401 | API Key 错误 | 验证 Key 格式 |
| 图像转换失败 | base64 错误 | 文件太大 | 压缩图像到 5MB 以下 |
| 视频生成超时 | processing 状态 | Fal.ai 负载高 | 增加轮询间隔 |
| 子工作流未执行 | "not found" | 未激活 | 激活子工作流 |
| 分辨率不支持 | 生成失败 | 设备不支持 | 选择较低分辨率 |

### 生产部署注意事项

**成本优化**:
- 使用 Modal 自部署降低成本
- 批量处理减少 API 调用
- 优化图像/视频大小

**性能优化**:
- 使用异步处理避免超时
- 实现结果缓存
- 并行处理多个请求

**安全建议**:
- 使用环境变量存储 API Keys
- 限制工作流的公开访问
- 实现内容审核

---

## 资源下载

### n8n 工作流文件

下载并导入到 n8n：

**子工作流**:
- [n8n_qwen_image_edit_2509.json](./n8n_qwen_image_edit_2509.json)

**完整工作流**:
- [wan_22_animate_move.json](./wan_22_animate_move.json)
- [wan_22_animate_replace.json](./wan_22_animate_replace.json)

**导入方法**:
1. 在 n8n 中点击右上角 "..." 菜单
2. 选择 "Import from File"
3. 选择下载的 JSON 文件

---

## 观看视频

[![China is winning the open AI model race](https://img.youtube.com/vi/IPU7hf-zoIk/0.jpg)](https://www.youtube.com/watch?v=IPU7hf-zoIk)

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

**Episode**: 25 | **版本**: v2.0 (分层解释版) | **最后更新**: 2025-01-17

**标签**: n8n, Qwen, Wan 2.2, Fal.ai, Chinese AI models, image editing, video generation
