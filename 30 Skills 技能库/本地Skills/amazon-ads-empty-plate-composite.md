---
title: amazon-ads-empty-plate-composite
type: skill
group: 广告素材
source_level: 第三方
upstream: Likely KuudoAI/skills（同 PolyForm-Shield 授权与英文体例，目录内未留链接）
local_path: C:/Users/Administrator/.workbuddy/skills/amazon-ads-empty-plate-composite/
updated: 2026-11-06
tags: [skill, workbuddy, 第三方, amazon]
---

# amazon-ads-empty-plate-composite

> [!note] ⚪ 第三方原样引入
> 专治「AI 把产品结构画歪」：先生成空镜场景，再抠真实产品照片合成，让滑轨、卡扣、铰链、磁吸这类机械结构严格对版。

## 速览

| 项 | 内容 |
|---|---|
| 来源 | **⚪ 第三方原样引入** |
| 上游 | Likely KuudoAI/skills（同 PolyForm-Shield 授权与英文体例，目录内未留链接） |
| 分组 | 广告素材 |
| 调用 | `@amazon-ads-empty-plate-composite` |
| 内部标题 | Empty plate + real product composite |
| 本地路径 | `C:/Users/Administrator/.workbuddy/skills/amazon-ads-empty-plate-composite/` |
| SKILL.md | 143 行 |
| 最后更新 | 2026-11-06 |

## 触发场景

结构画错了、抽屉/托盘不对、产品变形、悬浮感、换背景、空镜

## 内容结构

- **Decide first: generate, or composite?**
- **Rule one: one image proves one claim**
- **Rule two: the three constraints of an empty plate**
- **Rule three: describe mechanism as a parts list, never as a category**
- **Workflow**
- **Verify before upload**
- **Anti-patterns**
- **Resources**
- **Scope boundary**

## 附带文件

- `references/01-empty-plate-templates.md`
- `references/02-composite-spec.md`
- `references/03-failure-cases.md`

## 全文（SKILL.md 原文）

> 原 frontmatter 已剥离，正文原样归档，便于全文搜索。

# Empty plate + real product composite

Image models approximate, they do not model. They fail on anything whose
correctness depends on a relationship they cannot see — a tray inside a frame,
a rail between two layers, a plate flush against a surface. Asking for a better
description does not fix this, because the model has no representation of the
mechanism to describe.

The fix is to stop asking the model to draw the product.

## Decide first: generate, or composite?

| Product type | Approach |
|---|---|
| Soft goods, bottles, food, textiles, packaging | Generate directly. The model is sufficient. |
| Hard structure with a visible mechanism | **Composite.** Never generate the product. |
| The selling point *is* how it attaches | **Composite twice, from two angles.** |

Skipping this decision is the most expensive mistake in the workflow. A prompt
that describes mechanism is an invitation to invent one, and the model will
invent it confidently — a basket instead of a tray, a rail welded to the wrong
tier, a shelf hanging in mid-air.

## Rule one: one image proves one claim

"Fits in a five-inch gap" and "mounts magnetically" cannot share a frame. In a
head-on shot of a narrow gap both side panels are blocked by the machines, so
the back plate and magnets are **physically invisible** at that angle. Shooting
it that way while asking for visible magnets produces exactly the failure this
skill exists to prevent: an object with no attachment, reading as floating.

Split the claims:

- **Space claim** → the gap head-on; product seen only from the front; the
  flanking surfaces prove the fit.
- **Attachment claim** → one surface at 30–45°; back plate fully exposed; the
  product meets the panel with a closed seam.

One claim per image. Two images, one campaign asset each.

## Rule two: the three constraints of an empty plate

Every plate prompt needs all three. Drop any one and the composite will not sit.

1. **Give the dimension a physical anchor.** "About five inches wide, roughly
   one hand's width" — not a bare number. A naked number gets rendered as a
   generous gap, and the narrow-fit claim dies in the render.
2. **Name the surfaces that may appear, then stop.** "Only their plain side
   panels and a sliver of front bezel; no control panel, no display, no door
   ring, no brand plate." Unconstrained environments grow brand marks, display
   readouts and lettering reflected in brushed steel. All are automatic
   rejections, and all are invisible until you inspect the pixels.
3. **State the depth.** "You look straight down it to the painted wall behind."
   Without depth, the composite reads as pasted onto a flat backdrop no matter
   how good the cutout is.

Also state light direction in the plate. The composited shadow has to match it,
so fix the direction once and reuse it for every plate in the set.

## Rule three: describe mechanism as a parts list, never as a category

When the product must be generated anyway, abandon "two-tier shelf with
pull-out tray". To the model that phrase means *basket*. Use numbered parts
with an explicit who-moves / who-stays contract:

1. Two horizontal frames joined by four **unbroken** vertical rods, no joint,
   no separate unit hanging from them.
2. Top tier: fixed, welded to the frame, does not move.
3. Bottom tier: a fixed outer frame holding one separate shallow inner tray.
4. Only the inner tray slides, held horizontal; the rails live **between** the
   inner tray and the outer frame, so nothing protrudes past the frame outline.
5. Nothing touches the floor.

For attachment, replace the adjective with visible physics: "a flat steel back
plate the full height of the frame, flush against the painted panel, a hard
closed seam, no gap and no shadow line, two magnet pucks set into it." On its
own, "magnetically mounted" renders as levitation.

## Workflow

1. **Write the claim list.** One line per claim, one image per line.
2. **Generate the empty plate**, applying all three constraints. Wide format
   (1200 × 628 or larger) so cropping later is not destructive.
3. **Source the cutout from a white-background studio image**, not a lifestyle
   shot. Erase annotations first, then cut out. A lifestyle source drags stray
   lighting, props and brand marks into the composite; a white source is clean
   and keeps true proportions.
4. **Verify the source before compositing.** Frame continuous, no hardware
   outside the outline, tray shallow rather than basket-deep. If the
   white-background image is itself wrong, the whole route is void — go shoot
   the real unit. Never composite a wrong drawing cleanly; that just makes the
   error more convincing.
5. **Composite.** Proportional scale only (product height ≈ 40% of appliance
   height, never stretch to fill), edges aligned to the surfaces it grips,
   contact shadow only where it touches (5–8 px feather), cutout warmed
   +200–300 K to match the plate. Export 1200 × 628, PNG or JPEG, ≤ 5 MB.

## Verify before upload

Four checks, in this order:

1. Proportional scale — product height against the appliance it hangs on.
2. Shadow direction matches the plate's light.
3. No white fringe or halo along the cutout edge.
4. No text anywhere in frame — bottle labels, panel readouts, reflected
   lettering on steel.

Failing any of these means re-composite, not retouch.

## Anti-patterns

- Expecting a model to render hidden mechanism. It cannot, and confidence in
  the output is not evidence it did.
- One frame proving space fit and attachment together.
- Supplying five reference images. Two is the ceiling; extra layout images with
  arrows, callouts and headlines actively corrupt the read.
- Long negative lists inside a prompt. Largely ignored. The post-render visual
  pass is the real control.
- Using "pull-out tray", "drawer" and "basket" interchangeably. Each maps to a
  different wrong shape.
- Sizing the cutout by eye. Use the ratio reference in `assets/`.

## Resources

- [`references/01-empty-plate-templates.md`](references/01-empty-plate-templates.md)
  — four plate templates (slot between appliances, countertop, wall, inside a
  cabinet) plus the swap sentences that convert one into another
- [`references/02-composite-spec.md`](references/02-composite-spec.md) —
  cutout, scale, contact shadow, colour temperature, export and QC
- [`references/03-failure-cases.md`](references/03-failure-cases.md) — six
  observed failures, each mapped to the rule that prevents it
- `assets/ratio-reference-dimensioned.jpg` — white-background product photo
  with dimensions, for proportion benchmarking. Annotated: erase the
  dimension lines and arrows before using it as a cutout.

## Scope boundary

This skill decides **how to make the image**. Which program you are buying,
what may legally appear inside the pixels, and whether the asset will be
rejected belong to `amazon-ads-creative-assets`. Ask that skill first when the
program is still unknown; come here once the answer is "custom image, product
must be exact".

---

← 返回 [[00 本地Skills总览]]