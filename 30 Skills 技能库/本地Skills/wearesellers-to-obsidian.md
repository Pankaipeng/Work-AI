---
title: wearesellers-to-obsidian
type: skill
group: 笔记 / 信息归档
source_level: 自建
upstream: —（无上游，本地原创）
local_path: C:/Users/Administrator/.workbuddy/skills/wearesellers-to-obsidian/
updated: 2026-11-16
tags: [skill, workbuddy, 自建, obsidian, 归档, 知无不言, 我的skill]
---

# wearesellers-to-obsidian

> [!tip] 🟢 自建（我个人创建）
> 把知无不言（wearesellers.com/question/<id>）帖子连同评论区全文归档到 Obsidian「10 Inbox 资料库」，固定结构为「**要点提炼置顶** → 问题正文 → 评论区全文」。

> **判断依据**：内容是本地工作流沉淀（用户已登录 Chrome → 剪贴板 → 本地 Obsidian 路径），无上游仓库痕迹。

## 速览

| 项 | 内容 |
|---|---|
| 来源 | **🟢 自建（我个人创建）** |
| 上游 | —（无上游，本地原创） |
| 分组 | 笔记 / 信息归档 |
| 调用 | `@wearesellers-to-obsidian` |
| 内部标题 | 知无不言帖子 → Obsidian Inbox 归档 |
| 本地路径 | `C:/Users/Administrator/.workbuddy/skills/wearesellers-to-obsidian/` |
| 最后更新 | 2026-11-16 |

## 触发场景

发 wearesellers 帖子链接 + “整理到 obsidian / 归档 / 包含评论区 / 同理”

## 内容结构

- **0. 铁律：要点提炼必须在笔记最顶部**
- **1. 抓取：全选复制 + 读剪贴板（唯一推荐路径）**
- **2. 整理**
- **3. 落盘（路径 / frontmatter / 结构）**
- **4. 收尾（关标签页、删临时文件）**
- **5. 反模式对照**

## 关键结论（踩坑沉淀）

| 问题 | 答案 |
|---|---|
| 评论区怎么拿？ | 必须登录。匿名请求只返回「N 个回复」数字，列表为空 |
| 登录态怎么用？ | 在用户已登录 Chrome 中新开标签页 → 点空白处 → `Control+A` → `Control+C` → Python `win32clipboard` 读剪贴板，一次拿全页含楼中楼 |
| 能不能用 Playwright 复制 profile？ | 不能。Chrome v20 ABE 加密，复制 profile 拿不到登录态，实测是未登录页面，还白占 3 GB+ |
| 为什么不能逐屏截图？ | 48 条回复要 30+ 屏、十几分钟，还容易漏行 |
| 要点提炼放哪？ | **笔记开头**，不要文末补尾巴（用户 2026-11-16 明确要求） |

## 全文（SKILL.md 原文）

> 原 frontmatter 已剥离，正文原样归档，便于全文搜索。

# 知无不言帖子 → Obsidian Inbox 归档

## 0. 铁律：要点提炼必须在笔记最顶部

用户明确要求过：**要点提炼放笔记开头**，不要放结尾。结构从上到下固定为：

```
frontmatter → 标题 → 元信息引用块 → 【要点提炼】→ 问题正文 → 评论区全文
```

自己加的任何“结论/启示/速记”都必须进顶部那块，不要在文末再补一段尾巴。

## 1. 抓取：全选复制 + 读剪贴板（唯一推荐路径）

评论区是服务端按登录态下发的，匿名请求只会返回「N 个回复」但列表空白。所以：

1. 取可用浏览器，在用户已登录的 Chrome 里**新开一个标签页**（不要抢占用户已有页面）：

```
mcp__browser__browsers_list
mcp__browser__tabs_new  {browser_id: <chrome>, url: "https://www.wearesellers.com/question/<id>"}
```

2. 点页面空白处拿到焦点，全选 + 复制（注意键名用 `Control+A` 这种首字母大写写法，`Control+a` 会报 unsupported key）：

```
mcp__browser__cua_execute {
  browser_id, tab_id,
  actions: [
    {type:"click", x:700, y:400},
    {type:"keypress", keys:"Control+A"},
    {type:"keypress", keys:"Control+C"}
  ]
}
```

3. 本地读剪贴板落盘（pywin32 已装；不落盘就白抓了）：

```python
import win32clipboard, win32con
win32clipboard.OpenClipboard()
t = win32clipboard.GetClipboardData(win32con.CF_UNICODETEXT)
win32clipboard.CloseClipboard()
open(".clip.txt", "w", encoding="utf-8").write(t)
```

一次就能拿到整页文本，**含楼中楼（楼中楼的署名格式是 `作者 • 城市 • 日期 HH:MM`）**。48 条回复的帖子约 17K 字符。

> 页面很长时 Ctrl+A 可能只覆盖已渲染部分：抓完先数一遍 `•\s*20\d\d-\d\d-\d\d` 的命中数，和页面上的「N 个回复」对不上就滚到底再复制一次。

## 2. 整理

- 直接用 Read 读 `.clip.txt`（17K 字符可直接读入），由模型自己结构化成 markdown，**不要写正则解析器**——噪声（`0 0 感谢`、`回复 举报`、重复作者名、侧栏广告/推荐位）比正则好处理。
- 正文/回复**照抄原文，不改写、不删减**；楼中楼挂在对应主回复下，用 `> 💬 **楼中楼**` 引用块。
- 折叠回复（`N 个回复被折叠`）看不到就在文末标注一句，不要编。
- 匿名用户多条时按内容区分，不要合并成一条。

## 3. 落盘

路径：`10 Inbox 资料库/(<question_id>) <标题> - 知无不言跨境电商社区.md`

frontmatter：

```yaml
---
created: <YYYY-MM-DDTHH:MM:00 (UTC +08:00)>
tags: [亚马逊, <主题词>, 知无不言]
source: https://www.wearesellers.com/question/<id>
author: <楼主名>
---
```

标题下先放元信息引用块（分类 / 圈子 / 发布地+日期 / 浏览 / 关注 / 回复数 / 最新活动），然后是**要点提炼**，再正文、评论区。

要点提炼要落到用户自己的业务视角（亚马逊运营：关键词、广告、Listing、库存），别写“该帖讨论了……”这种空气话。分歧明显时给「视角 | 核心主张」小表；有多条高赞一致时归纳成一个可执行的最小闭环。

## 4. 收尾（必须做）

```
mcp__browser__tabs_close {browser_id, tab_id}    # 只关自己开的那个
```

删掉临时文件（`.clip.txt` 等）。**绝不复制 Chrome profile 目录**：ABE 加密的 cookie 复制出来解不开，还会白占 3 GB+，用完必须删干净。

## 5. 反模式对照

| 别做 | 原因 |
|---|---|
| 逐屏滚动截图 + 肉眼转录 | 48 条回复要 30+ 屏，几分钟到十几分钟，还容易漏行 |
| `curl` 直抓页面 | 回复列表为空，只有「N 个回复」数字 |
| 复制 Chrome profile 给 Playwright 用登录态 | v20 ABE 解不开，实测拿到的是未登录页面 |
| 把要点提炼写在文末 | 违反固定结构 |
| 用 Obsidian 之外的路径存笔记 | Inbox 固定为 `10 Inbox 资料库` |

---

← 返回 [[00 本地Skills总览]]
