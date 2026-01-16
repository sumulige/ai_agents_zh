# Episode 1: Prescription Refill AI Agent with n8n

> AI Agents A-Z - 用 n8n 构建实用的 AI Agent
> 难度: ⭐⭐ | 预估时间: 45 分钟

---

## [Level 1] 这一集在做什么？

本集教你用 n8n 构建一个**处方续药 AI Agent**，自动处理患者的药物续药请求。

**一句话**：像"药房助理"一样，通过对话验证患者身份，检查预批准药物，并自动生成续药订单。

**适用场景**：
- 医疗诊所的处方续药自动化
- 患者服务的 24/7 自动化响应
- HIPAA 合规的对话式 AI 应用

> 💡 **快速判断**：如果你需要学习**如何构建医疗保健领域的 AI Agent**，这一集适合你。
> 想了解更多？继续阅读 [Level 2]。

---

## [Level 2] 核心概念

### 你会学到什么

| 序号 | 概念 | 说明 |
|------|------|------|
| 1 | **AI Agent** | 使用 LangChain Agent 构建智能对话系统 |
| 2 | **Chat Trigger** | n8n 的对话触发器，支持实时聊天 |
| 3 | **Window Buffer Memory** | 限制对话历史的上下文窗口 |
| 4 | **Notion 作为数据库** | 使用 Notion 存储患者、药物和订单数据 |
| 5 | **HIPAA 合规** | 医疗数据隐私保护的最佳实践 |

### 涉及的 n8n 节点

| 节点类型 | 用途 | 新手友好度 |
|----------|------|------------|
| Chat Trigger | 接收用户聊天消息 | ⭐ 简单 |
| AI Agent (LangChain) | 智能对话和决策 | ⭐⭐⭐ 高级 |
| OpenAI Chat Model | GPT-4o 语言模型 | ⭐⭐ 中等 |
| Window Buffer Memory | 上下文记忆管理 | ⭐⭐ 中等 |
| Notion Tool | 读写 Notion 数据库 | ⭐⭐ 中等 |

### 涉及的外部服务

| 服务 | 免费额度 | 难度 | 官网 |
|------|----------|------|------|
| **OpenAI** | 按使用付费 | ⭐ | [openai.com](https://openai.com/) |
| **Notion** | 免费套餐足够 | ⭐ | [notion.so](https://notion.so/) |

> 💡 **了解够了？** 知道学什么就可以开始。继续阅读 [Level 3] 了解工作流结构。

---

## [Level 3] 工作流结构

### 工作流概览图

```
┌─────────────────────────────────────────────────────────────┐
│              "Prescription Refill Agent" 工作流               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [Chat Trigger] ──► [AI Agent]                             │
│  用户发送消息         智能对话处理                            │
│                            │                                 │
│                            ├──► [Notion: 获取患者信息]        │
│                            │     验证身份                      │
│                            │                                 │
│                            ├──► [Notion: 获取预批准药物]      │
│                            │     检查权限                      │
│                            │                                 │
│                            ├──► [Notion: 获取药物名称]        │
│                            │     显示选项                      │
│                            │                                 │
│                            └──► [Notion: 创建订单]            │
│                                  记录续药请求                  │
│                                                             │
│  [Window Buffer Memory] ──► [AI Agent]                     │
│  上下文记忆              保持对话连贯性                        │
│                                                             │
│  [OpenAI Chat Model] ──► [AI Agent]                       │
│  GPT-4o                驱动智能决策                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 数据流

```
患者发送消息
    │
    ├──► AI Agent 验证身份
    │     └─── 检查: 姓名 + 出生日期 + 联系方式
    │
    ├──► 查询预批准药物列表
    │     └─── 交叉验证: patient_id ↔ approved_prescriptions
    │
    ├──► 患者选择药物
    │     └─── 确认: medication_name ↔ medications_db
    │
    └──► 创建续药订单
          └─── 存入: prescription_orders 数据库
```

### 节点说明

| 节点 | 类型 | 配置要点 | 数据输出 |
|------|------|----------|----------|
| **When chat message received** | Chat Trigger | 生产 URL 用于用户访问 | 聊天消息 |
| **AI Agent** | LangChain Agent | HIPAA 合规的系统提示 | 对话响应 |
| **OpenAI Chat Model** | LLM | 使用 GPT-4o | AI 推理 |
| **Window Buffer Memory** | Memory | 15 轮对话历史 | 上下文 |
| **notion_get_all_patients** | Notion Tool | 患者 Database | 患者列表 |
| **notion_get_all_approved_prescriptions** | Notion Tool | 预批准 Database | 药物列表 |
| **notion_get_all_medication_names** | Notion Tool | 药物 Database | 药物名称 |
| **notion_add_order** | Notion Tool | 订单 Database | 新订单 |

### AI Agent 系统提示

该 Agent 使用详细的系统提示，包含：
- **角色定义**: GP Assistant，负责处方续药和 HIPAA 合规
- **工具协议**: 4 个 Notion 数据库的使用规则
- **验证流程**: 患者身份、药物权限、订单确认
- **通信要求**: 通俗语言、确认步骤、隐私保护、同理心
- **错误状态**: 系统不可达、数据冲突、患者情绪处理
- **禁止操作**: 不能修改批准状态、不能访问非处方记录

> 💡 **准备就绪？** 理解工作流结构后，继续阅读 [Level 4] 开始构建。

---

## [Level 4] 构建步骤

### 前置准备

在开始之前，请确保：

- [ ] n8n 已安装并运行（访问 http://localhost:5678）
- [ ] OpenAI API Key 已配置
- [ ] Notion 账号已创建
- [ ] Notion 集成已配置（获取 API Key）

### 步骤 1: 在 Notion 中创建数据库

**目标**: 创建 4 个 Notion 数据库存储数据

**操作**:

1. 创建 **Patients** 数据库，字段：
   - `patient_id` (标题，唯一)
   - `full_name` (文本)
   - `dob` (日期)
   - `email` (邮箱)
   - `phone` (电话)

2. 创建 **Pre-approved prescriptions** 数据库，字段：
   - `patient_id` (关联到 Patients)
   - `medication_id` (文本)
   - `approval_status` (选择)

3. 创建 **Medications** 数据库，字段：
   - `medication_id` (标题，唯一)
   - `name` (文本)

4. 创建 **Prescription orders** 数据库，字段：
   - `patient_id` (关联)
   - `medication` (文本)
   - `Order date` (日期)

**验证**: 所有数据库已创建并添加示例数据

---

### 步骤 2: 导入 n8n 工作流

**目标**: 导入处方续药 Agent 工作流

**操作**:

1. 在 n8n 中点击右上角 **"..."** 菜单
2. 选择 **"Import from File"**
3. 选择 `Prescription_refill_agent.json`

**验证**: 工作流显示在画布上，包含 Chat Trigger 和 AI Agent

---

### 步骤 3: 配置 Notion 凭证

**目标**: 连接 n8n 到你的 Notion 工作区

**操作**:

1. 点击任意 Notion 节点
2. 点击 **"Credential to connect"**
3. 选择 **"Create New"**
4. 输入 Notion Internal Integration Token
5. 保存凭证

**获取 Notion Token**:
1. 访问 [notion.so/my-integrations](https://www.notion.so/my-integrations)
2. 创建新集成
3. 复制 Internal Integration Token
4. 在 Notion 中为你的数据库添加该集成

**验证**: Notion 节点显示已连接

---

### 步骤 4: 配置 OpenAI Chat Model

**目标**: 连接 GPT-4o 模型

**操作**:

1. 点击 **"OpenAI Chat Model"** 节点
2. 在凭据中选择或创建 OpenAI 凭证
3. 确保 Model 设置为 `gpt-4o`

**验证**: 模型已选择，凭证有效

---

### 步骤 5: 更新 Notion Database ID

**目标**: 连接到你的 Notion 数据库

**操作**:

1. 依次点击每个 Notion Tool 节点
2. 在 Database 字段中选择对应的数据库
3. 记录每个数据库的 ID

**验证**: 每个节点都已选择正确的数据库

---

### 步骤 6: 激活并测试工作流

**目标**: 端到端验证续药流程

**操作**:

1. 激活工作流（左上角开关）
2. 复制 Chat Trigger 的生产 URL
3. 在浏览器中打开 URL
4. 开始测试对话：

```
用户: Hi, I need to refill my prescription
Agent: I can help with that. Please provide your full name and date of birth.
用户: John Doe, 1990-05-15
Agent: Thank you. Can you also provide your registered email or phone?
用户: john@example.com
Agent: I found your record. You have the following pre-approved medications...
```

**预期结果**:
- Agent 正确验证患者身份
- 显示预批准的药物列表
- 成功创建续药订单到 Notion

> 💡 **需要帮助？** 如果遇到问题，查看 [Level 5] 故障排除。

---

## [Level 5] 进阶内容

### HIPAA 合规注意事项

| 方面 | 要求 | 实现 |
|------|------|------|
| **数据隐私** | 不在对话中透露完整信息 | 只显示部分信息（如 "...6789"） |
| **访问控制** | 限制数据访问权限 | AI Agent 不能访问非处方记录 |
| **审计日志** | 记录所有操作 | Notion 自动记录所有变更 |
| **错误处理** | 不泄露系统错误 | 使用通用错误消息 |

### 扩展工作流

**添加电子邮件通知**:
```
[AI Agent] ──► [Email] ──► 患者确认邮件
               │
               └───► 药店通知
```

**添加 SMS 通知**:
```
[AI Agent] ──► [Twilio] ──► SMS 确认
```

**添加日程安排**:
```
[AI Agent] ──► [Google Calendar] ──► 预约提醒
```

### 数据库设计最佳实践

| 数据库 | 关键字段 | 索引建议 |
|--------|----------|----------|
| Patients | patient_id (唯一) | 唯一索引 |
| Approved prescriptions | patient_id + medication_id | 复合索引 |
| Medications | medication_id (唯一) | 唯一索引 |
| Orders | patient_id + order_date | 复合索引 |

### 系统提示优化

**添加新规则**:
```markdown
**新规则**:
- 只在工作时间 9am-5pm 处理请求
- 紧急情况引导患者致电 +12212222
```

### 故障排除

| 问题 | 症状 | 可能原因 | 解决方案 |
|------|------|----------|----------|
| Notion 连接失败 | "Unauthorized" 错误 | Token 无效或未添加集成 | 重新生成 Token，确保集成已添加 |
| 找不到患者 | AI Agent 返回未找到记录 | 患者数据不存在 | 检查数据库是否添加了测试数据 |
| Agent 记忆问题 | Agent 忘记之前的对话 | Memory 配置问题 | 增加 Window Buffer Memory 大小 |
| 订单未创建 | Agent 说已创建但 Notion 没有 | Database ID 错误 | 检查 notion_add_order 节点的配置 |

### 生产部署注意事项

**安全建议**:
- 使用环境变量存储 API Keys
- 限制 Chat Trigger URL 的访问
- 定期轮换 Notion Integration Token
- 启用 n8n 的访问日志

**性能优化**:
- 考虑使用更快的模型（如 GPT-4o-mini）
- 设置合理的超时时间
- 使用缓存减少重复查询

**监控指标**:
- 平均响应时间
- 订单成功率
- 患者验证失败率
- Agent 对话轮次

### 相关资源

**相关 Episode**:
- [Episode 2](../episode_2/) - 自动化每日摘要
- [Episode 3](../episode_3/) - LinkedIn 帖子人工审批
- [Episode 4](../episode_4/) - 深度研究子工作流

**外部资源**:
- [Notion API 文档](https://developers.notion.com/)
- [OpenAI API 文档](https://platform.openai.com/docs)
- [HIPAA 合规指南](https://www.hhs.gov/hipaa/index.html)

---

## 资源下载

### n8n 工作流文件

下载并导入到 n8n：

- [Prescription_refill_agent.json](./Prescription_refill_agent.json)

**导入方法**:
1. 在 n8n 中点击右上角 "..." 菜单
2. 选择 "Import from File"
3. 选择下载的 JSON 文件

---

## 观看视频

[![Build an AI agent for prescription refills (free n8n template)](https://img.youtube.com/vi/rcsZSB3Ns1c/0.jpg)](https://www.youtube.com/watch?v=rcsZSB3Ns1c)

**时长**: ~25 分钟 | **更新日期**: 2025-01-17

---

## 社区支持

遇到问题？加入社区获取帮助：

- [Skool 社区](https://www.skool.com/ai-agents-az/about?gw1)
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

**Episode**: 1 | **版本**: v2.0 (分层解释版) | **最后更新**: 2025-01-17

**标签**: n8n, AI Agent, prescription, healthcare, Notion, HIPAA, LangChain
