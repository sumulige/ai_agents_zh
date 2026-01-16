# Episode 12: Social Media Scheduling with Postiz and n8n

> AI Agents A-Z - 用 n8n 构建实用的 AI Agent
> 难度: ⭐⭐ | 预估时间: 35 分钟

---

## [Level 1] 这一集在做什么？

本集教你用 n8n 集成 **Postiz**（开源社交媒体调度工具），实现多平台内容自动化发布。

**一句话**：像"社交媒体经理"一样，统一调度 TikTok、YouTube、Instagram 等多个平台的内容发布。

**适用场景**：
- 多平台社交媒体运营
- 免费 Buffer/Hootsuite 替代
- 内容自动化分发

> 💡 **快速判断**：如果你需要**免费的开源社交媒体调度方案**，这一集适合你。
> 想了解更多？继续阅读 [Level 2]。

---

## [Level 2] 核心概念

### 你会学到什么

| 序号 | 概念 | 说明 |
|------|------|------|
| 1 | **Postiz 平台** | 开源社交媒体调度工具 |
| 2 | **Postiz API** | RESTful API 接口 |
| 3 | **平台路由** | 根据平台选择不同处理 |
| 4 | **内容适配** | 为不同平台调整内容 |
| 5 | **自托管部署** | 完全免费的私有部署 |

### 涉及的 n8n 节点

| 节点类型 | 用途 | 新手友好度 |
|----------|------|------------|
| Manual Trigger | 手动启动 | ⭐ 简单 |
| Set | 设置变量 | ⭐ 简单 |
| Switch | 条件路由 | ⭐⭐ 中等 |
| HTTP Request | API 调用 | ⭐⭐ 中等 |

### 涉及的外部服务

| 服务 | 免费额度 | 难度 | 官网 |
|------|----------|------|------|
| **Postiz** | 100% 免费（自托管） | ⭐⭐ | [postiz.com](https://postiz.com/) |
| **TikTok** | 免费 | ⭐⭐ | [tiktok.com](https://www.tiktok.com/) |
| **YouTube** | 免费 | ⭐⭐ | [youtube.com](https://youtube.com/) |
| **Instagram** | 免费 | ⭐⭐ | [instagram.com](https://instagram.com/) |

### 支持的平台

Postiz 支持发布到：
- TikTok
- YouTube Shorts
- Instagram (Reels/Posts)
- LinkedIn
- Twitter/X
- Facebook
- Pinterest
- Telegram
- Mastodon

> 💡 **了解够了？** 知道学什么就可以开始。继续阅读 [Level 3] 了解工作流结构。

---

## [Level 3] 工作流结构

### 工作流概览图

```
┌─────────────────────────────────────────────────────────────┐
│              "Postiz Social Media Scheduler" 工作流           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [Manual Trigger]                                           │
│       │                                                     │
│       ▼                                                     │
│  [Configure Me]                                             │
│  设置 Postiz API URL 和内容                                 │
│       │                                                     │
│       ▼                                                     │
│  [Switch] ──► 根据平台路由                                  │
│       │                                                     │
│       ├───► tiktok ──► [TikTok 处理]                       │
│       │                                                     │
│       ├───► youtube ──► [YouTube 处理]                     │
│       │                                                     │
│       └───► instagram ──► [Instagram 处理]                  │
│                        │                                    │
│                        ▼                                    │
│                  [HTTP Request]                             │
│                  调用 Postiz API                            │
│                        │                                    │
│                        ▼                                    │
│                  [发送到平台]                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### API 流程

```
创建帖子
    │
    ├──► Postiz API
    │     └─── POST /api/public/v1/post
    │
    ├──► 平台处理
    │     ├─── TikTok: 短视频
    │     ├─── YouTube: Shorts/视频
    │     └─── Instagram: Reels/图片
    │
    └─── 发布结果
          └─── 返回 post_id
```

### Postiz API 端点

| 端点 | 方法 | 说明 |
|------|------|------|
| `/api/public/v1/post` | POST | 创建并发布帖子 |
| `/api/public/v1/posts` | GET | 获取帖子列表 |
| `/api/public/v1/media` | POST | 上传媒体文件 |

> 💡 **准备就绪？** 理解工作流结构后，继续阅读 [Level 4] 开始构建。

---

## [Level 4] 构建步骤

### 前置准备

在开始之前，请确保：

- [ ] n8n 已安装并运行
- [ ] Postiz 已部署（云或自托管）
- [ ] 社交媒体账号已连接到 Postiz

### 步骤 1: 部署 Postiz

**目标**: 设置 Postiz 服务

**选项 A - 云端版**（免费）:
1. 访问 [app.postiz.com](https://app.postiz.com/)
2. 注册账号
3. 连接社交媒体账号

**选项 B - 自托管**（完全免费）:
```bash
# 使用 Docker
docker run -d \
  --name postiz \
  -p 5000:3000 \
  -e DATABASE_URL=your_db_url \
  gitlab.com/postizlabs/postiz:latest
```

**验证**: Postiz 可访问

---

### 步骤 2: 导入工作流

**目标**: 导入 Postiz 工作流

**操作**:

1. 在 n8n 中点击 **"..."** 菜单
2. 选择 **"Import from File"**
3. 选择 `workflow_postiz.json`

**验证**: 工作流显示在画布上

---

### 步骤 3: 配置 Postiz API

**目标**: 连接到 Postiz

**操作**:

1. 找到 **"Configure me"** 节点
2. 设置 `postiz_api`:
   - 云端版: `https://app.postiz.com/api/public/v1`
   - 自托管: `http://your-server:port/api/public/v1`

**验证**: API URL 正确

---

### 步骤 4: 设置发布内容

**目标**: 准备要发布的内容

**操作**:

1. 在 **"Configure me"** 节点中设置：
   - `share_video_url`: 视频文件 URL
   - `share_title`: 帖子标题
   - `share_description`: 描述文本

**验证**: 内容已设置

---

### 步骤 5: 配置平台路由

**目标**: 设置平台特定处理

**操作**:

1. 查看 **"Switch"** 节点
2. 确认路由规则：
   - tiktok
   - youtube
   - instagram

**验证**: 路由配置正确

---

### 步骤 6: 测试发布

**目标**: 发布第一条帖子

**操作**:

1. 手动执行工作流
2. 选择目标平台分支
3. 等待 API 响应
4. 检查平台发布状态

**预期结果**:
- API 调用成功
- 帖子已发布
- 返回 post_id

> 💡 **需要帮助？** 如果遇到问题，查看 [Level 5] 故障排除。

---

## [Level 5] 进阶内容

### 内容适配策略

**平台差异化**:
```javascript
// TikTok - 短平快
{
  "video": "<60s",
  "hashtag": "#fyp #viral",
  "description": "简短有力"
}

// YouTube Shorts - 描述丰富
{
  "video": "<60s",
  "title": "吸引人的标题",
  "description": "详细描述 + 标签"
}

// Instagram - 视觉优先
{
  "image/Reel": "高质量视觉",
  "hashtag": "#instagood",
  "first_comment": "更多标签"
}
```

### 批量发布

**定时发布**:
```
[Schedule] ──► [获取内容队列] ──► [Switch 平台] ──► [发布]
     │                │                         │
     └──► 最佳时间 ────┴─────────────────────────┘
```

### 内容队列管理

**使用数据库**:
```sql
CREATE TABLE content_queue (
  id UUID PRIMARY KEY,
  content TEXT,
  platforms TEXT[],
  scheduled_at TIMESTAMP,
  status TEXT
);
```

### 自动化内容来源

**与其他工作流集成**:
```
[Episode 7: 视频生成] ──► [Postiz 发布]
[Episode 8: Instagram] ──► [Postiz 发布]
[Episode 11: 励志视频] ──► [Postiz 发布]
```

### Postiz 功能

**核心功能**:
- 多平台发布
- AI 内容生成
- 媒体库管理
- 分析和洞察
- 团队协作
- 日程安排

### 故障排除

| 问题 | 症状 | 可能原因 | 解决方案 |
|------|------|----------|----------|
| API 连接失败 | 无法连接 | URL 错误 | 检查 API 地址 |
| 发布失败 | 错误响应 | 平台权限 | 重新授权 |
| 视频上传失败 | 超时 | 文件太大 | 压缩视频 |
| 账号未连接 | 认证错误 | OAuth 失效 | 重新连接账号 |

### 成本对比

| 工具 | 月费 | Postiz |
|------|------|--------|
| Buffer | $15+ | 免费 |
| Hootsuite | $99+ | 免费 |
| Later | $25+ | 免费 |

**Postiz 自托管**:
- 100% 免费
- 无限连接
- 无限发布

### 扩展功能

**AI 生成内容**:
```
[主题] ──► [OpenAI] ──► [生成文案] ──► [Postiz 发布]
```

**A/B 测试**:
```
[同一内容] ──► [不同标题] ──► [对比效果]
```

**自动回复**:
```
[评论监控] ──► [AI 回复] ──► [发布评论]
```

### 相关资源

**相关 Episode**:
- [Episode 7](../episode_7/) - YouTube Shorts
- [Episode 8](../episode_8/) - Instagram
- [Episode 11](../episode_11/) - 励志短视频

**外部资源**:
- [Postiz 官网](https://postiz.com/)
- [Postiz GitHub](https://github.com/gitroomhq/postiz)
- [Postiz 文档](httpsdocs.postiz.com/)

---

## 资源下载

### n8n 工作流文件

- [workflow_postiz.json](./workflow_postiz.json)

### Postiz 部署

- [Docker Hub](https://hub.docker.com/r/gitroomhq/postiz)
- [自托管指南](https://postiz.com/docs/self-hosting)

---

## 观看视频

[![RIP Blotato & Buffer! FREE social media post scheduling API - n8n automation (TikTok, YouTube, IG)](https://img.youtube.com/vi/Q5a9q-PzNz4/0.jpg)](https://www.youtube.com/watch?v=Q5a9q-PzNz4)

**时长**: ~25 分钟 | **更新日期**: 2025-01-17

---

## 社区支持

- [Skool 社区](https://www.skool.com/ai-agents-az/about?gw12)

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

**Episode**: 12 | **版本**: v2.0 (分层解释版) | **最后更新**: 2025-01-17

**标签**: n8n, Postiz, social media, scheduling, TikTok, YouTube, Instagram, automation
