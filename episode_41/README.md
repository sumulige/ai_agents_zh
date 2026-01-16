# Episode 41: FREE Z-Image-Turbo with n8n

> AI Agents A-Z - 用 n8n 构建实用的 AI Agent
> 难度: ⭐ | 预估时间: 20 分钟

---

## [Level 1] 这一集在做什么？

本集教你用 n8n **零成本使用 Z-Image-Turbo** 生成高质量 AI 图像。

**一句话**：像"一键生成器"一样，输入文字描述，几秒钟内获得 AI 生成的图像。

**适用场景**：
- 快速原型设计和概念图
- 社交媒体图片批量生成
- 无需付费 API 的图像生成

> 💡 **快速判断**：如果你需要**免费且快速**的 AI 图像生成，这一集适合你。
> 想了解更多？继续阅读 [Level 2]。

---

## [Level 2] 核心概念

### 你会学到什么

| 序号 | 概念 | 说明 |
|------|------|------|
| 1 | **Z-Image-Turbo** | 阿里巴巴开源的超快速文本生成图像模型 |
| 2 | **Modal 免费 GPU** | 使用 Modal.com 免费额度运行模型 |
| 3 | **n8n HTTP 调用** | 通过 HTTP 请求调用 Modal API |
| 4 | **Apache-2.0 许可** | 完全开源，可商用 |

### 涉及的 n8n 节点

| 节点类型 | 用途 | 新手友好度 |
|----------|------|------------|
| Manual Trigger | 手动触发工作流 | ⭐ 简单 |
| HTTP Request | 调用 Modal API | ⭐ 简单 |
| Convert to File | 将二进制转为文件 | ⭐ 简单 |

### 涉及的外部服务

| 服务 | 免费额度 | 难度 | 官网 |
|------|----------|------|------|
| **Modal** | $30 免费额度/月 | ⭐ | [modal.com](https://modal.com/) |
| **Z-Image-Turbo** | 通过 Modal 免费使用 | ⭐ | [HuggingFace](https://huggingface.co/Tongyi-MAI/Z-Image-Turbo) |

> 💡 **了解够了？** 知道学什么就可以开始。继续阅读 [Level 3] 了解工作流结构。

---

## [Level 3] 工作流结构

### 工作流概览图

```
┌─────────────────────────────────────────────────────────────┐
│              "Test Z-Image-Turbo" 工作流                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [Manual Trigger] ──► [HTTP Request] ──► [Convert to File] │
│  点击执行             调用Modal API        保存为文件        │
│                           │                                 │
│                           │                                 │
│                           ▼                                 │
│                    Modal 服务器                             │
│                    (Z-Image-Turbo)                          │
│                           │                                 │
│                           ▼                                 │
│                    返回图像二进制                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Modal 服务器架构

```
┌─────────────────────────────────────────────────────────────┐
│                  Modal 服务器架构                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  n8n ──► Flask API ──► ZImageTurboModel                    │
│                              │                              │
│                              └──► 生成图像 (1024x1024)       │
│                                                             │
│  输入: prompt, width, height, steps                        │
│  输出: PNG 图像二进制                                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 数据流

```
用户点击执行
    │
    ├──► HTTP POST 到 Modal 服务器
    │     └─── { prompt: "a pirate cat", width: 1024, ... }
    │
    ├──► Modal 返回图像 (base64/二进制)
    │
    └──► 转换为可下载的文件
```

### 节点说明

| 节点 | 类型 | 配置要点 | 数据输出 |
|------|------|----------|----------|
| **When clicking 'Execute workflow'** | Manual Trigger | 点击手动执行 | 触发信号 |
| **Generate image** | HTTP Request | POST 到 Modal `/generate` | 图像二进制 |
| **Convert to File** | Convert to File | `image` 字段转二进制 | PNG 文件 |

> 💡 **准备就绪？** 理解工作流结构后，继续阅读 [Level 4] 开始构建。

---

## [Level 4] 构建步骤

### 前置准备

在开始之前，请确保：

- [ ] n8n 已安装并运行（访问 http://localhost:5678）
- [ ] Modal 账号已创建 ([modal.com](https://modal.com/)）
- [ ] Modal CLI 已安装 (`pip install modal`)
- [ ] Python 3.11+ 已安装

### 步骤 1: 部署 Modal 服务器

**目标**: 在 Modal 上部署 Z-Image-Turbo 服务

**操作**:

1. 下载 `modal_z_image_fast.py` 文件

2. 安装 Modal CLI:
```bash
pip install modal
```

3. 登录 Modal:
```bash
modal token new
```

4. 部署服务器:
```bash
modal deploy modal_z_image_fast.py
```

5. 复制服务器 URL（格式：`https://your-app.modal.run`）

**验证**: 访问服务器 URL + `/`，应该看到健康检查响应

**预期结果**: Modal 服务器运行中，可以接收图像生成请求

---

### 步骤 2: 导入 n8n 工作流

**目标**: 将工作流导入 n8n

**操作**:

1. 在 n8n 中点击右上角 **"..."** 菜单
2. 选择 **"Import from File"**
3. 选择 `n8n_test_z_image_turbo.json`

**验证**: 工作流应该显示在画布上，包含 3 个节点

---

### 步骤 3: 配置 HTTP Request 节点

**目标**: 连接到你的 Modal 服务器

**操作**:

1. 点击 **"Generate image"** 节点
2. 将 URL 中的 `<<insert your modal.com url here>>` 替换为你的服务器 URL
3. 验证 JSON Body 格式：
```json
{
  "prompt": "a pirate cat",
  "width": 1024,
  "height": 1024,
  "steps": 9
}
```

**验证**: URL 格式正确，如 `https://xxx.modal.run/generate`

---

### 步骤 4: 测试工作流

**目标**: 生成第一张图像

**操作**:

1. 点击工作流左上角的 **"Execute workflow"** 按钮
2. 等待执行完成（约 5-10 秒）
3. 查看 **"Convert to File"** 节点输出
4. 点击下载生成的图像

**预期结果**: 获得一张 1024x1024 的 PNG 图像

---

### 步骤 5: 自定义提示词

**目标**: 生成你想要的图像

**操作**:

修改 HTTP Request 节点中的 JSON Body：
```json
{
  "prompt": "你的描述文字",
  "width": 1024,
  "height": 1024,
  "steps": 9
}
```

**提示词技巧**:
- 描述主体：`"a cat wearing sunglasses"`
- 添加风格：`"in cyberpunk style"`
- 指定颜色：`"blue and orange color palette"`

> 💡 **需要帮助？** 如果遇到问题，查看 [Level 5] 故障排除。

---

## [Level 5] 进阶内容

### 调整图像参数

| 参数 | 默认值 | 说明 | 可选值 |
|------|--------|------|--------|
| `prompt` | `"a pirate cat"` | 文本描述 | 任意文本 |
| `width` | 1024 | 图像宽度 | 512-2048 (16的倍数) |
| `height` | 1024 | 图像高度 | 512-2048 (16的倍数) |
| `steps` | 9 | 生成步数 | 4-20 (越多越精细但越慢) |

### Z-Image-Turbo vs Flux.2

| 特性 | Z-Image-Turbo | Flux.2 |
|------|---------------|-------|
| **速度** | 极快 (~1秒) | 较快 (~3秒) |
| **质量** | 优秀 | 卓越 |
| **许可** | Apache-2.0 (可商用) | Flux-1 许可 |
| **成本** | 免费 (Modal) | 付费为主 |

### 批量生成图像

创建循环工作流：

```
[List of Prompts] ──► [Loop Over Items] ──► [Generate Image]
                                              │
                                              ▼
                                    [Save to Folder/Google Drive]
```

### 使用 Google Colab 替代

如果不想部署 Modal，可以使用 Google Colab：

1. 访问 [Z-Image-jupyter](https://github.com/camenduru/Z-Image-jupyter)
2. 在 Colab 中打开 notebook
3. 运行所有单元格
4. 输入提示词生成图像

### 故障排除

| 问题 | 症状 | 可能原因 | 解决方案 |
|------|------|----------|----------|
| 服务器连接失败 | HTTP 错误 | Modal 服务器未部署或 URL 错误 | 检查 URL 格式，确保包含 `https://` |
| 图像格式错误 | 无法下载图像 | Convert to File 配置错误 | 确保源属性设置为 `image` |
| 生成速度慢 | 超过 30 秒 | Modal 冷启动 | 首次请求会慢，后续会快 |
| 图像质量差 | 输出不理想 | steps 参数太低 | 尝试增加到 15-20 |

### 生产部署注意事项

**成本优化**:
- Modal 新用户有 $30 免费额度
- Z-Image-Turbo 经过量化，GPU 占用低
- 监控 Modal 使用量避免超支

**性能优化**:
- 调整图像尺寸以平衡质量和速度
- 使用 Modal Volume 缓存模型
- 考虑批量处理以提高效率

**安全建议**:
- 不要在公开工作流中暴露 Modal URL
- 使用 API 密钥保护生产环境
- 定期轮换访问凭证

### 相关资源

**相关 Episode**:
- [Episode 42](../episode_42/) - Z-Image 制作说明视频
- [Episode 40](../episode_40/) - Flux.2[dev] 集成
- [Episode 24](../episode_24/) - 更多图像生成选项

**外部资源**:
- [Modal 文档](https://modal.com/docs)
- [Z-Image-Turbo 模型](https://huggingface.co/Tongyi-MAI/Z-Image-Turbo)
- [Google Colab 示例](https://github.com/camenduru/Z-Image-jupyter)

---

## 资源下载

### n8n 工作流文件

下载并导入到 n8n：

- [n8n_test_z_image_turbo.json](./n8n_test_z_image_turbo.json)

**导入方法**:
1. 在 n8n 中点击右上角 "..." 菜单
2. 选择 "Import from File"
3. 选择下载的 JSON 文件

### Modal 服务器代码

- [modal_z_image_fast.py](./modal_z_image_fast.py)

**部署方法**:
```bash
modal deploy modal_z_image_fast.py
```

---

## 观看视频

[![Z-Image-Turbo is FAST - how to use it for FREE
](https://img.youtube.com/vi/R8D1uBDfR1Q/0.jpg)](https://www.youtube.com/watch?v=R8D1uBDfR1Q)

**时长**: ~10 分钟 | **更新日期**: 2025-01-16

---

## 社区支持

遇到问题？加入社区获取帮助：

- [Skool 社区](https://www.skool.com/ai-agents-az/about)
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

**Episode**: 41 | **版本**: v2.0 (分层解释版) | **最后更新**: 2025-01-16

**标签**: n8n, Z-Image-Turbo, image generation, Modal, free, AI art
