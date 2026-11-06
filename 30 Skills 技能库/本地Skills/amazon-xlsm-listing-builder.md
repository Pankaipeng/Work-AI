---
title: amazon-xlsm-listing-builder
type: skill
group: Listing 文案
source_level: 自建
upstream: —（无上游，本地原创）
local_path: C:/Users/Administrator/.workbuddy/skills/amazon-xlsm-listing-builder/
updated: 2026-11-05
tags: [skill, workbuddy, 自建, amazon, 我的skill]
---

# amazon-xlsm-listing-builder

> [!tip] 🟢 自建（我个人创建）
> 把定稿文案 + 商品资料（可自动提取领星 ERP 导出表）填进亚马逊官方批量上架 .xlsm，产出能直接 Add Products via Upload 的文件，并解析回执报错。

> **判断依据**：SKILL.md frontmatter 标记 `agent_created: true`，且含本机专属的领星字段映射与实战报错对照。
>

## 速览

| 项 | 内容 |
|---|---|
| 来源 | **🟢 自建（我个人创建）** |
| 上游 | —（无上游，本地原创） |
| 分组 | Listing 文案 |
| 调用 | `@amazon-xlsm-listing-builder` |
| 内部标题 | 亚马逊批量上架表生成（资料 + 文案 → 可上传 xlsm） |
| 本地路径 | `C:/Users/Administrator/.workbuddy/skills/amazon-xlsm-listing-builder/` |
| SKILL.md | 309 行 |
| 最后更新 | 2026-11-05 |

## 触发场景

填上架模板、生成上架表、批量上传文件、flat file、父子变体合并、processing summary 复盘

## 飞书链接

| 用途 | 链接 |
|---|---|
| USAGE.md 在线版（可直接发同事） | https://bqnscp788os.feishu.cn/docx/SgQSd3tFXoD6i2xvMQzcmLVenFi |

## 内容结构

- **铁律**
- **新建 SKU 的隐藏必填（最容易翻车的一条）**
- **工作流**
  - 第 0 步：确认模板
  - 第 1 步：解析模板结构
  - 第 2 步：汇总商品资料
  - 第 3 步：补文案
  - 第 4 步：文案合规校验
  - 第 5 步：试映射（不写文件）
  - 第 6 步：写入并校验
  - 第 7 步：上传后拿回执复检
  - 第 8 步：交付
- **变体（父子）**
- **发货方式：FBA 与品牌豁免**
- **命令速查**
- **常见坑**
- **文件说明**

## 附带文件

- `USAGE.md`
- `references/amazon-required-fields.md`
- `references/copy-rules.md`
- `references/field-mapping.md`
- `references/input-schema.md`
- `references/learned-requirements.json`
- `references/template-structure.md`
- `references/variants-and-quality.md`
- `scripts/amazon_template.py`
- `scripts/erp_import.py`

## 全文（SKILL.md 原文）

> 原 frontmatter 已剥离，正文原样归档，便于全文搜索。

# 亚马逊批量上架表生成（资料 + 文案 → 可上传 xlsm）

把商品资料和 listing 文案落到亚马逊官方批量上架模板里，产出能直接被卖家后台接受的 .xlsm；上传后拿回执再复盘，形成闭环。

核心思路：**模板是唯一事实来源**。字段列位置、必填级别、下拉允许值、逐行必填项，全部从用户提供的那个 .xlsm 里动态解析，不靠记忆猜列号——亚马逊每个品类、每次改版都会变。

面向人的操作手册见 `USAGE.md`——**按亚马逊运营（非技术）视角写**：准备什么、怎么开口、拿到文件怎么传、报错怎么处理，命令集中放在文末附录。本文只写 agent 执行时需要的判断依据。

---

## 铁律

1. **绝不改用户的原文件**。一律另存新文件，脚本会拒绝 `--out` 与源文件同路径。
2. **先 inspect 再动手**。没跑过 `inspect` 就写数据，等于在猜列号。
3. **不编造商品信息**。SKU、UPC/EAN、价格、库存、图片 URL 必须来自用户或 ERP；缺了就列清单问，不填占位符。
4. **写入前必须过 lint + dry-run**。文案超限、枚举值非法、必填缺失，都要在写文件之前暴露。
5. **「模板 Required」不等于「亚马逊必填」**。新建 SKU（`action=Create`）时，模板标 `Conditionally Required` 的产品属性会被亚马逊按必填校验，缺一个就 90220、整行上架失败。详见下节。
6. **必填字段必须逐行写满**。模板只预填第一行，第 2 行起不会自动带（`fill` 会自动继承，但要看 `inherited_from_template_row`）。
7. **变体先确认 `variation_theme` 的合法值**。是 `COLOR` 不是 `Color`，猜必错。

---

## 新建 SKU 的隐藏必填（最容易翻车的一条）

模板 `Data Definitions` 把大量产品属性标成 `Conditionally Required`。实测结论：

| 动作 | 条件必填的处理 |
|---|---|
| `Create or Replace (Full Update)`（新建） | **按必填校验**，缺一个就 90220，整行失败 |
| `Edit (Partial Update)`（已有 ASIN） | 可省，只影响属性完整度 |

已被实测证实会阻断上架的字段（`KITCHEN` 品类，2026-11-05）：

| 字段 | 列 | 合法值 / 填法 |
|---|---|---|
| `model_name` | Q | 型号名，可与 `model_number` 相同 |
| `size` | AT | 尺寸描述，如 `8.5 x 3.3 x 3.3 inches` 或 `One Size` |
| `care_instructions` | AV/AW | **枚举**：`Hand Wash` / `Dishwasher Safe` / `Microwave Safe` |
| `number_of_items` | AQ | 销售单元件数，单品填 1 |
| `list_price` | BV | 标价（划线价），必须 ≥ 售价；和 `our_price`(CY) 是两个字段 |
| `contains_liquid_contents` | FV | **枚举**：`Yes` / `No`，填 No 就不需要 `liquid_volume` |

`fill` 已内置这套判定，`missing_for_new_sku` 非空即代表**上传必挂**；`upload_ready: false` 不要交付。
`report --learn` 还能把回执里学到的字段按 `product_type` 写进 `references/learned-requirements.json`，下次自动当必填拦。

---

## 工作流

### 第 0 步：确认模板

必须拿到**该品类**对应的 `.xlsm`（一个品类一份，不能混用）。确认三件事：品类对不对、marketplace 是不是目标站点、店铺预填（Preference Profile）在不在。

### 第 1 步：解析模板结构

```bash
python scripts/amazon_template.py inspect --template "<模板.xlsm>"
```

关键返回值：

| 字段 | 用途 |
|---|---|
| `layout.field_row` / `data_start_row` | 技术键行、数据起始行 |
| `copy_fields` | 文案相关字段的列与必填级别 |
| `meta.marketplace` / `language_tag` | 站点与语言 |
| `prefilled_row` | 已预填的值（品类、品牌、材质…），**只作用于第一行** |

### 第 2 步：汇总商品资料

**来源 A：ERP 导出表（推荐）**

```bash
python scripts/erp_import.py --erp "导出产品.xlsx" --sheet 产品 \
  --prefix "JS-" --parent-sku "JS-WP650A-PARENT" --variation-theme COLOR \
  --color-map "黑色=Black,白色=White,透明=Clear" \
  --asin-map "WP650A-5-V1=B0HNZWSKBM" \
  --brand "Nieifi" --item-type-keyword "under-sink-organizers" \
  --price 15.99 --fulfillment AMAZON_NA --gtin-exempt --skip-images \
  --care-instructions "Hand Wash" \
  --out listing.json
```

自动完成：SKU 加店铺前缀、识别颜色、材质归一、型号/包装尺寸重量提取与 **cm→英寸 / g→磅换算**、主图、父子结构、已上架 SKU 打 `Edit (Partial Update)` + ASIN、**补全新 SKU 的隐藏必填**（model_name / size / number_of_items / list_price / care_instructions / contains_liquid_contents）、主图重复告警、未完成字段写进 `_todo`。

业务参数：

| 参数 | 作用 |
|---|---|
| `--price 15.99` | 统一售价 |
| `--list-price 19.99` | 标价（划线价），不给则等于售价 |
| `--fulfillment AMAZON_NA` | 语义值即可（FBA）；`fill` 会自动转成模板要的 `Fulfillment by Amazon (NA)`。FBM 用 `MFN` |
| `--gtin-exempt` | 品牌豁免：无 ASIN 的子体填 `GTIN Exempt`，Product Id 留空 |
| `--skip-images` | 没有图片素材时不写主图，避免多色共用一张错图 |
| `--care-instructions` | 三选一：Hand Wash / Dishwasher Safe / Microwave Safe |
| `--contains-liquid` | Yes / No，默认 No |
| `--number-of-items` | 默认 1 |
| `--size` | 不给则用包装尺寸拼 |

**来源 B：用户口头/文档给的信息**

直接手写 JSON，结构见 `references/input-schema.md`，可抄 `assets/example-listing.json`。

### 第 3 步：补文案

文案到位后合并（按 SKU 对齐，只补空字段、不覆盖已有值）：

```bash
python scripts/erp_import.py --erp "导出产品.xlsx" ... --copy copy.json --out listing.json
```

同样可用 `amazon-listing-builder` 先出标题/五点/描述/ST，再合并进来。

**文案包怎么写**（一个 SKU 一条，字段名同 `references/input-schema.md`）：

```json
{"products": [
  {"sku": "JS-WP650A-PARENT", "title": "系列标题（不含颜色）"},
  {"sku": "JS-WP650A-5-V1", "_overwrite": ["material_1"],
   "material_1": "Polycarbonate", "title": "...", "bullet_1": "..."}
]}
```

- **只补空、不覆盖**是默认行为；要修正 ERP 的错误值（如材质）时，把键名写进该条的 `_overwrite`，覆盖动作会记进 `_erp.notes`。
- **颜色变体不能共用一套文案**。第五点和外观描述必须按颜色改写：透明款的 "transparent PC" 套到黑/白款上就是错误描述。
- **父体也要给 description 和五点**。这两列是 `Required`，父体缺了会被点名，有整条 feed 被拒的风险；用不含颜色细节的中性文案填上（父体不展示给买家）。

### 第 4 步：文案合规校验

```bash
python scripts/amazon_template.py lint --data listing.json
```

`errors > 0` 必须清零。检查：标题长度与禁售字符、五点长度、ST 字节数、促销词、联系方式、emoji、占位符、父子结构完整性，**以及新建 SKU 的隐藏必填**。规则见 `references/copy-rules.md`。

### 第 5 步：试映射（不写文件）

```bash
python scripts/amazon_template.py fill --template T.xlsm --data listing.json --dry-run --fix-enums
```

看五项：`writes` 数量、`unmapped_fields` 必须空、`enum_warnings` 必须空、`missing_for_new_sku` 必须空、**`upload_ready` 必须为 true**。

### 第 6 步：写入并校验

```bash
python scripts/amazon_template.py fill --template T.xlsm --data listing.json --purpose "FBA上架" --fix-enums --strict
```

**不传 `--out` 时按 `日期_商品_用途.xlsm` 自动命名**，例如 `2026-11-05_JS-WP650A_FBA上架.xlsm`，重名自动加 `-2`。命名规范是硬要求：日期在前、SKU 在中间、用途在最后，别人扫一眼目录就知道是新是旧、改的什么。

报告读法：

| 字段 | 说明 |
|---|---|
| `upload_ready` | **唯一交付判据**：`ok && 无 missing_required && 无 missing_for_new_sku` |
| `ok` | 产物本身是否校验通过（回读一致、VBA 保留等） |
| `rows_written` / `cells_written` | 写了多少行 / 格 |
| `inherited_from_template_row` | 从模板第一行继承的字段（品类/品牌/原产国/危险品） |
| `missing_required` | `Required` 缺项，必须清零 |
| `missing_for_new_sku` | **新建 SKU 的条件必填缺项，不清零就别传**（带 `suggestion` 填法） |
| `missing_conditional` | 其他条件必填，逐条过一遍（影响 IDQ） |
| `enum_warnings` | 仍不匹配的枚举值，需人工判断 |

### 第 7 步：上传后拿回执复检

用户把后台返回的 `<文件名>-processing-summary.xlsm` 发回来时：

```bash
python scripts/amazon_template.py report \
  --report "xxx-processing-summary.xlsm" \
  --template "T.xlsm" \
  --learn --product-type KITCHEN \
  --patch patch.json
```

做四件事：读出每个 SKU 的提交状态、把报错字段名映射回技术键与列号、按字段聚合并给出填法建议、把字段记进学习库（`--learn`）并导出待补骨架（`--patch`）。

回执里 `::submission_status` / `::number_of_attributes_with_errors` 两列是亚马逊回写的，比 `Feed Processing Summary` 表格更直接。

### 第 8 步：交付

告诉用户：产物路径、行数与变体结构、`upload_ready` 结论、还缺哪些字段（按 `missing_for_new_sku` / `missing_conditional` / `_todo`）、上传入口、无图导致不会 Active 的风险。首次建议 ≤15 行试水。

---

## 变体（父子）

三行结构，缺一个字段就建不起来：

```
父体: sku + parentage=Parent + variation_theme + brand        （不写价格/库存/图片/UPC）
子体: sku + parentage=Child + parent_sku + variation_theme + 颜色值 + 价格 + 库存 + 主图 + UPC/ASIN
```

- `variation_theme` 是大写 slug：`COLOR`、`SIZE/COLOR`、`COLOR/NUMBER_OF_ITEMS`…**不是** `Color`
- 已上架的子体用 `action=Edit (Partial Update)` + `product_id_type=ASIN`
- 每个颜色必须有自己的主图，三色共用一张是硬伤
- SKU 约定：`店铺前缀-工厂款号`，父体加 `-PARENT`

细节与 IDQ 字段对照见 `references/variants-and-quality.md`。

---

## 发货方式：FBA 与品牌豁免

**⚠️ 最容易静默失败的一列：`fulfillment_channel_code`（CT）**

模板下拉里写的是**展示值**，不是 SP-API 的机器码：

| 想要的渠道 | 模板里必须写的值 | 写了会失败的错值 |
|---|---|---|
| FBA | `Fulfillment by Amazon (NA)` | `AMAZON_NA` ← 2026-11-06 实证踩坑 |
| FBM | `Fulfillment by Merchant (Default)` | `DEFAULT` / `MFN` |

填 `AMAZON_NA` 时回执照样 `SUCCESS`、不报错，但亚马逊识别不了该值，**按默认落成卖家自配送**，
表现为「上架成功了，但是 FBM 链接」，要手工去后台转 FBA。

判断方法：读 `Dropdown Lists` 表该字段的真实清单，**不要照抄 API 文档**。
`amazon_template.py` 已对此做语义归一（`_ENUM_SEMANTIC`，无条件生效），输出里的
`value_coercions` 会写明 `AMAZON_NA → Fulfillment by Amazon (NA)`。见到这条不用惊讶，是正常的。

**FBA 必填的几项**（不是模板的 Required，但缺了要么建不出 offer，要么上不了架）：

| 列 | 字段 | 值 |
|---|---|---|
| CT | `fulfillment_channel_code` | `Fulfillment by Amazon (NA)`（EU 站为 `(EU)`，照模板下拉写） |
| EE–EJ | `item_package_dimensions` | 包装三维 + 单位，**FBA 算费依据，必须准** |
| EK–EL | `item_package_weight` | 包装重量 + 单位，同上 |
| CY | `purchasable_offer…our_price` | 售价，纯数字 |
| BV | `list_price` | 标价，必须 ≥ 售价 |
| BT | `condition_type` | `New` |
| AR | `item_package_quantity` | 1 |

**FBA 不要填的**：`quantity`（库存由入库单驱动；官方明确「quantity 有值会转成卖家自配送」）、`lead_time_to_ship_max_days`（那是 FBM 的备货时间）、`skip_offer`（填 Yes 就不建 offer 了）。

**降级判断**：即使 CT 写对了，新 SKU 没有 FBA 库存时前台仍不可售，这正常；
但后台「管理库存」的配送方式应显示亚马逊物流。若显示卖家自配送，就是 CT 值没被识别。

**品牌豁免（无 UPC）**：`product_id_type` 填 `GTIN Exempt`，`product_id` 留空。已有 ASIN 的老链接仍用 `product_id_type=ASIN` + ASIN 值，不要改成豁免。`lint` 见到 `GTIN Exempt` / `ASIN` 就不再报缺 UPC。

**没有图片**：`main_product_image_locator` 不是模板 Required，可以留空上传，但**listing 不会变 Active**，后台会显示缺主图。这种情况要提前跟用户说清楚，别让用户以为传完就上架了。

---

## 命令速查

```bash
S=scripts/amazon_template.py

python $S inspect --template T.xlsm [--full] [--out map.json]
python $S lint    --data D.json [--title-max 200] [--strict]
python $S fill    --template T.xlsm --data D.json [--purpose FBA上架] [--dry-run] [--fix-enums] [--strict]
python $S report  --report P.xlsm [--template T.xlsm] [--learn --product-type KITCHEN] [--patch patch.json]

python scripts/erp_import.py --erp E.xlsx --prefix JS- --parent-sku P --out D.json \
  [--copy C.json] [--price 15.99] [--list-price 19.99] [--fulfillment AMAZON_NA] \
  [--gtin-exempt] [--skip-images] [--care-instructions "Hand Wash"] [--size "One Size"]
```

`fill` 可选：`--sheet`、`--row`（覆盖数据起始行）、`--field-row`、`--engine openpyxl|com`、`--out`（默认自动命名）。

---

## 常见坑

- **模板 Required 不是全貌**。新建 SKU 时条件必填会变必填，漏填直接 90220。这类字段模板里分散在几处，光看 `Required` 一列看不出来。
- **枚举值必须逐字匹配**。`Create or Replace` 无效（要 `Create or Replace (Full Update)`）；`Color` 无效（要 `COLOR`）。靠 `--fix-enums` 和 `enum_warnings` 确认。
- **下拉里显示什么就填什么，别翻译成机器码**。曾经以为"下拉显示人话、上传要机器码"，于是把 FBA 渠道填成 `AMAZON_NA`，结果回执全 SUCCESS 但商品变成卖家自配送。实测三组：

  | 列 | 误填（会静默失败） | 正解 |
  |---|---|---|
  | Listing Action | `Create or Replace` | `Create or Replace (Full Update)` |
  | Fulfillment Channel | `AMAZON_NA` | `Fulfillment by Amazon (NA)` ← 2026-11-06 踩坑 |
  | Variation Theme | `Color` | `COLOR`（全大写 slug） |

  判断方法：**读该列下拉的真实清单**（`Dropdown Lists` 表，via `defined_names`），不要照抄 SP-API 文档。
  选项带 `(Default)` 前缀时取括号后的主体。`amazon_template.py` 的 `_ENUM_SEMANTIC` 已对渠道列做机器码→展示值归一。
- **`list_price` 不是售价**。售价是 `purchasable_offer…our_price`（CY），标价是 `list_price`（BV），两个都要填，且标价 ≥ 售价。
- **材质列是枚举，没有 ABS 也没有 PC**。只有 `Plastic` / `Polycarbonate` / `Silicone` / `Stainless Steel` 这类写法，`erp_import.py` 已内置归一。
- **保养说明也是枚举**。只有 `Hand Wash` / `Dishwasher Safe` / `Microwave Safe`，不是自由文本。
- **Search Terms 别只用一半**。US 站点上限 250 bytes，只用 90 bytes 等于白扔流量位。填到 200–240 bytes，且不要再放标题里已有的词（重复词不增加索引，只浪费字节）。
- **模板预填只作用第一行**。`product_type` / `brand` / `country_of_origin` / `supplier_declared_dg_hz_regulation` 都是逐行必填，第 2 行起模板不给。`fill` 会自动从第一行继承，报告里看 `inherited_from_template_row`。
- **父体别带 offer 字段**。价格/库存/图片会让父体变成幽灵 offer。
- **一行一个产品**，别插入/删除行，会破坏下拉校验区域。
- **单位**。ERP 多为 cm/g，US 模板要英寸/磅，且单位是独立列。`erp_import.py` 已换算，手写数据注意别把单位塞进数值。
- **图片引擎差异**：`openpyxl` 丢模板说明图（不影响上传），保留 VBA / 数据校验 / settings。要 100% 保真用 `--engine com`（需 Excel + `pip install pywin32`）。
- **一个品类一份模板**，Kitchen 的模板不能拿去做 Garden。

---

## 文件说明

| 文件 | 内容 |
|---|---|
| `USAGE.md` | 面向亚马逊运营的使用说明（准备三样东西 → 一句话开口 → 看 upload_ready → 上传 → 报错丢回执），命令参考在附录。在线版：https://bqnscp788os.feishu.cn/docx/SgQSd3tFXoD6i2xvMQzcmLVenFi |
| `scripts/amazon_template.py` | inspect / lint / fill / report 主工具 |
| `scripts/erp_import.py` | 领星 ERP 导出 → 上架 JSON 骨架 |
| `references/amazon-required-fields.md` | 亚马逊实际必填字段清单与报错对照（实战沉淀） |
| `references/template-structure.md` | 模板行结构、逐行必填、枚举陷阱、引擎差异 |
| `references/field-mapping.md` | JSON 别名 ↔ 技术键 ↔ 列号，取值格式 |
| `references/variants-and-quality.md` | 父子变体、SKU 命名、IDQ 评分对应字段 |
| `references/copy-rules.md` | lint 实际执行的文案判定 |
| `references/input-schema.md` | 数据 JSON 结构与必填清单 |
| `references/learned-requirements.json` | 从上传回执学到的按品类必填字段（自动维护） |
| `assets/example-listing.json` | 单品示例 |

文案本身还没写的，先用 `amazon-listing-builder` 出标题/五点/描述/ST。本 skill 只负责落地成可上传文件，不负责创作文案质量。

---

← 返回 [[00 本地Skills总览]]