---
title: 00 本地Skills总览
type: moc
updated: 2026-11-06
tags: [skill, moc, workbuddy, 我的skill]
---

# 本地 Skills 总览

本机 `~/.workbuddy/skills/` 共 **11 个**技能中已判定来源的 11 个：
**🟢 自建 5 个** · 🟡 第三方+本地适配 2 个 · ⚪ 第三方原样 4 个。

原始文件仍是唯一事实来源，本库只做索引与全文备份。改规则请改本地 skill，再重跑 `build_skill_notes.py` 同步。

## 🟢 我个人创建（5）

| Skill | 一句话 |
|---|---|
| [[listing-orchestrator\|`listing-orchestrator`]] | 不生产文案的编排层：规定 builder / optimizer / listing-skill 谁在第几步出场、规则打架谁优先、排序公式怎么算、交付前跑脚本校验。 |
| [[amazon-xlsm-listing-builder\|`amazon-xlsm-listing-builder`]] | 把定稿文案 + 商品资料（可自动提取领星 ERP 导出表）填进亚马逊官方批量上架 .xlsm，产出能直接 Add Products via Upload 的文件，并解析回执报错。 |
| [[lingxing-fba-carrier-compare\|`lingxing-fba-carrier-compare`]] | 同一批货、两家货代整批比价：每家各仓取最便宜含税海卡渠道，加总成整批合计运费，输出差额、差额占比、时效与一句话结论。 |
| [[lingxing-fba-shipping-plan\|`lingxing-fba-shipping-plan`]] | 装箱单 + 货件列表 + 一家货代报价，自动产出出货明细（含最优渠道/单价/时效）、恒信报关发票（40 列）、仓库贴标备注对接表，并能把箱唛按票发到工厂对接群。 |
| [[amazon-report-archive\|`amazon-report-archive`]] | 把亚马逊后台下载的回款/超库龄/仓储费/广告报表按规范名归档，并生成「N月回款备注.xlsx」与「N月利润核对表.xlsx」；缺数据一律留空不编造。 |

> 判定依据：`SKILL.md` 带 `agent_created: true`，或内容只对本机成立（你自己的飞书 tenant / 领星字段 / 本机 MCP 前缀 / 本项目复盘），且无上游仓库痕迹。

## 🟡 第三方 + 本地适配（2）

| Skill | 一句话 | 来源 |
|---|---|---|
| [[amazon-listing-alexa-optimizer\|`amazon-listing-alexa-optimizer`]] | 按 AI 购物时代（Alexa for Shopping）的 8 大维度给已有 Listing 体检打分，覆盖 2026 标题 75 字符新规、Item Highlights 125 字符、后台属性完整性。 | **三方适配** |
| [[amazon-listing-builder\|`amazon-listing-builder`]] | 以 Cosmo 语义算法 + Alexa/Rufus 对话式购物为基准，用八步工作流把 Listing 从「堆关键词」改成「语义覆盖 + 需求证据 + 答案型内容」。 | **三方适配** |

> 上游 SKILL.md 未改，本机差异全放在独立 `local-tool-mapping.md` overlay 里 —— 升级上游时只需保留 overlay 文件。

## ⚪ 第三方原样引入（4）

| Skill | 一句话 | 来源 |
|---|---|---|
| [[amazon-listing-skill\|`amazon-listing-skill`]] | 合规向的 Listing 写作与审计：各站点（含沙特、埃及、土耳其、阿联酋）标题/五点/描述/后台 Search Terms 的字数、禁词、商标敏感度规则库。 | **第三方** |
| [[amazon-keyword-strategy\|`amazon-keyword-strategy`]] | 从竞品或自家 ASIN 反查出词，判标品/非标品，做词根六分类，最后输出逐词带打法（主攻/截流/否定）的广告作战图。 | **第三方** |
| [[amazon-ads-creative-assets\|`amazon-ads-creative-assets`]] | 端到端做广告创意素材：按规格生成合规图片（商品图/场景图/品牌 logo/展示广告横幅）与广告文案，再通过 Amazon Ads MCP 的 creative assets API 注册、版本化、查状态。 | **第三方** |
| [[amazon-ads-empty-plate-composite\|`amazon-ads-empty-plate-composite`]] | 专治「AI 把产品结构画歪」：先生成空镜场景，再抠真实产品照片合成，让滑轨、卡扣、铰链、磁吸这类机械结构严格对版。 | **第三方** |

## 按分组

### Listing 编排

- 🟢 [[listing-orchestrator]] — 写/改/优化 listing、跑八步、规则冲突、字符超限、标题排序、五点排序、listing 定稿

### Listing 文案

- 🟢 [[amazon-xlsm-listing-builder]] — 填上架模板、生成上架表、批量上传文件、flat file、父子变体合并、processing summary 复盘
- [[amazon-listing-alexa-optimizer]] — 检查/优化/诊断/评分 Listing、75 字符新规、Item Highlights、Alexa 推荐
- [[amazon-listing-builder]] — 写 Listing/标题/五点/描述/A+/QA/Search Terms、Cosmo 算法、Rufus、语义搜索
- [[amazon-listing-skill]] — 写或审 listing、本地化、商标风险、合规检查、marketplace 特定规则

### 关键词 / 选品

- [[amazon-keyword-strategy]] — 搭关键词库、新品建词库、关键词调研、反查建库、PPC 选词、否定词建立、关键词分层

### 广告素材

- [[amazon-ads-creative-assets]] — SB/SD/DSP 广告创意、图片尺寸与格式、素材被拒、assetId、上传注册创意资产
- [[amazon-ads-empty-plate-composite]] — 结构画错了、抽屉/托盘不对、产品变形、悬浮感、换背景、空镜

### 供应链 / 发货

- 🟢 [[lingxing-fba-carrier-compare]] — 比价、哪家便宜、货代报价对比、两家运费差多少、整批走哪家
- 🟢 [[lingxing-fba-shipping-plan]] — 领星发货计划、装箱单转发票、报关发票、贴标备注、仓库对接表、指定货代出计划

### 财务 / 报表

- 🟢 [[amazon-report-archive]] — 归档本月报表、整理月度报表、生成利润核对表、做回款备注、跑一下 N 月的表

## 飞书链接汇总

| Skill | 用途 | 链接 |
|---|---|---|
| [[amazon-xlsm-listing-builder\|amazon-xlsm-listing-builder]] | USAGE.md 在线版（可直接发同事） | https://bqnscp788os.feishu.cn/docx/SgQSd3tFXoD6i2xvMQzcmLVenFi |
| [[amazon-report-archive\|amazon-report-archive]] | 在线使用说明（永远最新） | https://bqnscp788os.feishu.cn/docx/Na7xdV9xmoIZxXxx3vHcLCGanxf |
| [[amazon-report-archive\|amazon-report-archive]] | 技能包下载（.zip，约 43KB） | https://bqnscp788os.feishu.cn/file/LsclbdKDPoZouXxQhyscQkmkn9b |
| [[amazon-report-archive\|amazon-report-archive]] | 飞书云盘根目录「财务数据」 | https://bqnscp788os.feishu.cn/drive/folder/AItLfwU1clx70TdEqoDcaHAKnTo |
| [[amazon-keyword-strategy\|amazon-keyword-strategy]] | 作者社区介绍（新西楼.AI，非本组织） | https://my.feishu.cn/wiki/WNi0wh3mIiOLhzkCLFPcwUr9nrg |

> `bqnscp788os.feishu.cn` 是你本组织的 tenant（自建 skill 的对外文档）；`my.feishu.cn` 那条是第三方作者的社区，非本组织。

## 来源怎么判断（下次同步自查）

| 信号 | 含义 |
|---|---|
| frontmatter `agent_created: true` | WorkBuddy 生成的本地 skill → 自建 |
| 存在 `local-tool-mapping.md` / `local-mapping.md` | 上游 + 本机 overlay → 三方适配 |
| 有 `LICENSE` + 英文正文 + 作者外链 | 外部仓库原样引入 → 第三方 |
| 内容含 `_backup/`、本机路径、你的飞书 token | 本地迭代产物 → 自建 |

## 使用方式

对话里 `@skill名` 直接调用，例如 `@listing-orchestrator`。

## 待办

- [ ] `ziniao-*` 系列 15 个（紫鸟 / OpenAPI 导入，同一秒创建，非自建）暂未归档
- [ ] `amazon-ads-empty-plate-composite` 上游未留链接，待确认是否同属 KuudoAI/skills
