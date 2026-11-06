---
title: listing-orchestrator
type: skill
group: Listing 编排
source_level: 自建
upstream: —（无上游，本地原创）
local_path: C:/Users/Administrator/.workbuddy/skills/listing-orchestrator/
updated: 2026-11-06
tags: [skill, workbuddy, 自建, 我的skill]
---

# listing-orchestrator

> [!tip] 🟢 自建（我个人创建）
> 不生产文案的编排层：规定 builder / optimizer / listing-skill 谁在第几步出场、规则打架谁优先、排序公式怎么算、交付前跑脚本校验。

> **判断依据**：内容含「本项目实测数据」「本次真实踩过」的本机复盘，且自带 local-mapping.md 独立 overlay，无任何上游仓库痕迹。
>

## 速览

| 项 | 内容 |
|---|---|
| 来源 | **🟢 自建（我个人创建）** |
| 上游 | —（无上游，本地原创） |
| 分组 | Listing 编排 |
| 调用 | `@listing-orchestrator` |
| 内部标题 | Listing Orchestrator — 编排与仲裁层 |
| 本地路径 | `C:/Users/Administrator/.workbuddy/skills/listing-orchestrator/` |
| SKILL.md | 120 行 |
| 最后更新 | 2026-11-06 |

## 触发场景

写/改/优化 listing、跑八步、规则冲突、字符超限、标题排序、五点排序、listing 定稿

## 内容结构

- **0. 定位（先读这一节，否则会重复造轮子）**
- **1. 何时用 / 何时不用**
- **2. 标准调用链（S0–S8，顺序不可跳）**
- **3. 冲突仲裁（本 skill 的核心价值）**
  - 3.1 五条优先级（从上到下，高位直接压制低位）
  - 3.2 已知冲突条目（照此裁决，不要重新讨论）
- **4. 三条排序规则（摘要，公式见 reference/scoring-rules.md）**
- **5. 输入必须完整（S0 拦截）**
- **6. 变体派生（主推先定，别的照抄）**
- **7. 交付前校验（S8）**
- **8. 反模式（本次真实踩过，逐条对应）**
- **9. 文件索引**

## 附带文件

- `README.md`
- `reference/conflict-arbitration.md`
- `reference/input-card-template.md`
- `reference/local-mapping.md`
- `reference/scoring-rules.md`
- `scripts/validate.py`

## 全文（SKILL.md 原文）

> 原 frontmatter 已剥离，正文原样归档，便于全文搜索。

# Listing Orchestrator — 编排与仲裁层

## 0. 定位（先读这一节，否则会重复造轮子）

本 skill **不生产文案**。文案由三个已有 skill 生产，本 skill 只做四件事：

| 职责 | 为什么必须单独有一层 |
|---|---|
| **编排**：规定谁在第几步出场 | 三个 skill 都自称能"从头写 listing"，同时加载会互相覆盖 |
| **仲裁**：规则打架时定谁优先 | builder 建议五点 200–300，listing-skill 上限 255，optimizer 不表态，历史上靠人肉裁 |
| **量化**：三条排序规则给出可复算公式 | 三个 skill 都只说"按重要性排"，没有一条给了算法 |
| **把关**：输入缺字段拦截 + 交付前脚本校验 | 模型自查不可靠（漏一个冒号就 FAILED），必须外部判定 |

> 一句话：**builder 建库、optimizer 打分、listing-skill 收敛格式、本 skill 管秩序和验收。**

## 1. 何时用 / 何时不用

**用**：新品 listing 全流程；已有文案要改；多变体（父子体）文案；三个 skill 结论不一致；字符数反复超限；被要求"按买家思维重写"。

**不用**：只问单个字段的规则解释（直接查 `06_rules/platform_rules.md` 或 `amazon-listing-skill/references/`）；只做关键词调研（走 `amazon-keyword-strategy`）；只做 A+ 图片。

## 2. 标准调用链（S0–S8，顺序不可跳）

| 步 | 动作 | 用谁 | 门禁 / 产出 |
|---|---|---|---|
| **S0** | 收集输入卡 | 本 skill `reference/input-card-template.md` | 12 项必填缺任意一项**先问，不许猜**；缺项影响文案 → 停 |
| **S1** | 数据采集 | 卖家精灵 MCP（`mcp__ss_mcp__`），映射见 `reference/local-mapping.md` | 失败重试 1 次 → 仍失败标 `DATA_MISSING`，**禁止**抓 amazon.com 凑数 |
| **S2** | 建三库（词库 / 问题库 / 证据库） | `amazon-listing-builder` Step 1–3 | 三库不齐不进 S3。问题库≥15 题、证据库每条带来源等级 A/B/C |
| **S3** | 生成 v1 | `amazon-listing-builder` Step 4–8 | 同时给≥2 版可对比草稿 |
| **S4** | 诊断评分 | `amazon-listing-alexa-optimizer` | 输出 8 维度评分 + 行动清单，N/A 按 5/10 中性计，不得判死 |
| **S5** | 收敛格式 | `amazon-listing-skill`（Title Case、`header: description`、无句末标点、ST 249 字节） | 五点命中 `200–255` 区间；IH 不与标题重复词 |
| **S6** | 排序复核 | 本 skill `reference/scoring-rules.md` | 标题 / IH / 五点各给一张"候选-得分-依据"表，可复算 |
| **S7** | 变体派生 | 本 skill 第 6 节 | 主推定稿后，颜色变体只允许改 N 处，逐处列清单 |
| **S8** | 交付前校验 | `scripts/validate.py` | 必须打印 `PASSED` 才能交付；`FAILED` 逐条改完重跑 |

**S3 之后任何一次改动，都必须重跑 S8。** 改一个字也算。

## 3. 冲突仲裁（本 skill 的核心价值）

### 3.1 五条优先级（从上到下，高位直接压制低位）

1. **平台硬规则** > 任何 skill 的写作偏好（2026 标题 75 / IH 125 / 合计 200 / 禁用字符 / 后台 ST 249 字节）
2. **用户口述** > ERP 导出表 > MCP 调研数据（数据类冲突唯一裁决顺序）
3. **更严者优先**（长度上限、禁用词、格式约束取两边更严的一侧）
4. **职责不越界**：生成用 builder，评分用 optimizer，定稿格式用 listing-skill，三者不交叉覆盖
5. **裁不了就摆出来**：把两种写法并列给用户二选一，**不许自己拍**，也不许"综合一下"

### 3.2 已知冲突条目（照此裁决，不要重新讨论）

| 冲突点 | 三方说法 | 裁决 |
|---|---|---|
| 五点长度 | builder 200–300 / listing-skill 10–255 | **200–255**（重叠区，两边同时满足） |
| 标题结构 | builder 核心词前置 / optimizer 可读句子 | 两者可兼得：`品牌 + 权重最高短语 + 差异属性`，一个逗号断句，不写完整句 |
| IH 与标题关系 | builder 无硬约束 / 平台"不得重复标题关键词" | **平台优先**，IH 只用标题没出现过的词 |
| 数据来源 | builder 禁浏览器抓 Amazon / optimizer 主推浏览器 | **MCP 优先**（本机已接入），浏览器只在 MCP 缺字段时兜底并标注 |
| 五点 header 写法 | 三家都没规定 | 用户规则优先：**header 写成买家问题**（如 `Will it fall off:`），不写名词短语 |
| ST 是否要与标题去重 | listing-skill 要求查覆盖 | **不去重**。后台字段独立计算，标题里有也必须再写 |
| 描述格式 | 平台允许基础 HTML / listing-skill 要纯文本 | **纯文本 + 空行分段**（最稳，不触发解析问题） |
| 变体文案差异度 | 无规定 | 只改与颜色/材质强相关的字段，其余逐字一致（见第 6 节） |

## 4. 三条排序规则（摘要，公式见 `reference/scoring-rules.md`）

1. **标题**：权重 = 月搜索量 × 购买率，降序占位。品牌固定首词，权重最高短语紧跟，差异属性收尾。
2. **Item Highlights**：按"竞品组内唯一性 × 买家决策权重"排，先放别人没有的（透明/双安装），再放参数。
3. **五点**：重要性 = 差评频次占比 × 对应词购买率，降序。**决策前置修正**：影响"买不买"的排前面，影响"评价好不好"的排后面。

三条都必须输出可复算的表，不许只给结论。

## 5. 输入必须完整（S0 拦截）

见 `reference/input-card-template.md`。四项**缺一即停**：

- 材质与颜色**逐 SKU** 对应关系（不是"PC 塑料"一句带过）
- 尺寸口径（本体净尺寸 or 包装尺寸，必须指明）
- 主推 SKU（不变体稿并行写）
- 合规禁区（不能宣称什么）

## 6. 变体派生（主推先定，别的照抄）

1. 先把主推 SKU 写到 `PASSED`，**再**派生其余颜色。
2. 只允许改 4 类字段：标题颜色词、材质词、颜色相关描述句、颜色相关 IH 短语。
3. 派生后必须列"逐处差异清单"（字段 / 原文 / 新文 / 原因），差异条数超过 5 处说明在主推阶段就没写对。
4. Search Terms **三色共用一份**，不按颜色拆分（除非该颜色有独立搜索量证据）。

## 7. 交付前校验（S8）

```bash
python scripts/validate.py listing.json                 # 单稿
python scripts/validate.py clear.json black.json white.json   # 多变体一致性
```

校验项：标题 75 / 禁用字符 / 促销词 / 实词≤2 次 / Title Case；IH 125 / 无句读 / 不重复标题词；五点 200–255 / 冒号 header / 无句末标点；描述 2000 / 无 HTML 外链；ST 249 字节 / 小写 / 无重复词；全字段商标黑名单；绝对化宣称红线；多变体差异白名单。

**只有 `PASSED` 才能交付。** `WARN` 允许交付但必须在交付说明里逐条列出。

## 8. 反模式（本次真实踩过，逐条对应）

| 反模式 | 症状 | 正确做法 |
|---|---|---|
| 参数表思维 | 五点写成 `Material: PC, Size: 8.5 in` | 先写痛点场景，参数塞进句子尾部 |
| 变体并行 | 三色同时写，改一处要改三遍 | 主推写完 PASSED 再派生 |
| 边改边换前提 | 第 4 版才发现尺寸是包装尺寸 | S0 把口径问死 |
| 靠"再优化一版"救 | v1→v5 每版只挪句式 | 换视角重写，不基于前版改（破锚定） |
| 模型自检 | "我已检查过字符数" | 跑 `scripts/validate.py` |
| 竞品商标进文案 | 五点出现 Scrub Daddy | 全字段黑名单拦截，截流只放广告手动精准 |

## 9. 文件索引

```
listing-orchestrator/
├── SKILL.md                              ← 本文件：编排、仲裁、门禁
├── reference/conflict-arbitration.md     ← 仲裁表详版 + 本次 v1–v6 真实冲突复盘
├── reference/scoring-rules.md            ← 三条排序规则的公式、阈值、反例对照
├── reference/input-card-template.md      ← 输入卡填空模板 + 缺字段询问清单
├── reference/local-mapping.md            ← 本机环境适配（MCP 工具名/参数/刻度/能力边界）
└── scripts/validate.py                   ← 交付前确定性校验
```

上游 skill 更新时，**只覆盖它们自己的目录**；本目录的 `local-mapping.md` 是独立 overlay，不会被动到。

---

← 返回 [[00 本地Skills总览]]