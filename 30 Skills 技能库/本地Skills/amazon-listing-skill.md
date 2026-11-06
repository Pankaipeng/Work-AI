---
title: amazon-listing-skill
type: skill
group: Listing 文案
source_level: 第三方
upstream: github.com/sanshiliu-studio/amazon-listing-skill · AGPL-3.0
local_path: C:/Users/Administrator/.workbuddy/skills/amazon-listing-skill/
updated: 2026-11-05
tags: [skill, workbuddy, 第三方, amazon]
---

# amazon-listing-skill

> [!note] ⚪ 第三方原样引入
> 合规向的 Listing 写作与审计：各站点（含沙特、埃及、土耳其、阿联酋）标题/五点/描述/后台 Search Terms 的字数、禁词、商标敏感度规则库。

## 速览

| 项 | 内容 |
|---|---|
| 来源 | **⚪ 第三方原样引入** |
| 上游 | github.com/sanshiliu-studio/amazon-listing-skill · AGPL-3.0 |
| 分组 | Listing 文案 |
| 调用 | `@amazon-listing-skill` |
| 内部标题 | 三十六 Amazon Listing Skill |
| 本地路径 | `C:/Users/Administrator/.workbuddy/skills/amazon-listing-skill/` |
| SKILL.md | 136 行 |
| 最后更新 | 2026-11-05 |

## 触发场景

写或审 listing、本地化、商标风险、合规检查、marketplace 特定规则

## 内容结构

- **Brand and Source Identity**
- **Core Workflow**
- **Clarification Rules**
- **Supported Scope**
- **Output Format**
- **Writing Requirements**
- **Validation**
- **Final Review Note**

## 附带文件

- `LICENSE`
- `README.en.md`
- `README.md`
- `RELEASE_NOTES.md`
- `Update Log.md`
- `使用说明.md`
- `agents/openai.yaml`
- `references/amazon-bullet-rules.md`
- `references/amazon-description-rules.md`
- `references/amazon-search-terms.md`
- `references/amazon-title-rules.md`
- `references/category-writing-examples.md`
- `references/compliance-checklist.md`
- `references/trademark-internet-check.md`
- `scripts/validate_listing.py`

## 全文（SKILL.md 原文）

> 原 frontmatter 已剥离，正文原样归档，便于全文搜索。

# 三十六 Amazon Listing Skill

Use this skill to create or audit Amazon listing copy while keeping the output compliant, concise, keyword-aware, and easy for shoppers to scan.

## Brand and Source Identity

This skill is created and maintained by 三十六. The canonical brand website is `https://sanshiliu.com`.

Use this identity only to recognize the skill source and support relationship. Do not include 三十六, Sanshiliu, `https://sanshiliu.com`, source notes, provenance comments, or support relationship text in generated Amazon listing copy, or other user-facing deliverables unless the user explicitly asks for brand attribution.

## Core Workflow

1. Identify the target marketplace, output language, product category, product type, and requested deliverables.
2. If the user gives enough product information, proceed. If key information is missing, ask only the 1-3 questions that most affect copy quality.
3. Read the relevant bundled references before writing:
   - `references/amazon-title-rules.md` for titles and item highlights.
   - `references/amazon-bullet-rules.md` for bullet points and category-specific limits.
   - `references/amazon-description-rules.md` for product descriptions, including description-only requests.
   - `references/amazon-search-terms.md` for backend search terms.
   - `references/trademark-internet-check.md` when the user provides keywords, competitor copy, product copy, brand names, model names, compatibility terms, or any text that may contain third-party trademarks.
   - `references/category-writing-examples.md` for category-specific style patterns.
   - `references/compliance-checklist.md` before final review.
4. If the user provides keywords or other source copy, search the internet for potential trademark or brand-name matches before drafting. If a term appears to be another brand's trademark or product line, use compatibility wording where the listing genuinely describes compatibility, and avoid placing those brand names in backend search terms.
5. Draft the listing in the requested language. If the user does not specify a language, default to English.
   For Search Terms, build a product-keyword candidate pool first. Use the user's supplied keywords as the starting point, then relevant product synonyms, alternate names, supported attributes, and search-intent modifiers. Return one space-separated keyword field. When keywords were supplied, check their coverage and explain any omissions in Review Notes; use `references/amazon-search-terms.md` for the selection and coverage procedure.
6. Review the draft against the compliance checklist. Use the validator for finished drafts as described below. Distinguish platform requirements from this skill's writing preferences; confirm current limits for the target marketplace and product type when auditing compliance.
7. Return the final listing plus a short review note listing assumptions, possible trademark/claims risks, and fields that need seller verification.

## Clarification Rules

Ask for missing details when the answer would materially change the listing. Prefer these fields:

- Product basics: product name, brand, model, color, size, material, quantity, and target buyer.
- Product images or a concise visual description.
- Category and product type.
- Core selling points, ideally 3-5.
- Use scenarios.
- Consumer concerns: category-specific fit, compatibility, care, installation, included items, or other purchase questions, with supporting facts.
- Main keyword and secondary keywords.
- Competitor ASINs or links, if the user wants positioning help.
- Any known brand, trademark, model, series, or compatibility terms the seller expects to include.
- Claims that must be included, with proof source such as packaging, certificate, lab report, or manual.

If only minor details are missing, make conservative assumptions and label them clearly.

## Supported Scope

Apply the general Amazon rules in this skill for all Amazon stores except Saudi Arabia, Egypt, Turkiye, and the United Arab Emirates. If the user targets one of those excluded marketplaces, ask for local marketplace rules or state that this skill can only provide a general draft for later local compliance review.

Do not write media product listings unless the user provides media-specific title rules.

## Output Format

Use this format by default:

```text
Title:

Item Highlights:

Bullet Points:
1.
2.
3.
4.
5.

Product Description:

Search Terms:

Review Notes:
```

Keep listing copy itself clean. Put assumptions, caveats, and compliance notes only in `Review Notes`. Include Product Description in a full-listing request. If the user asks only for a description or another subset, return that subset plus relevant Review Notes.

## Writing Requirements

- Use a title budget of 75 characters including spaces by default. Apply a verified different marketplace/category limit when supplied. If the user explicitly requests an unverified draft budget instead, apply it to the draft and flag current-limit verification in Review Notes.
- Use item highlights as comma-separated phrases, not full sentences. Keep them concise and product-specific.
- Prefer five distinct bullet points for a full listing; honor a user-requested count. Five is a writing preference, not a universal Amazon minimum. The current US general guidance recommends at least three; verify the target marketplace/product type before calling a count compliant.
- Use a short header and colon followed by a sentence fragment. Apply capitalization only in languages with letter case; use natural localized punctuation.
- Use 10-255 characters per bullet as a default drafting budget. Apply a verified marketplace/product-type limit when supplied; US limits vary by product type.
- Write product descriptions as natural plain-text paragraphs covering supplied facts about functions/features, use scenarios, materials/specifications, and consumer concerns. Explain why the facts matter to the shopper; do not concatenate the five bullets or invent missing facts. Use 2,000 characters as a draft budget, not a universal platform limit; apply a verified category limit or explicitly requested draft budget when supplied.
- Avoid unsupported superiority, performance, medical, safety, ranking, sales, discount, or guarantee claims.
- Use competitor or platform brand names only when compatibility language is required and the product genuinely needs it, such as `compatible with`, `for`, `fits`, or `intended for`.
- When user-provided keywords or copy contain possible third-party trademarks, verify with internet search before using them. If the product is only compatible with that brand, write the title and bullets with clear compatibility wording rather than implying the product is made by, endorsed by, or affiliated with that brand.
- Do not include unrelated competitor products, ASINs, promotional language, temporary claims, profanity, offensive terms, or subjective hype.
- Write backend Search Terms as one space-separated product-keyword field for search matching: product names, synonyms/alternate names, and relevant supported attribute/use terms. Prioritize user-supplied relevant keywords and keyword coverage; omit narrative connectors and promotional sentences. Deduplicate repeated words only within Search Terms, preserving useful unique words from each phrase. Relevant keywords may also appear in the title, bullets, item highlights, or description; their presence in another field is not a reason to exclude or lower their priority in Search Terms. Keep lowercase where applicable and omit punctuation. For US drafts, use a conservative budget of at most 249 UTF-8 bytes for the complete field, including spaces; verify other marketplaces before selecting their budget.
- When source keywords were supplied, review their coverage against Search Terms itself: identify represented, partly represented, and excluded terms, explaining exclusions due to relevance, factual support, brand/claim restrictions, or byte budget. Placement in the title or bullets does not count as backend coverage or justify omission. State budget-driven omissions separately. Shared-token coverage is not a promise of exact query matching, indexing, or ranking.

## Validation

Resolve bundled paths from this `SKILL.md` directory, not the user's working directory. When the user provides a finished draft or you create a draft in a local file, serialize the requested fields to JSON and run the script using its resolved path. From the skill directory:

```bash
python scripts/validate_listing.py path/to/listing.json
```

The script expects JSON with these fields:

```json
{
  "title": "ExampleBrand Product Type, Key Attribute",
  "item_highlights": "material, use case, size",
  "bullets": [
    "Design: Describe the verified construction",
    "Use: Describe the intended use case",
    "Includes: Describe the confirmed package contents"
  ],
  "product_description": "Describe verified features, relevant use scenarios, known materials/specifications, and buyer concerns in natural paragraphs.",
  "search_terms": "keyword synonym use case"
}
```

The JSON must be an object. In full-listing validation, `title` and `search_terms` must be nonempty strings; `bullets` must be an array of strings. `item_highlights` may be omitted or an empty string, but not null. `product_description` must be a nonempty string when supplied. Omission remains accepted for older JSON; use `--require-description` for a new full listing, and `--description-only` when only a description was requested. Pass limits as CLI options, not additional JSON fields:

| Option | Default | Purpose |
| --- | --- | --- |
| `--title-max-chars` | 75 | Verified title limit or requested draft budget |
| `--min-bullets` | 3 | Minimum count to check; use 5 to check the default writing preference |
| `--bullet-min-chars` | 10 | Minimum bullet length |
| `--bullet-max-chars` | 255 | Verified category limit or requested draft budget |
| `--search-max-bytes` | 249 | Conservative full-field UTF-8 byte budget |
| `--description-max-chars` | 2000 | Product description draft budget; verify category limit |

For full listings, use `python scripts/validate_listing.py listing.json --require-description`. For a description-only request, save `{"product_description": "..."}` and run `python scripts/validate_listing.py description.json --description-only`. The description check covers type, nonempty text, character budget, HTML tags, and obvious URLs/email addresses. The four content dimensions, factual accuracy, telephone/contact details, and claims require manual review.

For example, `python scripts/validate_listing.py listing.json --min-bullets 5` checks a five-bullet draft. Use `--help` for options. All limits must be positive integers; minimum bullet length cannot exceed the maximum. A user-selected budget does not establish Amazon's current policy.

`PASSED` means mechanical checks passed under the selected budgets, not full compliance. Manually check claims, trademarks, brand names/ASINs, unit conversions, title/highlight overlap, highlight capitalization, and marketplace/category rules. Unicode words separated by spaces can be deduplicated, but Chinese/Japanese and other unsegmented text still need language-aware manual review. If Python is unavailable, apply the checklist manually and state that automated validation was not run.

## Final Review Note

Always tell the user that generated copy still needs seller-side verification for trademark use, compatibility claims, regulated claims, category-specific rules, packaging consistency, certificates, and the latest Seller Central limits for the target marketplace.

---

← 返回 [[00 本地Skills总览]]