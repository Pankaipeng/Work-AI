---
title: amazon-report-archive
type: skill
group: 财务 / 报表
source_level: 自建
upstream: —（无上游，本地原创）
local_path: C:/Users/Administrator/.workbuddy/skills/amazon-report-archive/
updated: 2026-11-04
tags: [skill, workbuddy, 自建, amazon, 我的skill]
---

# amazon-report-archive

> [!tip] 🟢 自建（我个人创建）
> 把亚马逊后台下载的回款/超库龄/仓储费/广告报表按规范名归档，并生成「N月回款备注.xlsx」与「N月利润核对表.xlsx」；缺数据一律留空不编造。

> **判断依据**：内置你自己的飞书 tenant（bqnscp788os）云盘路径与技能包发布记录，对外只有你的使用说明链接。
>

## 速览

| 项 | 内容 |
|---|---|
| 来源 | **🟢 自建（我个人创建）** |
| 上游 | —（无上游，本地原创） |
| 分组 | 财务 / 报表 |
| 调用 | `@amazon-report-archive` |
| 内部标题 | 亚马逊月度报表归档（纯本地） |
| 本地路径 | `C:/Users/Administrator/.workbuddy/skills/amazon-report-archive/` |
| SKILL.md | 241 行 |
| 最后更新 | 2026-11-04 |

## 触发场景

归档本月报表、整理月度报表、生成利润核对表、做回款备注、跑一下 N 月的表

## 飞书链接

| 用途 | 链接 |
|---|---|
| 在线使用说明（永远最新） | https://bqnscp788os.feishu.cn/docx/Na7xdV9xmoIZxXxx3vHcLCGanxf |
| 技能包下载（.zip，约 43KB） | https://bqnscp788os.feishu.cn/file/LsclbdKDPoZouXxQhyscQkmkn9b |
| 飞书云盘根目录「财务数据」 | https://bqnscp788os.feishu.cn/drive/folder/AItLfwU1clx70TdEqoDcaHAKnTo |

## 内容结构

- **0. 边界：这个 skill 只做本地**
- **1. 先确认路径与店铺（第零步，不可跳过）**
- **2. 快速执行卡**
- **3. 输出物（与 9月 参考目录形态一致：全部平铺、文件名带月份）**
- **4. 执行流程**
  - 4.0 先看 error_kind，再决定说什么
  - 4.1 定月份
  - 4.2 识别报表类型（按表头签名，不靠文件名猜）
  - 4.3 每月必备 6 份，缺哪份就报出来
  - 4.4 生成两张表
- **5. 利润核对表口径（26 列，2026-11-04 定稿）**
- **6. 已知坑**
- **7. 分发给同事**

## 附带文件

- `使用说明.md`
- `references/feishu-manual-upload.md`
- `scripts/build_profit_sheet.py`
- `scripts/build_receipt_notes.py`
- `scripts/compute_profit.py`
- `scripts/config.py`
- `scripts/prepare_month.py`
- `scripts/scan_reports.py`

## 全文（SKILL.md 原文）

> 原 frontmatter 已剥离，正文原样归档，便于全文搜索。

# 亚马逊月度报表归档（纯本地）

## 0. 边界：这个 skill 只做本地

| 做 | 不做 |
|---|---|
| 扫描下载目录、按表头识别报表类型、判定所属月份 | ❌ 不登录飞书、不调 `lark-cli` |
| 按规范名复制到固定输出目录 | ❌ 不建月份文件夹、不上传云盘 |
| 生成回款备注.xlsx、利润核对表.xlsx | ❌ 不写在线表格 |
| 列出缺失报表、影响哪些列 | ❌ 不替用户补数、不猜数字 |

飞书那边的操作（若以后要恢复）见 `references/feishu-manual-upload.md`，那只是**人工上传的对照说明**，不含任何脚本。

## 1. 先确认路径与店铺（第零步，不可跳过）

**首次运行、换机器、或换店铺，必须先把路径和店铺问清楚，不许猜、不许用示例路径、不许沿用作者机器上的「巨杉 / 美国」。**

```bash
S="C:/Users/Administrator/.workbuddy/skills/amazon-report-archive/scripts"
python "$S/prepare_month.py" --print-config     # 看当前生效的路径 / 店铺 / 来源
```

两块信息都可能缺，返回的都是同一个「该问用户了」结构（不是报错）：

```json
{
  "ok": false,
  "need_input": ["in_dir"],
  "reason": "还没有配置亚马逊报表下载目录",
  "ask": "亚马逊后台的月度报表（回款表、广告报表那些）平时下载到哪个文件夹？",
  "out_dir_suggestion": "C:\\Users\\<你>\\Documents\\财务报表归档",
  "how": "拿到路径后重跑：python prepare_month.py --in \"<报表目录>\" --save-config"
}
```

```json
{
  "ok": false,
  "need_input": ["shop", "site"],
  "reason": "不知道这张表的「店铺 / 站点」该写什么，不能替你猜",
  "ask": "利润核对表里的「店铺 / 站点」写什么？例如 店铺=巨杉、站点=美国（目录名里看到的是「青阳 / 美国」，如果是同一家就不用改）",
  "suggestion": {"shop": "青阳", "site": "美国"},
  "note": "目录名里的名字可能是浏览器环境名或昵称，所以只作参考，不自动填表"
}
```

看到 `need_input` 就停下来问，话术（一次问全，别挤牙膏）：

> 开跑前先确认三件事：① 亚马逊报表（回款表、广告报表那些）平时下载到哪个文件夹？② 生成的表格放哪？默认放「文档/财务报表归档」，想换地方跟我说。③ 这张利润核对表的「店铺」和「站点」写什么？比如 店铺=巨杉、站点=美国。

拿到答案后带 `--save-config` 跑一次，路径和店铺都记住，以后不再问：

```bash
python "$S/prepare_month.py" --in "<报表目录>" --out "<输出目录>" \
       --shop "<店铺名>" --site "<站点>" --save-config --dry-run
```

配置写在 `~/.amazon-report-archive/config.json`，**故意放在 skill 目录之外**：skill 通过 zip 上传或市场更新是整包覆盖的，放在里面会被下一次更新冲掉。

优先级：`--in` / `--out` / `--shop` / `--site` 参数 → 环境变量 `AMZ_REPORT_DIR` / `AMZ_OUT_DIR` / `AMZ_SHOP` / `AMZ_SITE` → 用户配置文件 → 未配置（触发询问）。

**店铺 / 站点只影响两处，不影响金额**：利润核对表的「店铺」「站点」两列，以及报表副本的文件名前缀（`{店铺}{站点}{M}月回款.csv`）。所以推断错不会算错钱，但会写错表头，仍然要问。

> 自动推断只做参考，不自动填表：输出目录里的历史利润核对表 → 历史报表文件名（这两类可信）；报表目录名（如 `亚马逊-美国-青阳` **只是参考**，那可能是浏览器环境名或店铺昵称，不一定是店铺）。表中「店铺/站点」来自用户明确输入或历史产出，认不出就停下来问。

**用户上传附件的情况**：附件散落在工作目录时，先把附件归拢到一个临时目录（如 `<输出目录>/_upload_<月份>/`），再把该目录当 `--in` 传进去。不要为"附件模式"另加参数——下游三个脚本都是扫目录的。

模板（利润核对表）规则：取输出目录里最近的一份历史核对表，**并排除目标月自己的那份**（重跑 10 月不会拿「10月利润核对表」当模板）。换新模板时用 `--template <路径>` 指定。

## 2. 快速执行卡

```bash
S="C:/Users/Administrator/.workbuddy/skills/amazon-report-archive/scripts"

# 首次：看当前配置（in_dir / 店铺 为空就得先问用户）
python "$S/prepare_month.py" --print-config

# 首次：确认路径+店铺后记住
python "$S/prepare_month.py" --in "<报表目录>" --out "<输出目录>" \
       --shop "<店铺名>" --site "<站点>" --save-config --dry-run

# 日常：只看计划，不落盘（推荐先跑一次）
python "$S/prepare_month.py" --month 2026-10 --dry-run

# 日常：正式执行：复制报表 + 生成两张表
python "$S/prepare_month.py" --month 2026-10

# 不带 --month 默认 = 上一个自然月
python "$S/prepare_month.py"

# 一个输出目录放多家店铺时，给产出加「店铺站点」前缀，避免互相覆盖
python "$S/prepare_month.py" --month 2026-10 --prefix auto
```

`prepare_month.py` 输出一段 JSON：`plan`（要复制哪些）、`copied`/`unchanged`、`missing`（缺哪些报表 + 影响哪些列）、`outputs`（两张表的路径）、`warnings`、`assumed`（哪些路径是默认值而非用户指定）、`config_saved`。

## 3. 输出物（与 `9月` 参考目录形态一致：全部平铺、文件名带月份）

```
OUT_DIR/
├── {店铺}{站点}{M}月回款.csv                      ← 原始报表的规范命名副本
├── {店铺}{站点}{M}月超库龄.csv
├── {店铺}{站点}{M}月仓储费.csv
├── {店铺}{站点}{M}月商品推广_推广的商品_报告.xlsx
├── {店铺}{站点}{M}月品牌推广_广告活动_报告.xlsx
├── {店铺}{站点}{M}月展示型推广_推广的商品_报告.xlsx
├── {M}月回款备注.xlsx                        ← 生成：32 列，只含 type=Amazon Fees
└── {M}月利润核对表.xlsx                       ← 生成：复制上月模板，覆盖第 3 行 + 备注
```

`{店铺}{站点}` 来自用户输入（如 `巨杉美国`、`青阳美国`），**不是写死的**。两张生成表默认不带前缀以保持文件名稳定；多家店铺共用一个输出目录时用 `--prefix auto`，变成 `巨杉美国10月利润核对表.xlsx`。

规范命名映射写在 `scan_reports.NAME_RENAME`（模板用 `{shop}{site}{m}` 占位），改那里即可。

## 4. 执行流程

### 4.0 先看 `error_kind`，再决定说什么

失败时别把所有情况都说成"报表没下载"，按 `error_kind` 区分：

| `error_kind` | 含义 | 该跟用户说什么 |
|---|---|---|
| `need_input` | 还没配 `in_dir`，或不知道店铺/站点 | 直接问（见第零步），不要猜；店铺可把 `suggestion` 摆给用户确认 |
| `dir_not_found` | 路径不存在 | "这个路径打错了？"再把 `nearby_dirs` 里的候选原样贴给用户，通常就是正确路径 |
| `empty_dir` | 目录里没有 csv/xlsx | "报表可能还没下载，或者选到了上层目录"，并贴 `sample_files` |
| `no_reports` | 有表格文件，但没有一份能识别成亚马逊报表 | "可能选错文件夹，或报表还在压缩包里"，并贴 `sample_files` |
| `month_empty` | 有报表但没有目标月份的 | 贴 `months_available`（近 6 个月各几份），问"要做哪个月" |

`nearby_dirs` 会定位到**具体哪一层写错了**并拼出完整可用路径，例如把 `Super Broswer` 纠正成 `Super Browser\亚马逊-美国-青阳`，可以直接让用户确认后重跑。

### 4.1 定月份

| 场景 | 参数 |
|---|---|
| "归档上个月"（默认） | 不带 `--month`，= 上一个自然月 |
| "做 10 月的" | `--month 2026-10` |
| 不确定有哪些 | 先 `python "$S/scan_reports.py" --days 30` 看清单 |

月份以**报表自身**为准：10 月的报表即使在 11 月下载，也归 10 月。判定优先级：文件名（`2026年10月` > `2026Oct` > `2026-10` > `10月`）→ **文件内日期**（`snapshot-date` / `date/time` / `month_of_charge` / `return-date`）→ 文件修改时间（标 `inferred`）。

### 4.2 识别报表类型（按表头签名，不靠文件名猜）

| 类型 | 签名特征 |
|---|---|
| 回款 | `date/time` + `settlement id` + `total`（前 9 行是说明，第 10 行才是表头，32 列） |
| 超库龄 | `snapshot-date` + `qty-charged` + `amount-charged` |
| 仓储费 | `estimated_monthly_storage_fee` |
| 商品推广 | `7天总销售额` |
| 品牌推广 | `费用类型` |
| 展示型推广 | `费用类型` + `广告组名称` |

排除项：`~$*`（Excel 锁文件）、`FIXED-*`（促销模板等工作文件）、`_*`（临时文件）、pdf/图片/压缩包，以及**本 skill 自己的产出**（`*回款备注*`、`*利润核对表*`——回款备注与回款表头完全一样，不拦掉会被当成回款表重算，数字全错）。

### 4.3 每月必备 6 份，缺哪份就报出来

| 缺 | 影响 |
|---|---|
| 回款 | **整张表都出不来**，直接停 |
| 仓储费 | 仓储费列留空（无法与长仓费合并） |
| 商品推广 | 关键词广告费留空 |
| 品牌推广 | 品牌广告费留空 |
| 展示型推广 | 视频广告费留空 |
| 超库龄 | 只留档，不影响数值 |

**没有的报表不补、不按 0 计**。汇报时必须原文列出 `missing` 及影响列，让用户去补。

### 4.4 生成两张表

```bash
# 只要回款备注
python "$S/build_receipt_notes.py" --month 2026-10
#   -> OUT_DIR/10月回款备注.xlsx   (第1行=原始表头32列，第2行起=type=="Amazon Fees")

# 只要利润核对表
python "$S/build_profit_sheet.py" --month 2026-10 --out "$OUT_DIR/10月利润核对表.xlsx" \
       --shop "巨杉" --site "美国"
#   --shop/--site 不传则按 配置 > 历史产出 推断，认不出会留空并 warning
#   加 --no-notes：表里只留 26 列数据，备注另存为同名 .备注.txt
#   加 --template <路径>：指定模板
```

- `build_profit_sheet.py` 会**逐列校验模板表头**与 `compute_profit.COLUMNS` 是否一致，模板换版直接报错，不会静默错位。
- 数值按原样写入，**不做 2 位四舍五入**（品牌 71.128、视频 1559.7454 原样保留）。
- 模板被 Excel/WPS/预览面板占用会 `PermissionError` → 先关掉再跑。

## 5. 利润核对表口径（26 列，2026-11-04 定稿）

模板：`{M-1}月利润核对表.xlsx`（第 1 行空、第 2 行表头、第 3 行数据）。9 月与 10 月双月回归通过。

| 列 | 规则 | 2026-10 |
|---|---|---|
| 店铺 / 站点 | **来自用户输入**（`--shop` / `--site`，或已保存的配置，或历史产出里的值）；认不出就停下来问，绝不默认写「巨杉 / 美国」 | 巨杉 / 美国 |
| 回款额 | `sku` 非空行 `total` 求和（含 Refund/Liquidations 负数冲回） | 158827.37 |
| 秒杀/七天亚马逊促销收费 | type 含 `Amazon Fee` 且 `description` 含 `Deal` → 绝对值（Participation + Performance） | 2483.82 |
| 优惠券亚马逊收费 | Coupon 费用；无来源就留空 | 空 |
| 合仓费 | **固定留空**：回款表没有可靠口径，`Inbound Placement` 是入库配置费不是合仓费 | 空 |
| 仓储费 | **= 当月月度仓储费报表 `estimated_monthly_storage_fee`(US) + 回款表 `FBA Long Term Storage Fee`**；报表未下载则**整列留空**（不填长仓费、不写 0） | **空** |
| 关键词 / 品牌 / 视频广告费 | 三块广告 xlsx 的 `花费` 列求和（按表头名定位，列号每月不同）；缺报表 → 留空 | 39129.42 / 71.128 / 1559.7454 |
| 弃置费 | `description` 含 `disposal` → 绝对值 | 38.39 |
| **其他** | **只装两类**：`Vine Enrollment Fee` + `FBA Customer Returns Fee`；必须在备注里拆明细 | **312.88**（Vine 200 + FBA退货 112.88） |
| 提现手续费 / 销售增值税 | 回款表无对应项（`Transfer` 是提现本身，不是费用，必须排除） | 空 |
| 数量 | `sku` 非空 且 type **不含** `Refund` / `Liquidation` 的 `quantity` 求和（`Liquidations` 与 `Liquidations Adjustments` **都不计**） | 26388 |
| 采购成本 / 运输费用 / 关税 / 海外仓中转 / 其他杂费 / 文案翻译费 / 图片视频费 / 差评处理费 / 站外促销费 / 其他费用 | 回款表无数据源，留空给业务侧填 | 空 |
| 销售额 | `sku` 非空行 `product sales` 求和（**填"销售额"列**，不要落到"其他费用"） | 336769.72 |

三条必须记住的约定：

1. **不进任何列的账户级费用**：店铺月租（`Subscription`）、FBA 入库配置服务费、零散杂项。它们会写进备注第 4 条，但**不加进"其他"**（9 月参考表的口径，2026-11-04 用户确认）。
2. **`Cost of Advertising` 只做核对**：2026-10 是 41644.92，三块广告报表合计 40760.2934，差 884.6266 属结算跨月/其他广告类型，不并入费用合计。
3. **仓储费的时间差**：回款表 `FBA Inventory Storage Fee` 扣款日约在次月初，扣的是**上月**仓储费（10/7 扣的 5817.98 是 9 月的），`compute_profit` 只把它记到 `_回款表仓储费扣款(属上月,不入表)`，永不用它填本月仓储费。
4. **数量口径 2026-11-04 变更（补丁）**：`Liquidations Adjustments` 从"计入"改为"**不计入**"。影响：10 月 26459 → **26388**（-71）；9 月 26162 → **26159**（-3）。**9 月参考表的 26162 是旧口径**（含 Adjustments 3 件），新口径下 9 月必然差 3，这是口径差异不是算错，不必追平。

## 6. 已知坑

1. 回款备注与回款表**表头一模一样**（都是 32 列），只能靠文件名区分 → `classify(rows, name)` 必须先看名字，否则会把生成的备注当原始回款重算。
2. 广告 xlsx 的 sheet 维度记录缺失，`openpyxl(read_only=True)` 只能读到 1 列 → 必须完整加载。
3. 数字命名的 CSV（如 `734942020761.csv`）看不出月份，也**不能**只看下载时间 → 读文件内 `snapshot-date`。
4. `~$xxx.xlsx` 是 Excel 锁文件、`FIXED-xxx-Products.xlsx` 是促销上传模板，都不是报表。
5. 月度仓储费报表通常**次月 10-15 日**才出。没到就留空，不要用回款表扣费顶替。
6. 9 月参考表的仓储费 5817.9542 与当月报表 US 合计 5817.9801 差 0.0259（报表版本不同），属历史差异，不必追平。
7. 输出目录若同时是输入目录（参考目录既是下载目录又放产出），靠坑 1 的拦截逻辑保护，不会自我读取。
8. 目标 xlsx 被 Excel/WPS 打开会 `PermissionError` → 先关闭。
9. **`in_dir` 为空不是 bug**：`config.py` 里刻意没有任何默认路径，返回 `need_input` 就是"该问用户了"的信号。不要去代码里硬塞一个路径绕过去——那正是同事装完跑不通的老原因。
10. 重跑同一个月不要用当月产出当模板；`template_candidates(exclude_month=m)` 已经挡掉，手写 `--template` 时注意别指向本月自己。
11. **店铺/站点不要从目录名硬猜**：`D:\...\亚马逊-美国-青阳` 里的「青阳」是浏览器环境名/昵称，实际店铺是「巨杉」。目录名只能当 `suggestion` 摆给用户确认，写进配置或历史产出里的才可信（2026-11-04 实测踩到）。
12. 报表副本的文件名依赖店铺站点；改了店铺后旧副本不会被自动改名，需要的话手动删掉重跑（脚本只按大小判断是否重拷）。

## 7. 分发给同事

zip 包直接上传安装（`专家 · 技能 · 连接器` → 技能 → 添加技能 → 上传技能），**不用解压，也不用改任何代码**。

装上后第一次运行会问路径和店铺，答一次就写进 `~/.amazon-report-archive/config.json`，之后不再问。同一台机器换店铺 / 换目录时：

```bash
python "$S/prepare_month.py" --in "<新目录>" --shop "<新店铺>" --site "<站点>" --save-config --dry-run
```

模板：把上一月的 `{M-1}月利润核对表.xlsx` 放进输出目录即可，脚本自动取最近一份并排除本月自己；也可 `--template` 指定。

`scripts/config.py` 里已经没有任何机器相关路径，纯粹是配置加载逻辑，不需要改。

---

← 返回 [[00 本地Skills总览]]