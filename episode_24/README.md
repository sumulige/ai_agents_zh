# Episode 24: Generate Images with Qwen Image, Flux.1 and Cloudflare Workers AI

> AI Agents A-Z - 用 n8n 构建实用的 AI Agent
> 难度: ⭐⭐ | 预估时间: 30 分钟

---

## [Level 1] 这一集在做什么？

本集教你如何使用 Modal.com 和 Cloudflare Workers AI 每天免费生成 1,000+ 张 AI 图像，完全集成在 n8n 中。

**一句话**：像"图像工厂"一样，用免费 GPU 基础设施批量生成高质量 AI 图像。

**适用场景**：
- 每月 30,000 张免费图像生成
- 批量内容创作
- 无本地 GPU 的 AI 图像处理
- TikTok 视频素材生成

> **快速判断**：如果你需要**零成本批量生成 AI 图像**，这一集适合你。
> 想了解更多？继续阅读 [Level 2]。

---

## [Level 2] 核心概念

### 你会学到什么

| 序号 | 概念 | 说明 |
|------|------|------|
| 1 | **Modal.com 免费额度** | $30/月免费 GPU 额度部署模型 |
| 2 | **Cloudflare Workers AI** | Workers 免费额度内的 Flux Schnell |
| 3 | **子工作流 (Subworkflows)** | 创建可复用的图像生成模块 |
| 4 | **多模型部署** | Flux.1 [dev], Flux Schnell, Qwen Image |

### 涉及的 n8n 节点

| 节点类型 | 用途 | 新手友好度 |
|----------|------|------------|
| Execute Workflow Trigger | 子工作流触发器 | ⭐ 简单 |
| HTTP Request | 调用 Modal/Cloudflare API | ⭐⭐ 中等 |
| Merge | 合并多个图像结果 | ⭐ 简单 |
| Convert to File | 转换图像格式 | ⭐ 简单 |
| Set | 设置参数 | ⭐ 简单 |

### 涉及的外部服务

| 服务 | 免费额度 | 难度 | 官网 |
|------|----------|------|------|
| **Modal** | $30/月 | ⭐⭐ | [modal.com](https://modal.com/) |
| **Cloudflare Workers AI** | Workers 免费额度内 | ⭐⭐ | [developers.cloudflare.com](https://developers.cloudflare.com) |
| **Qwen Image** | 通过 Modal 免费 | ⭐ | [qwenlm.github.io](https://qwenlm.github.io/) |

### 成本对比

| 服务 | 每日限额 | 每月限额 |
|------|----------|----------|
| Cloudflare Workers | ~10,000 张 | ~300,000 张 |
| Modal ($30) | ~1,000 张 | ~30,000 张 |
| **合计** | ~11,000 张 | ~330,000 张 |

> **了解够了？** 知道学什么就可以开始。继续阅读 [Level 3] 了解工作流结构。

---

## [Level 3] 工作流结构

### 主工作流概览图

```
┌─────────────────────────────────────────────────────────────┐
│              "Multi-Service Image Generation" 工作流          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [Manual/Webhook Trigger] ──► [Set Parameters]              │
│  触发器                  配置所有参数                          │
│       │                            │                        │
│       ▼                            ▼                        │
│  ┌───────────────────────────────────────────────┐           │
│  │           并行执行多个图像服务                   │           │
│  ├───────────────────────────────────────────────┤           │
│  │                                               │           │
│  ▼          ▼               ▼          ▼                    │
│  │          │               │          │                    │
│ [Cloudflare] [Modal Flux] [Modal Flux] [Modal Qwen]         │
│  Workers      Dev         Schnell      Image                │
│  子工作流     子工作流      子工作流     子工作流              │
│  │          │               │          │                    │
│  └──────────┴───────────────┴──────────┘                    │
│                   │                                          │
│                   ▼                                          │
│           [Merge Results]                                   │
│           合并所有图像结果                                    │
│                   │                                          │
│                   ▼                                          │
│           [Use/Save Images]                                 │
│           保存或使用图像                                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Modal 子工作流架构

```
┌─────────────────────────────────────────────────────────────┐
│                   Modal 服务器架构                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  n8n ──► Modal FastAPI ──► AI Model                        │
│                              │                              │
│                              └──► 图像生成                   │
│                                                             │
│  可用模型:                                                  │
│  - FLUX.1 [dev] - 高质量图像生成                             │
│  - FLUX.1 Schnell - 快速图像生成                             │
│  - Qwen Image - 阿里图像模型                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 数据流

```
用户输入提示词
    │
    ├──► Cloudflare Workers (Flux Schnell) ──► 图像 1
    │
    ├──► Modal Flux.1 [dev] ─────────────────► 图像 2
    │
    ├──► Modal Flux Schnell ─────────────────► 图像 3
    │
    └──► Modal Qwen Image ───────────────────► 图像 4
         │
         └──► 合并所有图像并保存
```

### 节点说明

| 节点 | 类型 | 配置要点 | 数据输出 |
|------|------|----------|----------|
| **Execute Workflow Trigger** | Trigger | 子工作流入口 | 输入参数 |
| **HTTP Request** | HTTP Request | POST 到 Modal/Cloudflare | 图像数据 |
| **Get accounts** | HTTP Request | GET Cloudflare 账户 ID | 账户信息 |
| **Check image size** | If | 验证尺寸 <= 1MP | 通过/拒绝 |
| **Convert to File** | Convert to File | 转换 base64 到文件 | 二进制数据 |
| **Merge** | Merge | 合并多个图像 | [{image, service}] |

> **准备就绪？** 理解工作流结构后，继续阅读 [Level 4] 开始构建。

---

## [Level 4] 构建步骤

### 前置准备

在开始之前，请确保：

- [ ] n8n 已安装并运行（访问 http://localhost:5678）
- [ ] Modal 账号已创建 ([modal.com](https://modal.com/)）
- [ ] Cloudflare 账号已创建
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

### 步骤 2: 部署 Modal 服务器

**目标**: 在 Modal 上部署图像生成服务

**操作**:

1. 下载 Modal Python 应用：
   - `modal_nunchaku_flux_dev.py` - FLUX.1 [dev]
   - `modal_nunchaku_flux_schnell.py` - FLUX.1 Schnell
   - `modal_nunchaku_qwen.py` - Qwen Image

2. 部署服务器:
```bash
modal deploy modal_nunchaku_flux_dev.py
modal deploy modal_nunchaku_flux_schnell.py
modal deploy modal_nunchaku_qwen.py
```

3. 复制输出的服务器 URL

**验证**: 访问服务器 URL + `/`，应看到健康检查响应

---

### 步骤 3: 导入 n8n 子工作流

**目标**: 导入图像生成子工作流

**操作**:

1. 依次导入以下文件：
   - `n8n_modal_flux_dev.json`
   - `n8n_modal_flux_schnell.json`
   - `n8n_modal_qwen.json`
   - `n8n_cloudflare_flux_schnell.json`

2. **重要**: 激活所有子工作流

**验证**: 每个子工作流都有 "Execute Workflow Trigger" 节点

---

### 步骤 4: 配置 Modal 服务器 URL

**目标**: 连接到 Modal 服务器

**操作**:

1. 在每个 Modal 子工作流中
2. 找到 **HTTP Request** 节点
3. 将 URL 替换为你的 Modal 服务器地址

**验证**: URL 格式正确，以 `https://` 开头

---

### 步骤 5: 配置 Cloudflare 凭证

**目标**: 设置 Cloudflare Workers AI

**操作**:

1. 在 Cloudflare 子工作流中
2. 配置 Bearer Token 凭证
3. 输入 Cloudflare API Token

**验证**: 凭证已连接

---

### 步骤 6: 测试图像生成

**目标**: 生成第一张图像

**操作**:

1. 激活主工作流
2. 手动执行工作流
3. 输入测试提示词：
   - Prompt: "A serene mountain landscape at sunset"
4. 执行工作流
5. 查看生成的图像

**预期结果**: 获得 4 张来自不同服务的图像

> **需要帮助？** 如果遇到问题，查看 [Level 5] 故障排除。

---

## [Level 5] 进阶内容

### TikTok 恐怖故事视频工作流

本集包含一个完整的工作流，用于创建 TikTok 恐怖故事视频：

- [n8n_tiktok_scary.json](./n8n_tiktok_scary.json)

该工作流结合了：
- 图像生成 (Flux/Qwen)
- TTS 语音生成
- 视频合成

### 各模型特点对比

| 模型 | 质量 | 速度 | 适用场景 |
|------|------|------|----------|
| **Flux.1 [dev]** | 最高 | 慢 | 高质量艺术图像 |
| **Flux Schnell** | 高 | 快 | 批量生成 |
| **Qwen Image** | 高 | 中 | 中文内容 |

### 成本优化策略

1. **优先级顺序**：
   - Cloudflare Workers (最便宜)
   - Modal Flux Schnell
   - Modal Flux Dev

2. **尺寸限制**：
   - Cloudflare: 最大 1MP (1024x1024)
   - Modal: 灵活但消耗更多额度

3. **批量处理**：
   - 合并请求减少 API 调用
   - 使用缓存避免重复生成

### 提示词优化

| 技巧 | 示例 |
|------|------|
| 风格指定 | "cinematic, photorealistic" |
| 质量提升 | "high quality, detailed, 8K" |
| 负面提示 | "low quality, blurry, distorted" |
| 艺术风格 | "oil painting, watercolor, digital art" |

### 免费额度最大化

```
Cloudflare Workers: ~300,000 图像/月
Modal ($30): ~30,000 图像/月
────────────────────────────────────
总计: ~330,000 图像/月
或 ~11,000 图像/天
```

### 故障排除

| 问题 | 症状 | 可能原因 | 解决方案 |
|------|------|----------|----------|
| Modal 部署失败 | CLI 错误 | Python 版本不兼容 | 使用 Python 3.10-3.11 |
| 子工作流未执行 | "Workflow not found" | 子工作流未激活 | 激活所有子工作流 |
| 图像尺寸错误 | 生成失败 | 尺寸超出限制 | Cloudflare 限制 1MP |
| API 认证失败 | HTTP 401/403 | Token 错误 | 验证凭证配置 |
| 额度耗尽 | 请求被拒绝 | Modal $30 用完 | 等待重置或升级 |

### 生产部署注意事项

**成本优化**:
- 优先使用 Cloudflare Workers
- 监控 Modal 使用量避免超支
- 调整图像尺寸平衡质量

**性能优化**:
- 并行调用多个服务
- 实现结果缓存
- 使用队列处理大批量请求

**安全建议**:
- 使用环境变量存储 API Keys
- 限制工作流的公开访问
- 实现内容过滤

---

## 资源下载

### n8n 子工作流文件

下载并导入到 n8n：

- [n8n_cloudflare_flux_schnell.json](./n8n_cloudflare_flux_schnell.json)
- [n8n_modal_flux_dev.json](./n8n_modal_flux_dev.json)
- [n8n_modal_flux_schnell.json](./n8n_modal_flux_schnell.json)
- [n8n_modal_qwen.json](./n8n_modal_qwen.json)
- [n8n_tiktok_scary.json](./n8n_tiktok_scary.json) - 完整视频工作流

**导入方法**:
1. 在 n8n 中点击右上角 "..." 菜单
2. 选择 "Import from File"
3. 选择下载的 JSON 文件

### Modal 服务器代码

- [modal_nunchaku_flux_dev.py](./modal_nunchaku_flux_dev.py)
- [modal_nunchaku_flux_schnell.py](./modal_nunchaku_flux_schnell.py)
- [modal_nunchaku_qwen.py](./modal_nunchaku_qwen.py)

**部署方法**:
```bash
modal deploy modal_nunchaku_flux_dev.py
```

---

## 观看视频

[![100% FREE AI image generation with n8n (Flux and Qwen)](https://img.youtube.com/vi/2tycZNP5_IA/0.jpg)](https://www.youtube.com/watch?v=2tycZNP5_IA)

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

**Episode**: 24 | **版本**: v2.0 (分层解释版) | **最后更新**: 2025-01-17

**标签**: n8n, Modal, Cloudflare, Flux.1, Qwen Image, free image generation
