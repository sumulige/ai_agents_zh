# Episode 19: Run FLUX.1 Kontext [dev] with Modal.com

> AI Agents A-Z - 用 n8n 构建实用的 AI Agent
> 难度: ⭐⭐ | 预估时间: 25 分钟

---

## [Level 1] 这一集在做什么？

本集教你用 Modal.com 的免费 GPU 基础设施运行 FLUX.1 Kontext [dev] 图像编辑模型，并与 n8n 集成。

**一句话**：像"图像编辑器"一样，上传一张图片和提示词，让 AI 自动修改图像内容。

**适用场景**：
- 免费的图像到图像编辑
- AI 图像修改和重绘
- 无需本地 GPU 的 AI 图像处理

> **快速判断**：如果你需要**零成本运行 FLUX.1 Kontext**，这一集适合你。
> 想了解更多？继续阅读 [Level 2]。

---

## [Level 2] 核心概念

### 你会学到什么

| 序号 | 概念 | 说明 |
|------|------|------|
| 1 | **Modal.com** | 无服务器 GPU 平台，有 $30/月免费额度 |
| 2 | **FLUX.1 Kontext [dev]** | Black Forest Labs 的图像编辑模型 |
| 3 | **FastAPI** | Modal 上的 Python Web 框架 |
| 4 | **n8n 表单触发器** | 收集用户输入和图像 |

### 涉及的 n8n 节点

| 节点类型 | 用途 | 新手友好度 |
|----------|------|------------|
| Form Trigger | 收集图像和提示词 | ⭐ 简单 |
| Set | 配置 Modal 服务器 URL | ⭐ 简单 |
| Code | 中继图像数据 | ⭐⭐ 中等 |
| HTTP Request | 调用 Modal API | ⭐⭐ 中等 |
| Extract from File | 转换图像格式 | ⭐ 简单 |
| Form | 展示编辑后的图像 | ⭐ 简单 |

### 涉及的外部服务

| 服务 | 免费额度 | 难度 | 官网 |
|------|----------|------|------|
| **Modal** | $30/月 | ⭐⭐ | [modal.com](https://modal.com/) |
| **FLUX.1 Kontext** | 通过 Modal 免费 | ⭐ | [blackforestlabs.ai](https://blackforestlabs.ai/) |

> **了解够了？** 知道学什么就可以开始。继续阅读 [Level 3] 了解工作流结构。

---

## [Level 3] 工作流结构

### 工作流概览图

```
┌─────────────────────────────────────────────────────────────┐
│           "FLUX.1 Kontext Image Editing" 工作流              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [Form Trigger] ──► [Configure me] ──► [Setup default]      │
│  用户表单          设置服务器URL          URL格式化           │
│       │                  │                   │               │
│       │                  └───────────────────┘               │
│       │                                      │               │
│       ▼                                      ▼               │
│  [Relay uploaded image] ──────────────────────┘               │
│  中继图像数据                                              │
│       │                                                       │
│       ▼                                                       │
│  [Edit image using Flux Kontext]                              │
│  调用 Modal API 编辑图像                                      │
│       │                                                       │
│       ▼                                                       │
│  [Turn image to base64]                                      │
│  转换为 base64                                              │
│       │                                                       │
│       ▼                                                       │
│  [Map image data]                                            │
│  格式化图像数据                                              │
│       │                                                       │
│       ▼                                                       │
│  [Present image]                                             │
│  展示编辑结果                                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Modal 服务器架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Modal 服务器架构                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  n8n ──► FastAPI ──► FLUX.1 Kontext [dev]                  │
│                              │                              │
│                              └──► 图像编辑 (Image to Image)   │
│                                                             │
│  输入: 图像 + 提示词                                          │
│  输出: 编辑后的图像 (PNG)                                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 数据流

```
用户上传图像 + 提示词
    │
    ├──► 格式化服务器 URL
    │
    ├──► POST 到 Modal /edit_image
    │     └─── {prompt, image}
    │
    └──► 返回编辑后的图像
```

### 节点说明

| 节点 | 类型 | 配置要点 | 数据输出 |
|------|------|----------|----------|
| **On form submission** | Form Trigger | 收集图像文件和提示词 | {Image_to_edit, Prompt} |
| **Configure me** | Set | 设置 `modal_server_url` | {modal_server_url} |
| **Setup default** | Set | 格式化 URL (移除路径) | {SERVER_URL} |
| **Relay the uploaded image** | Code | 中继图像二进制数据 | 图像数据 |
| **Edit the image using Flux Kontext [dev]** | HTTP Request | POST `/edit_image` | 编辑后的图像 |
| **Turn image to base64** | Extract from File | 转换为 base64 | {data, mime} |
| **Map image data** | Set | 格式化为展示格式 | {mime, base64_data} |
| **Present image** | Form | 显示编辑后的图像 | HTML 图像 |

> **准备就绪？** 理解工作流结构后，继续阅读 [Level 4] 开始构建。

---

## [Level 4] 构建步骤

### 前置准备

在开始之前，请确保：

- [ ] n8n 已安装并运行（访问 http://localhost:5678）
- [ ] Modal 账号已创建 ([modal.com](https://modal.com/)）
- [ ] Modal CLI 已安装 (`pip install modal`)
- [ ] Python 3.11+ 已安装

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

**目标**: 在 Modal 上部署 FLUX.1 Kontext 服务

**操作**:

1. 下载 `flux_kontext_dev_modal.py` 文件

2. 部署服务器:
```bash
modal deploy flux_kontext_dev_modal.py
```

3. 复制输出的服务器 URL（格式：`https://xxx.modal.run`）

**验证**: 访问服务器 URL + `/`，应看到健康检查响应

**预期结果**: Modal 服务器运行中，可以接收图像编辑请求

---

### 步骤 3: 导入 n8n 工作流

**目标**: 导入图像编辑工作流

**操作**:

1. 在 n8n 中点击右上角 **"..."** 菜单
2. 选择 **"Import from File"**
3. 选择 `flux_kontext_dev_modal.json`

**验证**: 工作流应该显示在画布上

---

### 步骤 4: 配置 Modal 服务器 URL

**目标**: 连接到你的 Modal 服务器

**操作**:

1. 点击 **"Configure me"** 节点
2. 将 `modal_server_url` 替换为你的服务器 URL

**示例**: `https://gyoridavid--flux-kontext-fastapi-fastapi-app.modal.run`

**验证**: URL 格式正确，以 `https://` 开头

---

### 步骤 5: 配置 API 凭证

**目标**: 设置 Modal 认证

**操作**:

1. 点击 **"Edit the image using Flux Kontext [dev]"** 节点
2. 配置 Bearer Token 凭证
3. 使用 Modal API Token

**验证**: 凭证已连接

---

### 步骤 6: 测试工作流

**目标**: 编辑第一张图像

**操作**:

1. 激活工作流
2. 打开表单 URL
3. 上传一张图片
4. 输入提示词（如："make it look like a watercolor painting"）
5. 提交表单
6. 等待处理（约 10-30 秒）

**预期结果**: 显示编辑后的图像

> **需要帮助？** 如果遇到问题，查看 [Level 5] 故障排除。

---

## [Level 5] 进阶内容

### FLUX.1 Kontext 提示词技巧

| 提示词类型 | 示例 | 效果 |
|-----------|------|------|
| 风格转换 | "make it look like a watercolor painting" | 水彩风格 |
| 场景修改 | "add a sunset in the background" | 添加日落背景 |
| 质量提升 | "enhance the resolution and add detail" | 提升分辨率和细节 |
| 艺术风格 | "convert to oil painting style" | 油画风格 |
| 天气变化 | "make it a rainy day" | 变成雨天 |

### Modal 定价

| 套餐 | 价格 | GPU 时间 |
|------|------|----------|
| **免费** | $0/月 | $30 额度 |
| **按需** | $0.57/小时 | L40s GPU |

### 批量图像处理

修改工作流添加循环：

```
[Multiple Images] ──► [Loop Over Items] ──► [Edit each image]
                                                  │
                                                  ▼
                                    [Save to Folder/Google Drive]
```

### 自定义模型参数

在 Modal Python 代码中可以调整：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `num_inference_steps` | 28 | 推理步数（越多越慢但质量更高）|
| `guidance_scale` | 3.5 | 引导强度 |
| `max_sequence_length` | 512 | 最大序列长度 |

### 故障排除

| 问题 | 症状 | 可能原因 | 解决方案 |
|------|------|----------|----------|
| 服务器连接失败 | HTTP 错误 | Modal 服务器未部署 | 检查服务器 URL 格式 |
| 图像上传失败 | 请求超时 | 图像太大 | 压缩图像到 5MB 以下 |
| 编辑结果不理想 | 输出不符合预期 | 提示词不够清晰 | 尝试更详细的描述 |
| GPU 不足 | 部署失败 | Modal 额度用完 | 等待额度重置或升级账户 |
| 冷启动慢 | 首次请求超时 | 容器需要时间启动 | 首次请求后后续会快 |

### 生产部署注意事项

**成本优化**:
- Modal 新用户有 $30 免费额度
- 调整 `keep_warm` 参数保持容器活跃
- 监控使用量避免超支

**性能优化**:
- 使用 Modal Volume 缓存模型
- 调整 GPU 类型平衡速度和成本
- 实现请求队列避免冷启动

**安全建议**:
- 使用 Modal Secrets 管理敏感信息
- 限制工作流的公开访问
- 实现速率限制防止滥用

### 相关资源

**相关 Episode**:
- [Episode 40](../episode_40/) - Flux.2[dev] 集成
- [Episode 24](../episode_24/) - Modal 图像生成
- [Episode 41](../episode_41/) - Z-Image Turbo

**外部资源**:
- [Modal 文档](https://modal.com/docs)
- [FLUX.1 模型页面](https://blackforestlabs.ai/)
- [Flux Kontext GitHub](https://github.com/Black-Forest-Labs/flux)

---

## 资源下载

### n8n 工作流文件

下载并导入到 n8n：

- [flux_kontext_dev_modal.json](./flux_kontext_dev_modal.json)

**导入方法**:
1. 在 n8n 中点击右上角 "..." 菜单
2. 选择 "Import from File"
3. 选择下载的 JSON 文件

### Modal 服务器代码

- [flux_kontext_dev_modal.py](./flux_kontext_dev_modal.py)

**部署方法**:
```bash
modal deploy flux_kontext_dev_modal.py
```

---

## 观看视频

[![Run Flux Kontext 100% FREE without having a GPU](https://img.youtube.com/vi/ndMi2Mo7znA/0.jpg)](https://www.youtube.com/watch?v=ndMi2Mo7znA)

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

**Episode**: 19 | **版本**: v2.0 (分层解释版) | **最后更新**: 2025-01-17

**标签**: n8n, Modal, FLUX.1 Kontext, image editing, free GPU
