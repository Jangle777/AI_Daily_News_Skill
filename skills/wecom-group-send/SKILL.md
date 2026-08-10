---
name: wecom-group-send
description: 通过企业微信群机器人 Webhook 向指定群聊发送消息。当用户需要向企业微信群推送消息、群通知、自动发送内容时触发。支持文本、Markdown、图文卡片（news）和模板卡片（template_card）四种消息格式。
agent_created: true
---

# 企业微信群发送 (wecom-group-send)

## 执行原则

本技能为**终点技能**（Terminal Skill）。curl 返回 `errcode:0` 即表示整个任务完成，执行者**必须立即停止所有后续操作**，不得重新拉取数据、重新生成内容、重新部署、追问用户、进入新一轮循环。唯一允许的收尾动作：清理临时文件 + 写 memory。完成即止。

## 避坑清单（必读）

| # | 坑 | 错误做法 | 正确做法 |
|---|-----|---------|---------|
| 1 | curl 崩溃 | `--data-binary @文件路径` → ACCESS_VIOLATION | `cat 文件 \| curl -d @-` 管道方式 |
| 2 | rm 崩溃 | `rm C:/xxx` → ACCESS_VIOLATION | `rm /c/xxx` |
| 3 | 中文乱码 | curl 命令行直接传中文参数 | Write 工具写临时文件 + `cat \| curl -d @-` |
| 4 | 封面图缺失/错路径 | 使用绝对路径硬编码 | 将封面图放在本技能 `assets/` 目录下，使用相对路径引用 |
| 5 | 封面图缓存 | 部署同名 `cover_banner.png`，沙箱不更新 | 每次用唯一文件名（如 `cover_v{序号}.png`） |
| 6 | 每日简报样式错 | 用 `news` 图文卡片 | **每日简报必须用 `template_card` + `news_notice`** |
| 7 | CloudStudio 沙箱过期 | 部署后不验证 → 卡片已推送但链接 `ECONNREFUSED`，用户打不开才发现 | **部署后必须主动验证，不依赖用户反馈**。见下方「部署验证」章节：curl 检测 HTML 页面是否返回 HTTP 200，失败则自动重新部署直到可用，再构造卡片 JSON。 |

## 消息类型选择

| 类型 | 适用场景 | 说明 |
|------|---------|------|
| `text` | 普通通知 | 最长 2048 字节 |
| `markdown` | 富文本通知 | 最长 4096 字节 |
| `news` | 通用图文卡片 | 最多 8 篇，description ≤ 512 字节；**不推荐用于每日简报** |
| `template_card` | **每日简报专用** | 必须用 `card_type: news_notice`，带封面图 + 列表 + 跳转按钮 |

## 工作流程

### Step 1: 确认参数

- **消息内容**（必填）
- **Webhook URL**（必填）：请配置你的企业微信群机器人 Webhook 地址
- **消息类型**（可选）：默认 `text`，支持 `text` / `markdown` / `news` / `template_card`
- 频率限制：每个机器人每分钟最多 20 条

### Step 2: 构造 JSON 并写入临时文件

#### 文本消息（text）
```json
{"msgtype":"text","text":{"content":"消息内容"}}
```
最长 2048 字节。

#### Markdown 消息（markdown）
```json
{"msgtype":"markdown","markdown":{"content":"## 标题\n内容"}}
```
最长 4096 字节。

#### 图文卡片（news）—— 非简报场景使用

```json
{"msgtype":"news","news":{"articles":[{"title":"标题","description":"描述","url":"链接"}]}}
```

最多 8 篇，description ≤ 512 字节。注意：`news` 类型**没有封面图列表和跳转按钮**，不适合每日简报，每日简报请使用下方 `template_card`。

#### 模板卡片（template_card）—— 每日简报专用（强制使用）

```json
{"msgtype":"template_card","template_card":{"card_type":"news_notice","main_title":{"title":"今日AI简报 | 2026年06月10日"},"card_image":{"url":"https://公网URL/cover_vN.png","aspect_ratio":2.25},"horizontal_content_list":[{"keyname":"•","value":"第一条摘要"},{"keyname":"•","value":"第二条摘要"}],"card_action":{"type":1,"url":"https://完整简报链接"},"jump_list":[{"type":1,"title":"共XX条精选，查看完整简报","url":"https://完整简报链接"}]}}
```

**字段约束**：

| 字段 | 规则 |
|------|------|
| `msgtype` | 固定为 `template_card` |
| `template_card.card_type` | 固定为 `news_notice` |
| `main_title.title` | `今日AI简报 \| YYYY年MM月DD日` |
| `main_title.desc` / `source` | **不填** |
| `card_image.url` | **必填**（缺失 → 42044），必须是公网 URL |
| `card_image.aspect_ratio` | **2.25** |
| `horizontal_content_list` | 最多 6 条，keyname=`"•"`，内容放 value |
| `jump_list[0].title` | `共XX条精选，查看完整简报` |

#### 封面图部署

1. 复制封面图（位于本技能 `assets/cover_banner.png`）到当前工作区 `deploy/`，**必须重命名**为带序号的唯一文件名（如 `cover_v7.png`），防止 CloudStudio 复用缓存。

2. 通过 CloudStudio 部署 `deploy/` 目录，获取公网 URL 填入 `card_image.url`。

3. 部署后验证公网图片尺寸为 1024×441，确保不是缓存的旧图。

#### 部署验证（防过期，必执行）

**CloudStudio 沙箱有时效限制，部署成功后公网 URL 可能在极短时间内就已失效。发送卡片前必须主动验证，不得跳过。**

```bash
# 用 curl 直接访问部署后的 HTML 页面，检查是否返回 HTTP 200
# 注意：这里的 URL 是部署返回的 shareLink + 文件名
curl -s -o /dev/null -w "%{http_code}" "https://{sandbox-host}/ai_briefing_YYYYMMDD.html"
```

**判断逻辑**：
- 返回 `200` → 沙箱正常，继续构造卡片 JSON → Step 3
- 返回 `000` / `ECONNREFUSED` / 超时 → **沙箱已过期**，立即重新部署 `deploy/` 目录，部署后用新的 shareLink 再次验证，直到返回 200
- 最多重试 2 次重新部署，若仍失败则报告中止

**注意**：验证的是 HTML 页面本身（用户最终打开的链接），不仅仅是首页根路径。

### Step 3: 通过 curl 发送

```bash
cat {当前工作区绝对路径}/.tmp_wecom_msg.json | curl -s -X POST '<webhook_url>' -H 'Content-Type: application/json; charset=utf-8' -d @-
```

### Step 4: 检查结果

- `{"errcode":0,"errmsg":"ok"}` → 发送成功，立即执行 Step 5 终止任务
- 其他 errcode → 根据 errmsg 排查，修正后重试，最多 1 次

### Step 5: 清理并终止

1. 清理临时文件：`rm -f /c/xxx/.tmp_wecom_msg.json`
2. 输出一行完成摘要：`✅ 已发送：消息类型 | 时间 | errcode=0`
3. 任务结束，停止所有后续操作
