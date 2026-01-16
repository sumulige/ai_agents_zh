# Episode 3: Create LinkedIn Posts with Human-in-the-Loop

> AI Agents A-Z - 用 n8n 构建实用的 AI Agent
> 难度: ⭐⭐ | 预估时间: 40 分钟

---

## [Level 1] 这一集在做什么？

本集教你用 n8n 自动化创建 **LinkedIn 帖子**，并加入**人工审批流程**（Human-in-the-Loop）。

**一句话**：像"内容助手"一样，AI 自动生成帖子，你审核批准后发布。

**适用场景**：
- 定期 LinkedIn 内容创作
- 企业社交媒体管理
- 需要人工审核的 AI 生成内容

> 💡 **快速判断**：如果你需要**自动化 LinkedIn 内容但保留控制权**，这一集适合你。
> 想了解更多？继续阅读 [Level 2]。

---

## [Level 2] 核心概念

### 你会学到什么

| 序号 | 概念 | 说明 |
|------|------|------|
| 1 | **Human-in-the-Loop (HIL)** | AI 生成 + 人工审核的工作流 |
| 2 | **RSS 内容聚合** | 从新闻源获取内容灵感 |
| 3 | **LLM 内容生成** | 使用 AI 创作社交媒体帖子 |
| 4 | **Gmail 等待响应** | 通过邮件收集审批反馈 |
| 5 | **条件分支** | 根据批准/拒绝执行不同操作 |

### 涉及的 n8n 节点

| 节点类型 | 用途 | 新手友好度 |
|----------|------|------------|
| Manual Trigger | 手动测试工作流 | ⭐ 简单 |
| RSS Feed Read | 读取 AI 新闻 | ⭐ 简单 |
| LLM Chain | 生成/修改内容 | ⭐⭐ 中等 |
| Structured Output Parser | JSON 格式输出 | ⭐⭐ 中等 |
| Gmail sendAndWait | 发送邮件并等待回复 | ⭐⭐ 中等 |
| If | 条件判断 | ⭐⭐ 中等 |

### 涉及的外部服务

| 服务 | 免费额度 | 难度 | 官网 |
|------|----------|------|------|
| **OpenAI** | 按使用付费 | ⭐ | [openai.com](https://openai.com/) |
| **Gmail** | 免费 | ⭐ | [gmail.com](https://gmail.com/) |
| **AI News Feed** | 免费 | ⭐ | [artificialintelligence-news.com](https://www.artificialintelligence-news.com/) |

> 💡 **了解够了？** 知道学什么就可以开始。继续阅读 [Level 3] 了解工作流结构。

---

## [Level 3] 工作流结构

### 工作流概览图

```
┌─────────────────────────────────────────────────────────────┐
│           "LinkedIn Post with Approval" 工作流                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [Manual Trigger]                                           │
│  开始工作流                                                  │
│       │                                                     │
│       ▼                                                     │
│  [RSS Read] ──► AI 新闻源                                   │
│       │                                                     │
│       ▼                                                     │
│  [Edit Fields] ──► 提取标题/链接/摘要                        │
│       │                                                     │
│       ▼                                                     │
│  [Aggregate] ──► 聚合所有文章                               │
│       │                                                     │
│       ▼                                                     │
│  [LLM Chain] ──► 筛选 5 篇 SMB 相关文章                     │
│  (+ Structured Output)                                      │
│       │                                                     │
│       ▼                                                     │
│  [Split Out] ──► 分离每篇文章                               │
│       │                                                     │
│       ▼                                                     │
│  [HTTP Request] ──► 获取文章完整内容                         │
│       │                                                     │
│       ▼                                                     │
│  [HTML Extract] ──► 提取正文                                │
│       │                                                     │
│       ▼                                                     │
│  [Aggregate] ──► 重新聚合                                   │
│       │                                                     │
│       ▼                                                     │
│  [LLM Chain1] ──► 生成 LinkedIn 帖子                        │
│       │                                                     │
│       ▼                                                     │
│  [LLM Chain2] ──► 匹配写作风格                              │
│       │                                                     │
│       ▼                                                     │
│  [Gmail sendAndWait] ──► 发送审批邮件                       │
│       │                                                     │
│       ├───► [用户批准] ──► [If: yes] ──► 发布/完成           │
│       │                                                     │
│       └───► [用户拒绝] ──► [If: no] ──► [LLM 修改] ──► 循环  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 数据流

```
启动工作流
    │
    ├──► 获取 AI 新闻 RSS
    │     └─── 5 篇 SMB 相关文章
    │
    ├──► 抓取完整内容
    │     └─── HTML 提取
    │
    ├──► 生成 LinkedIn 帖子
    │     └─── 添加吸引人的标题和摘要
    │
    ├──► 匹配写作风格
    │     └─── 参考示例帖子
    │
    ├──► 发送审批邮件
    │     └─── Gmail 表单: 批准/拒绝 + 修改意见
    │
    └──► 根据反馈行动
          ├─── 批准 → 发布帖子
          └─── 拒绝 → 应用修改 → 重新发送
```

### 节点说明

| 节点 | 类型 | 配置要点 | 数据输出 |
|------|------|----------|----------|
| **When clicking Test** | Manual Trigger | 手动启动 | 触发信号 |
| **RSS Read** | RSS Feed | AI News RSS | 文章列表 |
| **LLM Chain** | AI Chain | 筛选 SMB 相关文章 | 5 篇文章 JSON |
| **Structured Output Parser** | Output Parser | `{articles: [{title, link}]}` | 结构化输出 |
| **HTTP Request** | HTTP | 抓取文章 URL | HTML 内容 |
| **HTML Extract** | HTML | CSS 选择器提取 | 正文文本 |
| **LLM Chain1** | AI Chain | 生成 LinkedIn 帖子 | 帖子文本 |
| **LLM Chain2** | AI Chain | 匹配写作风格 | 风格化帖子 |
| **Gmail** | Gmail | sendAndWait 模式 | 用户响应 |

### 人工审批表单

Gmail 节点创建的表单包含：
- **Do you approve the post?** (下拉选择: yes/no)
- **Change requests** (文本框: 修改意见)

> 💡 **准备就绪？** 理解工作流结构后，继续阅读 [Level 4] 开始构建。

---

## [Level 4] 构建步骤

### 前置准备

在开始之前，请确保：

- [ ] n8n 已安装并运行
- [ ] OpenAI API Key 已配置
- [ ] Gmail OAuth 已连接

### 步骤 1: 导入工作流

**目标**: 导入 LinkedIn 帖子工作流

**操作**:

1. 在 n8n 中点击 **"..."** 菜单
2. 选择 **"Import from File"**
3. 选择 `Create_a_LinkedIn_post__with_approval_updated.json`

**验证**: 工作流显示在画布上

---

### 步骤 2: 配置内容源

**目标**: 设置新闻来源

**操作**:

1. 点击 **"RSS Read"** 节点
2. 确认 URL: `https://www.artificialintelligence-news.com/feed/`
3. 可选: 更换为其他 RSS 源

**验证**: RSS 能正常读取

---

### 步骤 3: 配置写作风格

**目标**: 设置你的写作风格参考

**操作**:

1. 找到 **"Writing style"** 节点
2. 修改 `your_style` 字段
3. 输入你喜欢的 LinkedIn 帖子示例

**验证**: 风格示例已保存

---

### 步骤 4: 配置 Gmail 审批

**目标**: 设置审批邮件

**操作**:

1. 点击 **"Gmail"** 节点
2. 确认已连接 Gmail 账号
3. 配置表单字段：
   - 下拉选择: yes/no
   - 文本框: 修改意见

**验证**: Gmail 节点已连接

---

### 步骤 5: 测试工作流

**目标**: 端到端测试

**操作**:

1. 点击 **"Test workflow"**
2. 等待邮件到达
3. 测试两种场景：
   - 选择 "yes" → 批准通过
   - 选择 "no" + 修改意见 → 重新生成

**预期结果**:
- 能正确生成 LinkedIn 帖子
- 审批流程工作正常
- 修改意见能被正确应用

> 💡 **需要帮助？** 如果遇到问题，查看 [Level 5] 故障排除。

---

## [Level 5] 进阶内容

### 自定义帖子生成

**修改 LLM Chain1 提示词**:
```
创建一个关于 [主题] 的 LinkedIn 帖子
- 目标受众: [你的受众]
- 语气: [专业/轻松/幽默]
- 长度: [短/中/长]
- 包含话题标签: 是/否
```

### 添加图片生成

```
[LLM 生成帖子] ──► [提取关键概念] ──► [DALL-E] ──► [附加图片]
```

### 自动发布到 LinkedIn

```
[If: 批准] ──► [LinkedIn 节点] ──► 发布帖子
```

### 批量生成模式

```
[多个主题] ──► [Loop] ──► [生成帖子] ──► [批量审批]
```

### 故障排除

| 问题 | 症状 | 可能原因 | 解决方案 |
|------|------|----------|----------|
| RSS 读取失败 | 空输出 | RSS 源不可用 | 尝试其他 RSS 源 |
| HTML 提取失败 | 内容为空 | CSS 选择器错误 | 检查网站结构变化 |
| Gmail 等待超时 | 工作流挂起 | 用户未响应 | 增加超时时间 |
| 修改未应用 | 内容不变 | LLM 忽略指令 | 强化提示词 |

### 发布到 LinkedIn

使用 n8n LinkedIn 节点:
```json
{
  "content": "{{ $json.linkedin_post }}",
  "visibility": "PUBLIC"
}
```

### 内容质量提升

| 技巧 | 说明 |
|------|------|
| **添加数据** | 在提示词中包含行业数据 |
| **引用来源** | 链接到原始文章 |
| **使用 emoji** | 让帖子更生动 |
| **添加话题标签** | 增加 #标签 |
| **A/B 测试** | 生成多个版本选择 |

### 生产部署注意事项

**定时执行**:
- 每周生成一次帖子
- 最佳发布时间: 周二-周四，上午 10 点

**内容管理**:
- 保存所有生成的帖子
- 记录哪些帖子获得最多互动
- 建立内容日历

### 相关资源

**相关 Episode**:
- [Episode 2](../episode_2/) - 每日摘要
- [Episode 5](../episode_5/) - 博客写作系统
- [Episode 12](../episode_12/) - Postiz 社交媒体调度

**外部资源**:
- [LinkedIn API](https://learn.microsoft.com/en-us/linkedin/shared/references/v2/api)
- [RSS 最佳实践](https://www.rssboard.org/rss-specification)

---

## 资源下载

### n8n 工作流文件

- [Create_a_LinkedIn_post__with_approval_updated.json](./Create_a_LinkedIn_post__with_approval_updated.json)

---

## 观看视频

[![How to create LinkedIn posts using a n8n AI agent](https://img.youtube.com/vi/o_oSYl6gSO8/0.jpg)](https://www.youtube.com/watch?v=o_oSYl6gSO8)

**时长**: ~25 分钟 | **更新日期**: 2025-01-17

---

## 社区支持

- [Skool 社区](https://www.skool.com/ai-agents-az/about?gw3)

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

**Episode**: 3 | **版本**: v2.0 (分层解释版) | **最后更新**: 2025-01-17

**标签**: n8n, LinkedIn, human-in-the-loop, social media, content creation, Gmail
