# Episode 17: Create Shopify product videos with Seedance, ElevenLabs, Latentsync, Flux Kontext and n8n

> AI Agents A-Z - 用 n8n 构建实用的 AI Agent
> 难度: ⭐⭐⭐ | 预估时间: 45 分钟

---

## [Level 1] 这一集在做什么？

本集教你用 n8n 自动化创建 **Shopify 产品视频** - 从产品图像到带有 AI 数字人讲解的完整视频。

**一句话**：像"电商视频工厂"一样，输入 Shopify 店铺 URL，自动获取产品图片并生成带有 AI 虚拟主播讲解的产品视频。

**适用场景**：
- Shopify 电商产品视频自动化
- 时尚类产品展示视频
- 社交媒体产品广告
- 更便宜的 Heygen 替代方案

> 💡 **快速判断**：如果你是**电商卖家**需要批量制作产品视频，这一集适合你。
> 想了解更多？继续阅读 [Level 2]。

---

## [Level 2] 核心概念

### 你会学到什么

| 序号 | 概念 | 说明 |
|------|------|------|
| 1 | **Shopify API 集成** | 从 Shopify 店铺获取产品信息 |
| 2 | **Seedance 视频** | 生成产品展示视频片段 |
| 3 | **Flux Kontext** | 生成一致的产品背景图像 |
| 4 | **Latentsync** | AI 数字人说话视频生成 |
| 5 | **ElevenLabs TTS** | 高质量语音合成 |

### 涉及的 n8n 节点

| 节点类型 | 用途 | 新手友好度 |
|----------|------|------------|
| Form Trigger | 收集店铺信息 | ⭐ 简单 |
| HTTP Request | 调用 Shopify API | ⭐⭐ 中等 |
| LLM Agent | 生成产品脚本 | ⭐⭐ 中等 |
| Loop Over Items | 并行处理产品 | ⭐⭐ 中等 |
| Switch | 条件分支 | ⭐⭐ 中等 |

### 涉及的外部服务

| 服务 | 免费额度/价格 | 难度 | 官网 |
|------|--------------|------|------|
| **Shopify** | 店主已有访问权限 | ⭐⭐ | [shopify.com](https://www.shopify.com) |
| **Fal.ai** | 按使用付费 | ⭐⭐ | [fal.ai](https://fal.ai/) |
| **ElevenLabs** | 按使用付费 | ⭐ | [elevenlabs.io](https://elevenlabs.io) |
| **OpenAI API** | 按使用付费 | ⭐ | [openai.com](https://openai.com/) |

> 💡 **了解够了？** 知道学什么就可以开始。继续阅读 [Level 3] 了解工作流结构。

---

## [Level 3] 工作流结构

### 工作流概览图

```
┌─────────────────────────────────────────────────────────────────┐
│           "Shopify Product Video Creator" 工作流                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  [Form Trigger] ──► [Setup defaults] ──► [Get Products]        │
│  店铺URL/搜索词      清理URL             调用Shopify API        │
│       │                  │                    │                │
│       │                  │                    ▼                │
│       │                  │            [Select Product]          │
│       │                  │            选择目标产品              │
│       │                  │                    │                │
│       │                  └────────────────────┘                │
│       │                                                │        │
│       ▼                                                ▼        │
│  [Generate Script] ──► [Generate Background] ──► [Create Video] │
│  LLM生成脚本          Flux生成背景          Seedance生成视频     │
│       │                    │                    │                │
│       │                    └────────────────────┘                │
│       │                                                │        │
│       ▼                                                ▼        │
│  [Add Voiceover] ──► [Create Talking Avatar] ──► [Final Video]  │
│  ElevenLabs配音      Latentsync数字人        合成输出            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 数据流

```
用户输入 (Shopify 店铺 URL + 搜索词)
    │
    ├──► 从 Shopify 获取产品列表
    │     └──► products: [{ title, image, price, ... }]
    │
    ├──► 用户选择或自动选择产品
    │
    ├──► LLM 生成产品介绍脚本
    │
    ├──► Flux Kontext 生成一致背景
    │
    ├──► Seedance 生成产品展示视频
    │
    ├──► ElevenLabs 生成配音
    │
    └──► Latentsync 创建数字人说话视频
```

### 节点说明

| 节点 | 类型 | 配置要点 | 数据输出 |
|------|------|----------|----------|
| **On form submission** | Form Trigger | 店铺 URL、搜索词 | 表单数据 |
| **Setup defaults** | Set | 清理 URL 格式 | `SHOPIFY_URL` |
| **Get the products** | HTTP Request | Shopify products.json API | 产品列表 |
| **Select product** | Switch/Limit | 选择单个产品 | 产品详情 |
| **Generate script** | LLM Agent | 使用 GPT-4.1 | 产品脚本 |
| **Generate background** | HTTP Request | Flux Kontext API | 背景图像 |
| **Create video** | HTTP Request | Seedance API | 产品视频 |
| **Generate voiceover** | HTTP Request | ElevenLabs API | 音频文件 |
| **Create talking avatar** | HTTP Request | Latentsync API | 数字人视频 |
| **Merge final** | HTTP Request | FFmpeg 合成 | 最终视频 |

> 💡 **准备就绪？** 理解工作流结构后，继续阅读 [Level 4] 开始构建。

---

## [Level 4] 构建步骤

### 前置准备

在开始之前，请确保：

- [ ] n8n 已安装并运行（访问 http://localhost:5678）
- [ ] Fal.ai 账号已创建 ([fal.ai](https://fal.ai/dashboard/keys)）
- [ ] ElevenLabs 账号已创建 ([elevenlabs.io](https://elevenlabs.io)）
- [ ] OpenAI API Key 已配置
- [ ] 目标 Shopify 店铺 URL

### 步骤 1: 获取 API 密钥

**目标**: 获取所有必需的 API 凭证

**操作**:

1. **Fal.ai API Key**:
   - 访问 [Fal.ai Dashboard](https://fal.ai/dashboard/keys)
   - 创建 API Key

2. **ElevenLabs API Key**:
   - 访问 [ElevenLabs](https://elevenlabs.io)
   - 创建 API Key

3. **OpenAI API Key**:
   - 访问 [OpenAI Platform](https://platform.openai.com/api-keys)
   - 创建 API Key

**验证**: 确保所有 API Key 已安全保存

---

### 步骤 2: 导入工作流

**目标**: 将工作流导入 n8n

**操作**:

1. 在 n8n 中点击右上角 **"..."** 菜单
2. 选择 **"Import from File"**
3. 选择 `shopify_ecommerce_product_videos.json`

**验证**: 工作流应该显示在画布上

---

### 步骤 3: 配置 API 凭证

**目标**: 连接所有外部服务

**操作**:

1. **配置 Fal.ai 凭证**:
   - 点击所有 Fal.ai HTTP Request 节点
   - 配置 httpHeaderAuth 凭证

2. **配置 ElevenLabs 凭证**:
   - 点击 ElevenLabs 相关节点
   - 配置 API Key

3. **配置 OpenAI 凭证**:
   - 点击 "OpenAI Chat Model" 节点
   - 配置 API 凭证

**验证**: 所有节点凭证已连接

---

### 步骤 4: 配置表单触发器

**目标**: 设置用户输入表单

**操作**:

1. 点击 **"On form submission"** 节点
2. 查看表单字段配置

**表单字段**:
- **Shopify store URL** (必需): 完整的 Shopify 店铺 URL
- **Search term** (必需): 用于搜索产品的关键词

---

### 步骤 5: 测试完整工作流

**目标**: 端到端验证

**操作**:

1. 激活工作流
2. 填写表单：
   - Shopify store URL: `https://example.myshopify.com`
   - Search term: `dress` 或其他产品类别
3. 提交表单
4. 等待处理（约 3-5 分钟）

**预期结果**: 工作流执行完成后返回带有 AI 数字人讲解的产品视频

> 💡 **需要帮助？** 如果遇到问题，查看 [Level 5] 故障排除。

---

## [Level 5] 进阶内容

### 服务成本对比

| 服务 | 相对成本 | 说明 |
|------|---------|------|
| **本方案** | 1x | Seedance + ElevenLabs + Latentsync |
| **HeyGen** | 5-10x | 专业数字人平台 |
| **传统制作** | 50x+ | 真人拍摄 + 后期 |

### ElevenLabs 语音选择

| 语音代码 | 性别 | 风格 | 适用场景 |
|----------|------|------|----------|
| `rachel` | 女 | 专业 | 产品介绍 |
| `josh` | 男 | 友好 | 休闲产品 |
| `eleven_multilingual_v2` | 混合 | 多语言 | 国际化 |

### Flux Kontext 背景生成

**背景风格选项**:
- Studio lighting（摄影棚灯光）
- Natural environment（自然环境）
- Minimalist（极简主义）
- Luxury（奢华风格）

### Latentsync 数字人配置

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `sync_mode` | `sync` | 同步模式 |
| `input_video` | 产品视频 | 背景视频 |
| `input_audio` | ElevenLabs 输出 | 语音文件 |

### 批量产品视频

创建批量处理工作流：

```
[Product List] ──► [Loop Over Items] ──► [Generate Video]
                                            │
                                            ▼
                                    [Save to Google Drive]
```

### 故障排除

| 问题 | 症状 | 可能原因 | 解决方案 |
|------|------|----------|----------|
| Shopify 访问失败 | 404/403 错误 | URL 格式错误或店铺不存在 | 检查店铺 URL 格式 |
| 产品未找到 | 空产品列表 | 搜索词不匹配 | 尝试不同的搜索词 |
| 数字人同步失败 | 音视频不同步 | Latentsync 服务问题 | 检查音频格式，重试 |
| 脚本质量差 | 介绍内容不理想 | LLM prompt 不够详细 | 修改脚本生成 prompt |
| 背景不一致 | 多个视频背景不同 | Kontext 配置问题 | 固定背景提示词 |

### 生产部署注意事项

**成本优化**:
- 批量处理时控制并发数
- 使用缓存避免重复生成
- 优先处理高价值产品

**性能优化**:
- 使用异步处理避免超时
- 设置合理的轮询间隔
- 考虑使用队列处理大量请求

**内容质量**:
- 优化产品选择逻辑
- 改进脚本生成模板
- 测试不同语音效果

### 相关资源

**相关 Episode**:
- [Episode 13](../episode_13/) - Hailuo 02 视频生成
- [Episode 14](../episode_14/) - Seedance 视频生成
- [Episode 23](../episode_23/) - UGC 视频制作

**外部资源**:
- [Shopify API 文档](https://shopify.dev/docs)
- [Fal.ai 文档](https://docs.fal.ai/)
- [ElevenLabs 文档](https://elevenlabs.io/docs)
- [加入 n8n](https://n8n.partnerlinks.io/fenoo5ekqs1g)

---

## 资源下载

### n8n 工作流文件

下载并导入到 n8n：

- [shopify_ecommerce_product_videos.json](./shopify_ecommerce_product_videos.json)

**导入方法**:
1. 在 n8n 中点击右上角 "..." 菜单
2. 选择 "Import from File"
3. 选择下载的 JSON 文件

### 附加资源

- [Fal.ai API keys](https://fal.ai/dashboard/keys)
- [加入 n8n](https://n8n.partnerlinks.io/fenoo5ekqs1g)

---

## 观看视频

[![Automate product videos for Shopify fashion stores - cheaper Heygen alternative (free n8n workflow)](https://img.youtube.com/vi/YCDNXaAIEtA/0.jpg)](https://youtu.be/YCDNXaAIEtA)

**时长**: ~25 分钟 | **更新日期**: 2025-01-16

---

## 社区支持

遇到问题？加入社区获取帮助：

- [Skool 社区](https://www.skool.com/ai-agents-az/about?gw11)
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

**Episode**: 17 | **版本**: v2.0 (分层解释版) | **最后更新**: 2025-01-16

**标签**: n8n, Shopify, product videos, Seedance, ElevenLabs, Latentsync, Flux Kontext, ecommerce
