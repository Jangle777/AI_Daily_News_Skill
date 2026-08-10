---
name: ai-briefing-template
description: 每日AI资讯简报HTML模板生成。提供极简杂志风CSS和HTML骨架，为edu-ai-briefing-principles技能输出的结构化新闻数据生成最终HTML文件。触发词：AI简报模板、生成简报HTML、套模板。重要：所有颜色使用HEX/RGBA格式，绝不使用oklch()，确保企业微信内置浏览器兼容。
agent_created: true
---

# AI 简报 HTML 模板

## 职责

根据 `edu-ai-briefing-principles` 输出的结构化新闻数据，套用杂志风模板生成最终HTML文件。

## CSS 模板

读取 `assets/template.css` 并原样内联到 HTML `<style>` 标签中。

**关键兼容性规则（不可违背）：**

1. **颜色格式**：CSS中所有颜色必须使用十六进制 (`#rrggbb`) 或 `rgba()` 格式。**严禁使用 `oklch()` 颜色函数**——企业微信内置浏览器不支持该CSS函数，会导致所有颜色失效，页面显示错误。
2. **字体降级**：衬线字体优先 `"Noto Serif SC", "Source Han Serif SC", "Songti SC", Georgia`，无衬线优先 `"PingFang SC", "Hiragino Sans GB", "Microsoft YaHei"`。
3. **响应式**：使用 `@media (max-width: 600px)` 适配移动端。

## HTML 骨架

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>今日 AI 速递 · YYYY年MM月DD日</title>
  <style>
    /* 内联 assets/template.css 全部内容 */
  </style>
</head>
<body>
  <!-- Masthead -->
  <div class="masthead">
    <h1>今日 AI 速递</h1>
    <div class="sub">YYYY年MM月DD日 · 星期X · 共 N 条</div>
  </div>

  <!-- Summary Box -->
  <div class="summary-box">
    <div class="summary-title">今日摘要</div>
    <ul class="summary-list">
      <li>摘要1（≤20字）</li>
    </ul>
  </div>

  <!-- 各栏目 -->
  <!-- 栏目名 | cat-类 | 标题 | URL | 来源 | 时间 | 正文段落数组 -->
  <div class="section-header"><h2>栏目名</h2></div>
  <div class="article cat-xxx">
    <div class="article-num">NN</div>
    <a class="title" href="URL" target="_blank">标题</a>
    <div class="meta-row">
      <span class="source-tag">来源</span>
      <span class="time-tag">时间</span>
    </div>
    <div class="summary">
      <p>正文段落</p>
    </div>
  </div>

  <!-- Footer -->
  <div class="footer">
    数据来源：aihot.virxact.com 及一手来源<br>
    仅供内部参考 · 内容不代表公司立场
  </div>
</body>
</html>
```

## 栏目与CSS类名映射

| 栏目名 | CSS类 | 用途 |
|--------|-------|------|
| 热点头条 | `cat-top` | 蓝色主题，当天最热门 |
| AI+教育 | `cat-edu` | 绿色主题，教育场景AI |
| 模型发布 | `cat-model` | 紫色主题，新模型发布 |
| 产品动态 | `cat-tool` | 蓝青色主题，新产品/功能 |
| 技巧与观点 | `cat-tip` | 琥珀色主题，AI技巧与观点 |
| 行业速览 | `cat-brief` | 灰色主题，精选速览 |

## 编号规则

全局编号贯穿全文（01, 02, 03 ... N），不在每个栏目内重新计数。编号 N 即为文章总条数。

## 头部条数

masthead 中的 `共 N 条` 必须等于实际文章 `.article` 标签的数量。**具体做法**：将所有文章填入HTML后，数一下写了多少个 `.article` 块，然后将这个数字填入 `共 N 条`。禁止手动估算或依赖直觉，必须逐条计数。

## 摘要规则

- 仅从"热点头条""AI+教育""模型发布""产品动态"四个栏目提取
- 摘要条目数必须等于这四个栏目的新闻总条数——每条新闻一条摘要，不可遗漏
- 每条摘要 ≤20 字
- "技巧与观点"和"行业速览"不纳入摘要
- `summary-list` 中每项以 `—` 开头

## 时间格式

北京时间，人话格式："今天 HH:MM" / "昨天 HH:MM" / "MM-DD HH:MM"
