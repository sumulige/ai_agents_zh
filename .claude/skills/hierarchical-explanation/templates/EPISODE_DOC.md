# Episode {N}: {TITLE}

> AI Agents A-Z - 用 n8n 构建实用的 AI Agent
> 难度: {DIFFICULTY} | 预估时间: {ESTIMATED_TIME}

---

## [Level 1] 这一集在做什么？

本集教你用 n8n 构建 {WHAT_IT_DOES}。

**一句话**：{ONE_LINER_ANALOGY}

**适用场景**：{USE_CASE_ONE_LINER}

> 💡 **快速判断**：如果你 {CONDITION}，这一集适合你。
> 想了解更多？继续阅读 [Level 2]。

---

## [Level 2] 核心概念

### 你会学到什么

| 序号 | 概念 | 说明 |
|------|------|------|
| 1 | **{CONCEPT_1}** | {WHY_IMPORTANT_1} |
| 2 | **{CONCEPT_2}** | {WHY_IMPORTANT_2} |
| 3 | **{CONCEPT_3}** | {WHY_IMPORTANT_3} |

### 涉及的 n8n 节点

| 节点类型 | 用途 | 新手友好度 |
|----------|------|------------|
| {NODE_1} | {PURPOSE_1} | ⭐/⭐⭐/⭐⭐⭐ |
| {NODE_2} | {PURPOSE_2} | ⭐/⭐⭐/⭐⭐⭐ |
| {NODE_3} | {PURPOSE_3} | ⭐/⭐⭐/⭐⭐⭐ |

### 涉及的外部服务

| 服务 | 免费额度 | 难度 | 官网 |
|------|----------|------|------|
| {SERVICE_1} | {TIER_1} | ⭐/⭐⭐ | {LINK_1} |
| {SERVICE_2} | {TIER_2} | ⭐/⭐⭐ | {LINK_2} |

> 💡 **了解够了？** 知道学什么就可以开始。继续阅读 [Level 3] 了解工作流结构。

---

## [Level 3] 工作流结构

### 工作流概览图

```
┌─────────────────────────────────────────────────────────────┐
│                    "{WORKFLOW_NAME}" 工作流                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [{TRIGGER}] ──► [{PROCESS_1}] ──► [{PROCESS_2}]           │
│      │               │                   │                  │
│      ▼               ▼                   ▼                  │
│  {TRIGGER_DESC}  {PROCESS_DESC}    {OUTPUT_DESC}           │
│                     │                                       │
│                     └──────────────► [{STORAGE}]            │
│                                           │                 │
│                                           ▼                 │
│                                    {STORAGE_DESC}           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 节点说明

| 节点 | 类型 | 配置要点 | 数据输出 |
|------|------|----------|----------|
| {NODE_1} | {TYPE_1} | {KEY_CONFIG_1} | {OUTPUT_1} |
| {NODE_2} | {TYPE_2} | {KEY_CONFIG_2} | {OUTPUT_2} |
| {NODE_3} | {TYPE_3} | {KEY_CONFIG_3} | {OUTPUT_3} |
| {NODE_4} | {TYPE_4} | {KEY_CONFIG_4} | {OUTPUT_4} |

### 数据流

```
{INPUT_DATA}
  │
  ├──► {TRANSFORMATION_1}
  │     └──► {INTERMEDIATE_DATA_1}
  │
  ├──► {TRANSFORMATION_2}
  │     └──► {INTERMEDIATE_DATA_2}
  │
  └──► {OUTPUT_DATA}
```

### 关键配置点

| 配置项 | 值 | 说明 |
|--------|-----|------|
| {CONFIG_1} | {VALUE_1} | {NOTE_1} |
| {CONFIG_2} | {VALUE_2} | {NOTE_2} |

> 💡 **准备就绪？** 理解工作流结构后，继续阅读 [Level 4] 开始构建。

---

## [Level 4] 构建步骤

### 前置准备

在开始之前，请确保：

- [ ] n8n 已安装并运行（访问 http://localhost:5678）
- [ ] {ACCOUNT_1} 账号已创建
- [ ] {API_KEY_1} 已获取并保存
- [ ] {DEPENDENCY} 已满足

### 步骤 1: {STEP_1_TITLE}

**目标**: {OBJECTIVE_1}

**操作**:

1. 在 n8n 中创建新工作流
2. 点击左侧节点面板，搜索 `{NODE_NAME}`
3. 将节点拖入画布
4. 配置参数：

```
参数1: {VALUE_1}
参数2: {VALUE_2}
参数3: {VALUE_3}
```

**验证**: 点击 "Test step" 检查节点是否正常工作

**预期结果**: {EXPECTED_OUTCOME_1}

---

### 步骤 2: {STEP_2_TITLE}

**目标**: {OBJECTIVE_2}

**操作**:

1. 添加 `{NODE_NAME}` 节点
2. 连接到上一步的输出
3. 配置 {KEY_SETTING}:

```json
{
  "key": "value",
  "setting": "config"
}
```

**验证**: {VERIFICATION_METHOD}

**预期结果**: {EXPECTED_OUTCOME_2}

---

### 步骤 3: {STEP_3_TITLE}

**目标**: {OBJECTIVE_3}

**操作**:

{DETAILED_INSTRUCTIONS}

**验证**: {VERIFICATION_METHOD}

**预期结果**: {EXPECTED_OUTCOME_3}

---

### 步骤 4: 测试完整工作流

**目标**: 端到端验证

**操作**:

1. 点击工作流左上角的 "Execute Workflow"
2. 观察 {OBSERVATION_POINT}
3. 检查 {CHECK_POINT}

**预期结果**: {FINAL_OUTCOME}

> 💡 **需要帮助？** 如果遇到问题，查看 [Level 5] 故障排除。

---

## [Level 5] 进阶内容

### 自定义扩展

#### 扩展 1: {EXTENSION_1}
{HOW_TO_CUSTOMIZE_1}

#### 扩展 2: {EXTENSION_2}
{HOW_TO_CUSTOMIZE_2}

### 故障排除

| 问题 | 症状 | 可能原因 | 解决方案 |
|------|------|----------|----------|
| {PROBLEM_1} | {SYMPTOM_1} | {CAUSE_1} | {SOLUTION_1} |
| {PROBLEM_2} | {SYMPTOM_2} | {CAUSE_2} | {SOLUTION_2} |
| {PROBLEM_3} | {SYMPTOM_3} | {CAUSE_3} | {SOLUTION_3} |

### 生产部署注意事项

**环境变量管理**:
- 敏感信息（API Key）使用环境变量
- 在 n8n 设置中配置 `.env` 文件

**错误处理**:
- 添加 Error Trigger 节点
- 配置失败通知（邮件/Slack）

**性能优化**:
- {PERFORMANCE_TIP_1}
- {PERFORMANCE_TIP_2}

**监控与日志**:
- 启用执行日志
- 设置定期健康检查

### 相关资源

**相关 Episode**:
- [Episode {N}]({LINK}) - {RELATIONSHIP}
- [Episode {N}]({LINK}) - {RELATIONSHIP}

**外部资源**:
- [{RESOURCE_1}]({LINK_1})
- [{RESOURCE_2}]({LINK_2})

---

## 资源下载

### n8n 工作流文件

下载并导入到 n8n：

- [{WORKFLOW_FILE_NAME}]({WORKFLOW_FILE_PATH})

**导入方法**:
1. 在 n8n 中点击右上角 "..." 菜单
2. 选择 "Import from File"
3. 选择下载的 JSON 文件

### 相关指南

- [{GUIDE_1}]({GUIDE_1_PATH})
- [{GUIDE_2}]({GUIDE_2_PATH})

---

## 观看视频

[![{VIDEO_THUMBNAIL_TEXT}]({VIDEO_THUMBNAIL_URL})]({VIDEO_URL})

**时长**: {VIDEO_DURATION} | **更新日期**: {DATE}

---

## 社区支持

遇到问题？加入社区获取帮助：

- [Skool 社区](https://www.skool.com/ai-agents-az/about?gw{EPISODE_NUMBER})
- [Discord 频道](#)
- [GitHub Issues](#)

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

**版本**: v1.0 | **最后更新**: {DATE} | **作者**: {AUTHOR}

**标签**: {TAGS}
