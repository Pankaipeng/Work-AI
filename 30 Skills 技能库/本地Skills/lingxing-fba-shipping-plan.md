---
title: lingxing-fba-shipping-plan
type: skill
group: 供应链 / 发货
source_level: 自建
upstream: —（无上游，本地原创）
local_path: C:/Users/Administrator/.workbuddy/skills/lingxing-fba-shipping-plan/
updated: 2026-11-12
tags: [skill, workbuddy, 自建, 领星, 我的skill]
---

# lingxing-fba-shipping-plan

> [!tip] 🟢 自建（我个人创建）
> 把 FBA 箱唛（自动解析成装箱明细）、领星货件（FBA号/PO/仓库，走 MCP 直接取）与**一家**货代的周报价表，自动生成 FBA 发货计划——出货明细（含该货代最优渠道/单价/时效）、恒信报关发票（55 列完整版）、仓库贴标备注对…

> **本笔记由 `scripts/sync_vault.py` 从 SKILL.md 生成**，不要手改正文；
> 技能变更后重跑脚本即可。生成时间 2026-11-12。

## 速览

| 项 | 内容 |
|---|---|
| 来源 | **🟢 自建（我个人创建）** |
| 上游 | —（无上游，本地原创） |
| 分组 | 供应链 / 发货 |
| 调用 | `@lingxing-fba-shipping-plan` |
| 本地路径 | `C:/Users/Administrator/.workbuddy/skills/lingxing-fba-shipping-plan/` |
| SKILL.md | 481 行 |
| 最后更新 | 2026-11-12 |

## 触发场景

把 FBA 箱唛（自动解析成装箱明细）、领星货件（FBA号/PO/仓库，走 MCP 直接取）与**一家**货代的周报价表，自动生成 FBA 发货计划——出货明细（含该货代最优渠道/单价/时效）、恒信报关发票（55 列完整版）、仓库贴标备注对…

## 对外链接

| 用途 | 说明 | 链接 |
|---|---|---|
| 使用说明 | 在线版，永远最新；由 scripts/sync_feishu.py 从 使用说明.md 覆盖更新 | https://bqnscp788os.feishu.cn/docx/LdkbdfqYkoLES1xoRpHcjyQ8nuh |
| 技能包 | 约 300KB，含发货计划 + 比价两个技能；由 scripts/make_skill_package.py 打包，改动后重新打包并覆盖上传同一个 file_token | https://bqnscp788os.feishu.cn/file/TjEMbEqYvoyu7uxmv2pcqVoOn5c |


## 内容结构

- **安装与分发**
- **用途**
- **浏览器交互：一律由用户执行（硬规则）**
- **开始前先收集输入**
- **货件数据怎么取（替代亚马逊后台导出）**
- **本地参照库（报关要素 / 箱规 / 服务代码）**
- **执行流程**
- **关键口径（易错，务必遵守）**
- **脚本约定**

## 附带文件

- `SKILL.md`
- `config.json`
- `使用说明.md`
- `安装说明.md`
- `references/business-rules.md`
- `references/field-mapping.md`
- `references/input-schemas.md`
- `references/notice-template.md`
- `references/quote-rules.md`
- `scripts/attachments.py`
- `scripts/build_plan.py`
- `scripts/build_refdata.py`
- `scripts/gate.py`
- `scripts/labels_to_packing.py`
- `scripts/make_bundle.py`
- `scripts/make_skill_package.py`
- `scripts/notice.py`
- `scripts/quote_parser.py`
- `scripts/quoting.py`
- `scripts/readers.py`
- `scripts/send_notice.py`
- `scripts/send_robot.py`
- `scripts/send_void.py`
- `scripts/sendlog.py`
- `scripts/sync_feishu.py`
- `scripts/sync_vault.py`
- `assets/config.example.json`
- `assets/links.json`
- `assets/service_codes.csv`
- `assets/产品箱规.csv`
- `assets/发货模板_恒信_10.22.xlsx`
- `assets/报关要素.csv`

## 全文（SKILL.md 原文）

> 原 frontmatter 已剥离，正文原样归档，便于全文搜索。

# 领星 FBA 发货计划生成（单货代）

## 安装与分发

- 技能包（含本技能 + `lingxing-fba-carrier-compare` 比价技能）：
  https://bqnscp788os.feishu.cn/file/TjEMbEqYvoyu7uxmv2pcqVoOn5c （约 300KB）
- 同事装法（一句话）：把技能包**拖进 WorkBuddy 对话框**，说「安装本技能包及其依赖」——
  包根目录有 `AI-安装指令.md`，AI 会自己解压、复制到 `<用户名>/.workbuddy/skills/`（只一层同名目录）、
  `pip install openpyxl pypdf`、自检并汇报，最后提醒重启。
  用户明确要手动装时，再让他解压放目录 + 装依赖 + 重启，详见本目录 `安装说明.md` 2.2。
- 改完技能要重新发版：`python scripts/make_skill_package.py --out <zip>` 打包（会自动把 `AI-安装指令.md`
  写进包根目录），再用同一个 `file_token` 覆盖上传（地址记在 `assets/links.json`），链接不变。
- 用户问「怎么装 / 装在哪 / 同事怎么用」直接答上面三条，别再翻其他文档。

## 用途

把一批待发 FBA 货物，从**箱唛 + 报价**两份原始资料变成可直接对外的三张表 + 一张校验报告：

| 输入 | 输出 |
|---|---|
| ① **FBA 箱唛 PDF**（唯一必给的原件，装箱明细由它解析） | ① **发票**：恒信报关资料，55 列，一票×SKU 一行 |
| ② **货件数据（领星 MCP `get_fba_shipment_list` 的 JSON，推荐）** | ② **仓库对接表**：箱子编号+货件信息+货件标题（贴标备注） |
| ③ **本周该货代的报价表（1 份）** | ③ **出货明细**：含该货代内选中的渠道/单价/时效 |
| ④ 字典表（地址库/型号库/服务类型，默认复用） | ④ **校验报告**：需人工确认的缺口 |
| ⑤ 本地参照库（报关要素/产品箱规/服务代码，`assets/*.csv`） | ⑤ **群通知**（可选）：一票一条文字 + FBA箱唛/货代唛头两份 PDF |

**货件数据不再依赖亚马逊后台导出**：领星 `get_fba_shipment_list` 一条记录同时给 FBA 货件号、
Reference ID（发票 PO Number）、目的仓库代码、亚马逊配送地址、送达时段与申报数量，走 MCP 取一次即可。
亚马逊「货件」页导出的 xlsx 仍然兼容（`--shipments` 认 .json/.xlsx/.csv），但没有 JSON 时优先用它。

**本技能只出一家货代的计划，整批不混用货代。** 需要先比较两家再定货代，请先用 `lingxing-fba-carrier-compare` 技能出结论，拿到选定货代后再回来跑本技能。

## 浏览器交互：一律由用户执行（硬规则）

**不要接管浏览器。** 不点鼠标、不按键、不填表单、不开新标签页、不猜下载结果。
领星建 STA、改箱规、导出箱唛、恒信上传建单、亚马逊填追踪号这类后台操作，**全部由用户在浏览器完成**，
本技能只负责「出操作单 → 收用户回传 → 继续算数出表」。理由：这些后台是懒加载 + 非语义化控件，
坐标级自动化不稳（点不中、误触发提交、判定不出是否下载成功），纠正成本远高于人手一遍；且一旦混手，出问题无法归因。

能走 MCP / 导出文件的绝不走浏览器（发货仓从 `fba_plan_group_list` 取，FBA 号 / Reference ID / 仓库
一律从 `get_fba_shipment_list` 取）。

交接时**必须**按这个模板给出，用户照着点即可：

```text
【这一步要你在浏览器做】
1. 打开 <页面/入口>
2. 操作：<3~6 步，每步一句，写清控件名与选项>
3. 关键参数：<箱规 / 送达时段 / 承运人 / 标签尺寸…，可直接抄>
4. 做完把这些发我：<只回传浏览器里才有的东西，如 5 个 FBA 号 + 建单生成的 PT 运单号；
   Reference ID / 仓库 / 数量都走 MCP 自己取，不要用户抄>
5. 文件下载到哪：<明确目录，或"下载后把路径发我">
```

用户回**三件套**即可继续：① 号码（FBA 号 / PT 运单号 / 报价合计；Reference ID 不用用户给）；
② 下载文件所在目录；③ 报错文字或截图。

同一 UI 动作用户已重试 2 次、或本阶段超过 10 分钟没有新产出时，**立刻 handoff**，不要硬撑。

## 开始前先收集输入

**用户只需要给两样：FBA 箱唛 PDF + 本周该货代的报价表。** 其余由脚本自己取或生成，别去要。

1. **FBA 箱唛 PDF**（必需，唯一要用户给的原始资料）——亚马逊 STA 导出的 `STA_*.zip`（要解压）。
   用 `scripts/labels_to_packing.py` 解析成装箱单：每页一箱，读到 FBA 货件号、箱序/总箱数、单箱毛重、
   MSKU/FNSKU、单箱数量、**箱号 `P…-B…`**。箱号只有箱唛上有，所以**箱唛下完才能出正式表**。
   外箱尺寸箱唛不印，按 SKU 从 `assets/产品箱规.csv` 补，缺失就报错不猜。
2. **货件数据**（必需，但**不用用户给**）——领星 MCP `get_fba_shipment_list` 的原始 JSON 存盘即可，
   一次给齐 FBA 号 / Reference ID / 仓库代码 / 配送地址 / 送达时段。见下节「货件数据怎么取」。
3. **货代报价表（必需，只要 1 份）**——本周最新那版，文件名含「恒信/辛巴」或「凯琦」等，脚本按文件名识别货代名。**不要传两家**，传了会直接报错并提示改用比价技能。
4. **字典表**（默认复用）——地址库、发票型号汇总、服务类型，取自 `assets/发货模板_恒信_10.22.xlsx`。
5. ~~Amazon Reference ID~~ **不用再手抄**：货件 JSON 的 `reference_id` 就是 PO Number，脚本自动带进发票；
   只在没取到 JSON（用发货计划 `mws_relate` 兜底）时才需要 `--ref-id FBA=参考号` 补。
6. **提货地点 / 所在货站**（**不用用户给**）——**提货地点是按 SKU 定的，取领星发货计划的「发货仓库」**（计划行 `wname`）：
   工厂直发的 SKU 是「工厂直发-厨之乐 / 工厂直发-武航」，备货仓的 SKU 是「中山青吉仓库」。
   同一票里两种经常混装，所以出货明细逐行写各自的发货仓库、仓库对接表一行一个、群通知每行标 `（XX贴）`。
   型号库「工厂」列只是老数据兜底，`config.carrier.提货地点` 是最后兜底。

> **不要向用户索要「装箱单」。** 历史上有两种错法：① 让用户去领星导装箱单（多一步手工）；
> ② 先建 STA 再拿箱唛反解析装箱单，却对用户说"用了你给的装箱单"（实际等于自己抄自己）。
> 正确说法：「把箱唛 PDF 给我，其余我自己取」。箱唛未下、用户要先看计划时，可按领星计划生成**粗装箱单**
>（SKU/箱数/单箱数/发货仓齐全，缺箱号与每箱重量），并明确标注「箱唛下完出正式版」。

用户只说"按模板来"、附件未上传时：先用 `assets/发货模板_恒信_10.22.xlsx` 跑通并展示结果形态，同时列出待补文件清单，不要凭空编造装箱数据。

## 货件数据怎么取（替代亚马逊后台导出）

走领星 MCP，两个调用就够，**取回来的 JSON 原样存盘**（如 `input/领星货件_<日期>.json`）再交给脚本：

```text
help(query="货件")                     # 找 get_fba_shipment_list
action get_fba_shipment_list           # search_field_time="create_date"
                                       # is_sta="1"（只要 STA 货件）
                                       # is_closed="0"、shipment_status[]="WORKING"
                                       # start_date/end_date = 建 STA 那天
                                       # seniorSearchList[] 可按 shipment_id 精确捞单票
                                       # 其余必填字段传空串，sort_field="gmt_create"
```

返回 `data.data.list[]` 每条的字段与用途：

| 字段 | 用途 |
|---|---|
| `shipment_id` | FBA 货件号（装箱单里的 FBA 编码，按它对票） |
| `reference_id` | **发票 PO Number 货件跟踪编码**（如 5B3AT7GN） |
| `destination_fulfillment_center_id` | 目的仓库代码（发票「地址库编码」/选渠道用） |
| `orig_ship_to_address` | 亚马逊配送地址（地址库缺该仓时兜底，邮编非数字会判为无效） |
| `sta_delivery_start_date` / `_end_date` | 送达时段（核 STA 填得对不对） |
| `total_quantity_shipped` | 后台申报量，用来和装箱单件数对账 |
| `item_list[].sku / quantity_shipped` | 该货件的 SKU 与分货数量 |

**一手数据只认货件 JSON**：`--shipments` 给 JSON 时，FBA 号、PO、仓库、地址、申报量全部来自它；
没给时脚本退到发货计划的 `mws_relate`（有 FBA 号+仓库，**没有 PO**，校验报告会提示用 `--ref-id` 补）。

## 本地参照库（报关要素 / 箱规 / 服务代码）

三份 CSV 都**从原始导出生成，不手改**；原始表更新后重跑一次即可：

```bash
python scripts/build_refdata.py \
  --products "D:/Pictures/领星产品材质、报关编码确认版.xlsx" \
  --service-template "D:/WXWork/<id>/Cache/File/<月>/辛巴发票模版2026-10-15.xlsx"
```

| 文件 | 内容 | 谁在用 |
|---|---|---|
| `assets/报关要素.csv` | SKU → 中英文报关名 / 材质 / 用途 / HSCODE / 报关型号 | 覆盖型号库的申报字段，**英文报关名为空直接报错，不编造** |
| `assets/产品箱规.csv` | SKU → 外箱长宽高 / 单箱数量 / 单箱重量（领星主数据） | 箱规预检，箱数与主数据不符提前报 |
| `assets/service_codes.csv` | 货代服务代码白名单（122 条，含包税方式/类型/分类） | 发票「服务*」逐行校验，`US-TRKP` 这类错码本地就拦，并给最接近的候选 |

脚本默认读这三份（`--customs-lib` / `--box-spec` / `--service-codes` 可换路径）。

## 执行流程

### 1. 确认配置

读 `config.json`，核对与用户口径若不符就改：

- `shop.店铺 / 客户订单号前缀 / 申报店铺名`（默认 巨杉 / JS / 默认店铺）
- `carrier.货代`（兜底货代名）、`carrier.提货地点`（**最后兜底**，正常都从领星计划行的发货仓库取，见 `--warehouse-map`）
- `invoice.申报单价 / 产品用途 / 产品品牌`（默认 0.5 / 置物 / Nieifi）
- `invoice.所在货站`（默认 中山集散中心）、`invoice.带电/带磁/危险品/报关方式/清关方式/交税方式`
- `invoice.extend_to_fba_order_detail`（默认 **true**）——输出 55 列完整版发票，与货代系统示例一致；
  置 false 只出 40 列精简版（货代系统会缺必填的「预计到仓时间 / 所在货站」）
- `quote.origins`（默认 深圳/广州/中山/汕头/华南）——决定取哪套起运地价格
- `quote.max_transit_days`（默认 null 不限）——要卡时效就填天数

### 查领星发货计划（拿发货仓作兜底、反查货件号）

货件号 → 发货计划 → 发货仓库（＝提货地点）。走领星 MCP，三步：

```text
help(query="发货计划")                     # 找 fba_plan_group_list
action fba_plan_group_list                 # receive_warehouse_type="1"，search_value=批次号或SKU，
                                           # search_field="group_gmt_create" 放宽日期
                                           # 返回 plan_list[].list[] 每行有：
                                           #   sku / wid / wname（发货仓库）/ order_sn / isp_id / box_num
                                           #   以及 mws_relate[].shipment_mws_sn（FBA 货件号）
action fba_plan_item_detail order_sn=…     # 需要更细时再看 storage_list[].warehouse_name
```

**提货地点 = 该 SKU 所在计划行的 `wname`（发货仓库）**，不是按票一个值：

| wname 形态 | 含义 | 谁去提货 |
|---|---|---|
| 工厂直发-厨之乐 / 工厂直发-武航 … | 供应商直发仓（`wid=97/98`，`is_supplier=1`） | 货代去工厂提 |
| 中山青吉仓库（`wid=1`） | 公司的备货仓 | 货代去仓库提 |

**一票里经常同时有两种**（例如 WP77-2/WP103A-3/WP72-1 走「工厂直发-厨之乐」，
WP650A-5-V1 走「中山青吉仓库」），所以必须逐 SKU 取，不能用整票一个值，也不能按整批一个值。

把领星返回的 JSON 原样存盘，传给 `--warehouse-map`，再用 `--plan-seq` 锁批次，避免同 SKU
被别的计划的发货仓库覆盖。型号库「发票型号汇总」的「工厂」列降级为兜底（老数据没删），
`config.carrier.提货地点` 是最后兜底。

### 2. 箱唛 → 装箱单（必跑，别跳过）

```bash
python scripts/labels_to_packing.py \
  --labels "箱唛/" \
  --shipments "领星货件_2026-11-11.json" \
  --out "装箱单_2026-11-11.xlsx" --json "装箱单_2026-11-11.json"
```

- 每页一箱，解析 `FBA Box 3 of 7 - 7.3kg` / `Single SKU / JS-WP535C-1-V1 / Qty 40` / 箱号 `P14559-B03` / FNSKU。
- **MSKU → 主 SKU 靠 FNSKU 桥接**（箱唛印的是 `JS-WP535C-1-V1`、`BS-127` 这类 MSKU，
  只有 FNSKU 能可靠对上 `WP535C-1-V1` / `WP165A-1`），所以务必同时给 `--shipments`。
- 外箱尺寸取 `assets/产品箱规.csv`，缺失/为 0 会列进 problems（尺寸错→CBM 和体积重全错，不许猜）。
- 打印每票箱数/件数/毛重，与 `get_fba_shipment_list` 的 `total_quantity_shipped` 对一眼即可。
- 箱唛未下、用户要先看计划：直接跳过本步，按领星计划生成粗装箱单，并在汇报里标明「缺箱号，非最终版」。

### 3. 跑生成脚本

```bash
export PYTHONUTF8=1

python scripts/build_plan.py \
  --packing "装箱单_2026-11-11.xlsx" \
  --shipments "领星货件_2026-11-11.json" \
  --quote "恒信（辛巴）海运报价-2026-11-02.xlsx" \
  --template "发货模板.xlsx" \
  --warehouse-map "领星计划_RP261023007.json" --plan-seq RP261023007 \
  --station "中山集散中心" --arrival 2026-11-05 \
  --out "./输出"
```

`--shipments` 认三种：**领星货件 JSON（推荐）**、亚马逊后台 xlsx、同结构 csv；
**可以不给**——不给就从 `--warehouse-map` 的 `mws_relate` 兜底（但拿不到 PO）。
`--quote` 只传一家；货代名按 `config.carrier_aliases` 从文件名识别。
`--template` 同时提供地址库/型号库/服务类型，也可用 `--address/--models/--services` 单独指定。
表头认不出时用 `--packing-sheet/--shipment-sheet` 指定工作表名。

发票必备参数（漏了货代系统传不上去）：

| 参数 | 含义 | 取值 |
|---|---|---|
| `--shipments` | 货件来源（FBA 号/PO/仓库/地址） | 领星 `get_fba_shipment_list` JSON 最全；xlsx/csv 亦可 |
| `--ref-id` | PO Number **兜底** | 平常不用给（JSON 里有 `reference_id`）；只有走 `mws_relate` 兜底时才需要 `FBA=参考号` |
| `--warehouse-map` | 按 SKU 定提货地点 | 领星 `fba_plan_group_list` 原始 JSON；也支持 `SKU=发货仓库` 文本或 CSV |
| `--plan-seq` | 只认本批计划 | 批次号 `RP261023007` 或计划单号 |
| `--pickup` | 提货地点**兜底值** | 领星没匹配到、型号库也没填工厂时才用 |
| `--station` | 所在货站 | 默认 中山集散中心 |
| `--arrival` | 预计到仓时间 | 默认当天 |
| `--customs-lib` | 报关要素库 | 默认 `assets/报关要素.csv` |
| `--service-codes` | 服务代码白名单 | 默认 `assets/service_codes.csv` |
| `--box-spec` | 领星主数据箱规 | 默认 `assets/产品箱规.csv` |

产出：`输出/发货计划_2026-11-04.xlsx`（出货明细 / 发票 / 仓库对接表 / 校验报告）+ 同名 `.json`（含每票中选渠道、备选渠道与合计运费）。
文件名按**当天日期**生成，同日重复执行自动加 `_2`、`_3`，不会覆盖上一份。

### 4. 处理校验报告

先看同名 `.json` 的 `issues`：

- **错误** 必须解决后重跑：地址库缺仓库（且领星货件里也拿不到地址）、货件来源里没有该 FBA 号、
  该货代查不到此仓含税价、**服务代码不在白名单**（会报「服务不存在」，脚本已给最接近的候选）、
  **报关要素缺英文报关名/HSCODE**（不编造，去 `assets/报关要素.csv` 补）。
- **警告** 逐条确认：数量与后台不符、箱号与箱数不符、型号库缺 SKU、**装箱单单箱数量与领星主数据箱规不一致**、
  缺 Reference ID（走 `mws_relate` 兜底且没给 `--ref-id`）、地址来自领星货件需核对收件人。
- **提示** 带进汇报：渠道无标准服务代码、同货代次优渠道差 <2%、箱体尺寸与主数据差 >1.5cm。

不允许把带"错误"的结果交给用户当成品。

### 5. 抽检输出

按 `references/field-mapping.md` 抽查：

- 发票行数 = Σ每票 SKU 数（10.22 批次 = 25 行）；`总箱数` 每票一致；`货箱编号` 折叠正确。
- 仓库对接表 `序号` 连续，`箱子编号` 每票从 01 重新开始，标题里的货代 = 本次货代。
- 出货明细每票都有渠道/单价/时效；`合计` 与逐票运费相加一致。

### 6. 发群通知（可选，箱唛 + 贴标备注 + 两份附件）

用户要「把箱唛和贴标备注发到工厂发货对接群」时才做，**默认不生成不发**。

发送形态与工厂群现有习惯一致：**一票一条文字，紧跟该票的 FBA箱唛 PDF + 货代唛头 PDF**。

**按提货点拆批发（`notice.split_by_pickup`，默认开）**：一批货里同时有「工厂直发-XX」和「中山青吉仓库」时，
不要混成整批一条条发，而是**按提货点分成多个批次，一批发完再发下一批，批内仍是一票一条+紧跟附件**。
同一票会在这两个批次各出现一次，小标题保留 `店铺第N票FBAxxx共X件(发XX物流)`，其中**件数写本批次的箱数**
（第1票厨之乐那批 3 件、中吉那批 3 件，合计整票 6 件）；附件在该批内各带一份（同一 PDF 只上传一次）。
不要拆时加 `--no-split-by-pickup`。

```bash
# 第一步只生成预览，不发（--labels 指向箱唛/唛头 PDF 目录，可给多个）
# 恒信唛头只有 PT 号时再加 --declaration "申报信息_日期.xlsx"
python scripts/build_plan.py ... --notice --labels "D:/发货资料/唛头" \
  --declaration "D:/发货资料/申报信息2026-11-04.xlsx" --out "./输出"

# 想要「少条消息、方便合并转发」时，再压成一批 2 条（1 条总清单 + 1 个合并 PDF）
python scripts/make_bundle.py --manifest "输出/..._通知.json" \
  --out-dir "输出/合并版" --out-manifest "输出/..._通知_合并.json"

# --notice-send 只是「生成 + 预览」，不会真的发出去（防误发）
python scripts/build_plan.py ... --labels "D:/发货资料/唛头" \
  --notice-send --notice-group "物流发货测试群" --out "./输出"
```

产出两个文件：`..._通知.md`（整批汇总，含附件对照清单）和 `..._通知.json`（分票清单，发送用）。

**`make_bundle.py` 的作用**：企业微信没有「合并转发」的发送接口，能做的就是**把消息条数压到最少**。
它把一批 5 票的「5 条文字 + 10 个附件」压成「1 条总清单 + 1 个合并 PDF」（封面是清单表，后面按票序排列，
带书签可跳票），转发的人多选 2 条即可「合并转发」，或者只转那个 PDF 也完整。
合并时**按箱号截页**：箱唛 PDF 是按整票 6 箱出的，厨之乐那半批只会保留 01～03 箱的页，不会把中吉的 04～06 箱也打进去；
页数与箱号对不上时退回「保留全部页」并打印提示。

单独发送 / 补发：`python scripts/send_notice.py --manifest "输出/..._通知.json" --group "群名"`（预览，末尾给确认码），
确认后 `--send --confirm <码>`；只想发整段文字、不带附件时用 `--file "输出/..._通知.md"`。

**发出去撤不回，所以两个发送脚本都带确认码闸门（硬规则）**：`--send` 必须配 `--confirm`，
码是「目标群 + 每条文字 + 每个附件的文件名和大小」的哈希，清单改过、换群、换附件都会变。
预览时会在清单同目录写下 `_确认码.json`，真发还要求**目标一致且未过期**（默认 30 分钟，`--confirm-ttl` 可调）；
过期、换群、改清单都要求重新预览。

正确顺序永远是：**跑预览 → 把批次/条数/确认码贴给用户 → 等用户明确说「发」→ 才带码发送**。

**「什么时候发」由用户定，不由脚本或 AI 定。** 群里可能刚好有人在发同一条消息，撞上就重复；
这种实时状态 AI 判断不了——webhook 是单向的看不到群消息，智能机器人也拿不到准确的在途消息。
所以不排队、不重试、不定时，等人开口再发。

三条禁令：

1. 不许替用户默认同意，不许自己补确认码；
2. 不许「先发测试群就当已确认」——测试群同样撤不回；
3. **不许为了验证闸门或试脚本而往任何真实群发**。要验证就单测 `gate.check()`，或用不存在的群名让它中途失败。

**附件必须与文字成组、连续发送（硬规则）**：群里要的是「一条文字 + 该票两份附件」紧挨着的一组，5 票就是连续的 15 条消息；
分两个提货点批次时就是两组 15 条，批次之间不插话、不补发。
所以发现漏发/发错时，**整批重发**，不要用补发模式去「打补丁」——补发会把附件甩到文案后面，破坏一票一组的连贯性。

```bash
# 整批重发（推荐）：5 条文字 + 10 个附件，逐票成组连续
python scripts/send_notice.py --manifest "输出/..._通知.json" --group "群名"
python scripts/send_notice.py --manifest "输出/..._通知.json" --group "群名" --send --confirm A1B2C3
```

只有当用户**明确说**「文字不用再发、只补附件」时，才用：

```bash
python scripts/send_notice.py --manifest "输出/..._通知.json" \
  --group "群名" --attachments-only --kind fba_label --send
```

**FBA 箱唛在 `STA_*.zip` 里，不要漏**：亚马逊后台导出的是压缩包，里面每票一份 `FBA_STA_(日期)-仓库-货件号-店铺-US-时间.pdf`；货代唛头是另一批 `PT*.pdf`。两批都要传给 `--labels`，缺一批会导致该批附件整体发不出去（校验报告会标「缺FBA箱唛」），最终只能整批重发。

**附件归票规则**（`scripts/attachments.py`）：先按文件名认类型（`FBA_STA`→箱唛、`-PT`→货代唛头、无特征→默认货代唛头），再按货件号认票（完整 → 前 8 位 → **PT 运单号** → 仓库代码）。缺口在「校验报告」记警告、该票只发文字；多余文件记提示，不静默丢弃。用户没给 `--labels` 时就只发文字，不要编造附件路径。

**恒信只给 PT 追踪码时**要加 `--declaration "申报信息_日期.xlsx"`：里面 `运单号`(PT) 与 `扩展运单号`(FBA) 一一对应，脚本据此把 PT 翻译成货件号；同时核对申报信息的仓库与本票是否一致（不一致记警告，必须人工确认）。

**硬性顺序：先在测试群跑通，再发正式群。** 正式群发出去撤不回。

1. 测试群把机器人拉进去并 @ 它发一句话（否则企业微信不给该会话发消息权限，会报「当前没有可发送的会话」）；
2. `send_notice.py --list` 确认群可见、群名逐字一致；
3. `--notice` 生成 → 审文案与附件归票（张数应等于票数×2）→ `--send` 发测试群 → 群里确认文字换行、附件能打开；
4. 一切正常后才对正式群执行。

文案模板在 `config.json` 的 `notice` 段，附件识别关键词在 `notice.attachments.kinds`，字段与排错见 `references/notice-template.md`。

#### 两种发送通道，按群的性质选

| 通道 | 前提 | 适用 |
|---|---|---|
| `send_notice.py`（企业微信智能机器人） | 机器人**必须是群成员**，还得在群里 @ 它一次 | 测试群、内部群 |
| `send_robot.py`（群机器人 webhook） | 群主在群里添加一个「群机器人」，只管给地址 | **工厂群 / 仓库群**，不用把 AI 拉进群 |

工厂群、仓库群一律用 `send_robot.py`：群机器人是**单向通道**，只能发不能收，读不到群消息、看不到成员、拿不到聊天记录，
随时在群设置里删掉就立刻失效；而且一条消息就是群里的一条，不用再「多选转发」，天然没有乱序问题。

配置在 `config.json`：

```json
"notice": {
  "robots": {
    "工厂直发-厨之乐": "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=aaa",
    "中山青吉仓库":   "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=bbb"
  },
  "robot_default": ""
}
```

键就是 `_通知.json` 里每条消息的 `batch`（= 提货点）。脚本按批次自动分流，一个批次推一个群，顺序与清单一致：

```bash
python scripts/send_robot.py --manifest "输出/..._通知.json"                     # 预览，末尾打印确认码
python scripts/send_robot.py --manifest "..." --send --confirm A1B2C3            # 真发（码要一致）
python scripts/send_robot.py --manifest "..." --batch "中山青吉仓库" --send --confirm A1B2C3
python scripts/send_robot.py --manifest "..." --webhook "https://..." --send --confirm A1B2C3
```

**发消息没有撤销按钮——必须走「预览 → 用户点头 → 带确认码发送」三步（硬规则）**：

1. 先跑不带 `--send` 的预览，把批次、目标群、条数、确认码完整贴给用户；
2. **等用户明确说「发」（或「确认」）之后**才执行发送；
3. 发送命令必须带 `--confirm <那个确认码>`，码由「批次 + 目标群 + 每条文字 + 每个附件的文件名与大小」哈希得出，
   清单改过、换了群、换了附件都会变码；对不上脚本直接拒绝，所以既不会「预览的是 A、发出去的是 B」，
   也不怕手滑多打一个 `--send`。

预览会在清单同目录留下 `_确认码.json`（含时间、目标、批次指纹），真发时还要过 `gate.check()`：
**未预览 → 拒；换群 → 拒；超过 `--confirm-ttl`（默认 30 分钟）→ 拒并要求重新预览**。
理由是「确认过内容」不等于「此刻适合发」——群里可能正在有人发同一条。

**禁止**：替用户默认同意、自行补 `--confirm`、先发给「测试群」就当已确认、或为了验证脚本/闸门而向任何真实群发送。
要验证闸门就单测 `gate.check()`，或用一个不存在的群名让它在中途失败。

群机器人的硬限制（超过就报错，脚本已内置处理）：

- 文字每条 ≤ 2048 字节，**普通文件 ≤ 20MB**，`media_id` 3 天有效**且与该 key 绑定**——同一份 PDF 发两个群要各传一次（脚本按群缓存）；
- 限速 **20 条/分钟**，脚本默认条间隔 1.2s，超了自动等到下一分钟；
- **发出去撤不回**：企业微信没有 webhook 消息的撤回接口，发出去只能靠群里人工处理，已读的没法收回。
  所以一律先预览、再让用户点头、最后带 `--confirm` 发送，见上面的三步硬规则。

### 6.5 发错了怎么处置（撤不回，只能作废）

企业微信对机器人消息**没有撤回接口**：删不掉、改不了、也置顶不了。所以发错之后不要想着「删掉重发」，
按这五步走，让工厂明确知道哪份作废：

1. **停手**。别急着补发或重发，先把错的是哪几条、错在哪想清楚。
2. **查发送日志**：清单同目录的 `_发送记录_YYYY-MM.jsonl`，每次真发都留了一条（时间、群/批次、
   确认码、条数、附件名、首行文字）。用它确认「群里现在那批是哪一版、几点发的」。
3. **推作废声明**，发到**同一个群**（作废个半批等于没作废）：

   ```bash
   python scripts/send_void.py --group "工厂直发-厨之乐" \
     --scope "巨杉美国第1-5票（2026-11-05）的唛头信息" --reason "提货地点写错"
   # 预览无误、用户确认后
   python scripts/send_void.py ... --send --confirm A1B2C3
   ```

4. **紧接着发更正版**：标题带「更正版」，附件文件名带版本（如 `_v2`），别和旧文件同名，
   否则工厂在群里看到两个同名 PDF 还是分不清。
5. **口头确认**：群里 @ 对应负责人，或在别的渠道同步一声，别只发一条消息就算完。

三个不要：不要用「补发附件」去掩盖错误（旧消息还在，越补越乱）；不要只发新版不说作废
（工厂很可能继续用旧的贴标）；不要假设「群里的人会自己看出来」。

降低发错概率的前置动作：预览 → 用户确认 → 带确认码发；能先发测试群就先发测试群；
校验报告里 `错误` 必须为 0 再进入发送环节。

### 7. 汇报

给一张按票的表：票 / 仓库 / 箱数 / 数量 / 计费重 / 渠道 / 单价 / 运费 / 时效，末尾给合计运费，再列"待办"。最后用 `present_files` 打开生成的 xlsx。

## 关键口径（易错，务必遵守）

- **只取含税列**：不含税列是元/CBM，不参与；只取 `含税` 的元/KG。
- **起运地取华南**：凯琦一个渠道组里有 深圳/义乌/厦门/青岛 四套价，中山发货只取「深圳\广州\中山\汕头」那套；恒信跳过「华东」列。
- **含税列可能分档**：`50KG+`/`51KG+`/`101KG+` 是同渠道不同最低计费重。**起收档不逐票套用**（公司整批累计可满足，config `quote.use_min_weight=false`），计费重 = MAX(实重, 体积重)，按 0.5KG 向上进位；只有把该开关设 `true` 时才会取到起收重量。
- **选渠道只在同一货代内部比价**：同仓多条渠道取运费最低；差 <2% 时提示人工复核，不静默改单。
- **第 N 票** = 货件号在箱唛/装箱单中**首次出现的顺序**，不是号码大小、不是后台列表顺序。
- **发票出 55 列完整版**（与货代系统示例 `FBA订单明细` 逐列对齐），必填列（带 `*`）一个都不能空：
  `PO Number 货件跟踪编码` = **Amazon Reference ID**（不是 PT 单号）、`预计到仓时间*`、`所在货站*`。
  40 列精简版货代系统不收，`invoice.extend_to_fba_order_detail` 保持 `true`。
- **一手货件数据只认领星 `get_fba_shipment_list`**：FBA 号、PO（`reference_id`）、仓库代码、地址、申报量
  全从它取，不再靠亚马逊后台导出或手抄截图；亚马逊导出只在没 JSON 时兼容。
- **发票「服务*」必须在服务代码白名单里**：`assets/service_codes.csv` 来自货代发票模板的「服务类型列表」，
  渠道代码对不上就本地报错（历史踩坑：`US-TRKP` 上传第 28 行才报「服务不存在」，正确是 `US-TRK`）。
- **报关要素以 `assets/报关要素.csv` 为准**（源：`领星产品材质、报关编码确认版.xlsx`）；
  英文报关名/HSCODE 为空时报错让人补，**不许编造**。
- **提货地点 = 领星发货计划里该 SKU 的「发货仓库」（计划行 `wname`），逐 SKU 取**：`工厂直发-XX` 去工厂提、`中山青吉仓库` 去仓库提，同一票常混装。型号库「工厂」列只是老数据兜底。群通知每行要标 `（XX贴）`。
- **发票一票×一 SKU 一行**；`总箱数` 填该票合计，`产品申报数量` 填**单箱数量**。
- **货箱编号折叠**：`FBA19V04K1GYU000001-6`（结束号不补零，单箱不折叠）。
- **唛头每票从 01 重排**，`箱子编号` 结尾带全角分号 `；`。
- **中文品名取型号库"中文品名"（肥皂盒）**；**出货明细"产品名称"取型号库"产品名称"（四钩肥皂架本色）**——两者不同，别混。
- 货件标题括号是 `(发恒信物流）`——半角左括号 + 全角右括号；模板见 `title_template`。
- 托盘位（`P1 - B14～B15`）只是找箱用，不进任何输出。
- 群通知与 `仓库对接表` **同源**（同一份 `mark_text` 与 SKU 件数）：分票发送时每票一条，唛头每票从 01 重排；`..._通知.md` 汇总版才是跨票连续 1~N。不要另起一套编号。

细节判定、字段映射、报价规则分别见：

- `references/business-rules.md`——票序、箱号、唛头、校验口径
- `references/field-mapping.md`——三张表逐列来源
- `references/input-schemas.md`——四类输入的表头要求与降级策略
- `references/quote-rules.md`——报价表结构、计费重、单货代选渠道与验证基线
- `references/notice-template.md`——群通知文案模板、附件归票规则、测试流程与排错
- `使用说明.md`——面向使用者的输入/输出清单与参数对照（可直接发给同事）

## 脚本约定

- 只依赖 `openpyxl`，无 pandas/PyYAML；CSV 报价表 UTF-8/GBK 自适应。
- `scripts/labels_to_packing.py`：**FBA 箱唛 PDF → 装箱单**（`--labels 目录 --shipments 领星货件.json --out xlsx`）。
  每页一箱；MSKU→主 SKU 用 FNSKU 桥接；尺寸查 `assets/产品箱规.csv`，缺失报错不猜。
  `--strict` 有任何问题即非 0 退出；`--json` 出结构化对账文件。**别再用一次性脚本手写解析**。
- `scripts/build_refdata.py`：从领星导出表重建三份本地参照库（报关要素 / 产品箱规 / 服务代码），
  原始表更新后重跑，**不要手改 CSV**。
  ⚠️ 例外：箱唛实测过的箱规（单箱数量/重量/外箱尺寸）可回写 `产品箱规.csv`，
  一旦回写就在 git 里留一句来源（「2026-11-11 箱唛实测」），因为重跑 `build_refdata.py` 会被领星旧数据覆盖。
- `scripts/make_skill_package.py`：把两个技能打成可分发 zip（排除 `__pycache__` / `assets/_backup` /
  `assets/links.json`，保留 `config.json` 与三份参照库）。发版后覆盖上传同一 `file_token`，下载链接不变。
- `scripts/sync_vault.py`：把 SKILL.md 同步成 Obsidian 库里的技能笔记（`30 Skills 技能库/本地Skills/`），
  技能改完跑一次，笔记别手改正文。
- `scripts/sync_feishu.py`：把 `使用说明.md` 覆盖到飞书在线文档（地址在 `assets/links.json`），
  改完使用说明跑一次，同事打开的就是最新版。
- 报价解析在 `scripts/quote_parser.py`：按「渠道类型行（含税/不含税）+ 起收档位行 + 仓库代码列」定位区块，恒信竖排/凯琦横排都能吃；结构不匹配时退回通用表头解析。
- 选渠道在 `Builder.select_channel`：只做同货代内比价，**不要**在这里加跨货代逻辑，那是 `lingxing-fba-carrier-compare` 的职责。
- 修改口径优先改 `config.json`；改代码前先读 `quote_parser.parse_sheet` 与 `quoting.calc_charge`。


---

← 返回 [[00 本地Skills总览]]
