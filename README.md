# AI 资讯推送机器人

🤖 基于 WorkBuddy Skill 体系的每日 AI 简报自动化生成与推送工具

## ✨ 功能介绍

### 核心功能

- **新闻自动采集**：通过 AIHOT API 自动采集过去 24 小时精选 AI 新闻
- **智能内容筛选**：按 P1/P2/P3/P4 优先级筛选，教育行业视角过滤
- **杂志风排版**：生成极简杂志风 HTML 简报，支持响应式设计
- **企业微信推送**：自动部署到 CloudStudio 获取公网 URL，通过模板卡片推送至企业微信群
- **摘要生成**：自动提取每条新闻 ≤20 字摘要，适配手机端阅读

### 栏目结构

| 栏目 | 说明 |
|------|------|
| 🔥 热点头条 | 当日最具影响力的 AI 大事件 |
| 🎓 AI+教育 | AI 在教育场景的落地应用（教学、学习、升学等） |
| 🤖 模型发布 | 主流大模型新版本发布 |
| 🛠️ 产品动态 | AI 产品新功能、新工具发布 |
| 💡 技巧与观点 | AI 使用技巧、行业观点 |
| 📋 行业速览 | 精选行业动态一句话速览 |

### 技术特点

- **教育行业视角**：专为教育从业者筛选，优先关注 AI 在教育场景的应用
- **严格字数控制**：单条摘要 ≤20 字，正文控制在 80-150 字
- **时效性保障**：仅收录前一天 00:00 至生成时刻的新闻
- **来源可追溯**：每条新闻标注来源与发布时间

## 🚀 使用方法

### 前置准备

1. **安装 Agent 桌面端工具
   - WorkBuddy（推荐）
   - Trae
   - 或其他支持 Skill 体系的 Agent 桌面端

2. **安装 Skill**
   - 将本项目 `skills/` 目录下的 4 个 Skill 文件夹复制到你的 Agent 工具的 Skill 目录
   - 或通过 Skill 管理界面导入这 4 个 Skill

### 快速开始

在 Agent 对话框中输入以下提示词即可触发完整流水线：

```
生成今日AI简报，按以下流水线执行：
1. @AIHOT 采集过去24小时精选AI新闻
2. @edu-ai-briefing-principles 筛选内容：P1/P2优先，P3进速览，P4丢弃；按五个栏目分配，控制摘要≤20字/条
3. @ai-briefing-template 套用杂志风模板生成HTML，文件名 ai_briefing_YYYYMMDD.html
4. @wecom-group-send 推送到企业微信群，部署deploy/到CloudStudio获取公网URL
完成后输出简报摘要。
```

### 流水线说明

| 阶段 | Skill | 职责 |
|------|-------|------|
| 1 | @AIHOT | 采集过去 24 小时精选 AI 新闻 |
| 2 | @edu-ai-briefing-principles | 优先级筛选、栏目分配、摘要提取、内容撰写 |
| 3 | @ai-briefing-template | 套用杂志风 CSS 模板，生成最终 HTML 文件 |
| 4 | @wecom-group-send | 部署到 CloudStudio，通过企业微信群机器人推送 |

## 📁 目录结构

```
AI资讯推送机器人/
├── skills/
│   ├── aihot/                    # AI 新闻采集 Skill
│   ├── edu-ai-briefing-principles/  # 内容筛选与撰写原则 Skill
│   ├── ai-briefing-template/     # HTML 模板 Skill
│   │   ├── SKILL.md
│   │   └── assets/
│   │       └── template.css     # 杂志风 CSS 样式
│   └── wecom-group-send/       # 企业微信推送 Skill
│       ├── SKILL.md
│       └── assets/
│           └── cover_banner.png  # 简报封面图
└── README.md
```

## ⚙️ 配置说明

### 企业微信机器人配置

在使用 `@wecom-group-send` 前，请先配置你的企业微信群机器人：

1. 在企业微信群中创建群机器人，获取 Webhook URL
2. 在 `wecom-group-send/SKILL.md` 中配置你的 Webhook 地址
3. 或在使用时通过参数传入 Webhook URL

### 封面图自定义

替换 `wecom-group-send/assets/cover_banner.png` 为你的自定义封面图，推荐尺寸 1024×441。

## 📝 输出示例

### 推送卡片样式：

```
📰 今日AI简报 | 2026年08月10日

[封面图]

— GPT-5.6 发布，ChatGPT Work 上线
— 本地数学 Agent TeXada 发布
— Claude 推出反思功能

[共X条精选，查看完整简报]
```

### HTML 简报效果：

- 极简杂志风排版
- 响应式设计，适配手机/桌面
- 每条新闻包含来源、时间、链接
- 摘要列表快速预览

## 🔧 Skill 说明

### AIHOT
- 基于 aihot.virxact.com API
- 实时更新，mode=selected 精选模式
- 自动过滤 24 小时时间窗

### edu-ai-briefing-principles
- 教育行业视角内容筛选标准
- P1/P2/P3/P4 四级优先级
- 五大栏目自动分配
- 摘要字数严格控制

### ai-briefing-template
- 极简杂志风 CSS 样式
- 响应式布局
- 自动编号与栏目配色
- 企业微信内置浏览器兼容

### wecom-group-send
- template_card 消息类型
- CloudStudio 自动部署
- 公网 URL 自动获取
- 发送状态校验

## 📄 许可证

MIT License

## 🤝 贡献

欢迎提交 Issue 和 PR！

---

**注意**：本工具仅供内部使用时请遵守相关 API 使用条款，合理控制调用频率，避免对服务造成压力。