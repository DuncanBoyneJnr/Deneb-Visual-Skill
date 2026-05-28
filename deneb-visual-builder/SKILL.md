---
name: deneb-visual-builder
description: Generate, adapt, debug, review, and explain Deneb Vega-Lite visuals for Power BI, including tooltips, formatting, interactivity, cross-highlighting, and safe templates. Also covers editing visual.json files directly in PBIR projects.
---

# Deneb Visual Builder

Use this skill when the user asks Claude to create, adapt, improve, debug, review, or template a Deneb visual for Power BI using Vega-Lite or Vega — whether generating a new spec or editing an existing `visual.json` in a PBIR project.

The default output is Deneb-safe Vega-Lite JSON unless the user explicitly needs Vega-only behaviour.

## Critical gotchas — read first

These are the failure modes that will silently break a visual. Get them right before anything else.

### 1. `pbiFormat` / `pbiFormatAutoUnit` REQUIRE an explicit `format` string

`formatType: "pbiFormat"` tells Vega which formatter to call. It does NOT fall back to the measure's model format string. If you set `formatType` without a `format`, every value renders as `undefined`.

❌ **Broken** — values display as "undefined":
```json
{ "field": "Revenue", "type": "quantitative", "formatType": "pbiFormat" }
```

✅ **Correct** — explicit Power BI format string:
```json
{ "field": "Revenue", "type": "quantitative", "format": "$#,##0", "formatType": "pbiFormat" }
```

### 2. d3 format strings are NOT Power BI format strings

The two systems look similar but are syntactically different. Never pair a d3 string with `pbiFormat` — it will render `undefined`. Either:

- Use a d3 string with NO `formatType` (Vega's default d3-format), OR
- Convert to a Power BI format string AND set `formatType: "pbiFormat"`

Cheat sheet for the most common conversions:

| d3 (default) | Power BI (`pbiFormat`) | Notes |
|---|---|---|
| `$,.0f` | `$#,##0` | Currency, no decimals |
| `$,.2f` | `$#,##0.00` | Currency, 2dp |
| `$,.0s` (SI prefix) | `$#,##0` + `formatType: "pbiFormatAutoUnit"` | Auto K/M/B |
| `,.0f` | `#,##0` | Integer with thousands |
| `,.2f` | `#,##0.00` | 2dp with thousands |
| `.0%` | `0%` | Percent, no decimals |
| `.1%` | `0.0%` | Percent, 1dp |
| `.2%` | `0.00%` | Percent, 2dp |
| `+.0%` | `+0%;-0%;0%` | Signed percent (positive shows `+`) |
| `+.1%` | `+0.0%;-0.0%;0.0%` | Signed percent, 1dp |

The Power BI 3-section semicolon format is `positive;negative;zero`. Use it to force a `+` sign on positive values — d3's `+` modifier has no clean PBI equivalent without the 3 sections.

### 3. Cross-highlight opacity must be applied to every layer

If only the base bar has `__selected__` opacity but the dot/label/circle layers don't, the unselected dots stay fully opaque while the bars dim — visually inconsistent. Add the same condition to every mark layer that should respond to selection.

Canonical pattern:
```json
"opacity": {
  "condition": { "test": { "field": "__selected__", "equal": "off" }, "value": 0.25 },
  "value": 1
}
```

Also: put `opacity` in the layer's `encoding`, not on `mark`. A `mark.opacity: 0.95` is a flat value that ignores selection state.

### 4. TopN belongs in Power BI's filter pane, not in a Vega transform

Doing `window` + `filter` inside the spec works, but:
- The model returns the full dataset every render and Vega discards 90%+.
- The "12" is buried in JSON instead of a user-editable filter slider.
- Some Power BI selection/drillthrough behaviour gets confused by transform-pre-filtered data.

Recommend Power BI's TopN visual filter (filter pane → TopN filter on the category by the measure). Only keep `window`+`filter` inside Vega when you need it scoped to one layer (e.g., label "top 8" while keeping all marks visible).

### 5. `pbiFormat` requires the field to be a Power BI measure or column

Calculated/derived fields produced by Vega transforms (`calculate`, `joinaggregate`, `window`) do not have a Power BI format string and `pbiFormat` cannot use them. Stick to d3 format strings for derived fields.

---

## Core purpose

Help the user produce high-quality Deneb visuals that work inside Power BI, not just generic Vega-Lite examples.

This means every answer considers:

- Power BI field binding through `data: { "name": "dataset" }`
- Deneb runtime constraints
- Power BI tooltip behaviour
- Power BI selection and cross-highlight behaviour (consistently across layers)
- row-context preservation
- Correct pairing of `format` + `formatType` (see Critical gotchas)
- Power BI theme colours via `pbiColor()` and Deneb colour schemes
- performance in Power BI
- clear setup instructions for fields in the Values well
- practical debugging steps

## Default behaviour

When generating a Deneb visual:

1. Ask for missing field names only when absolutely required.
2. If field names are supplied, generate the spec directly.
3. Use Vega-Lite by default.
4. Bind data with:
   ```json
   "data": { "name": "dataset" }
   ```
5. Use this schema unless the user asks for a different version:
   ```json
   "$schema": "https://vega.github.io/schema/vega-lite/v6.json"
   ```
6. Include `mark.tooltip: true` for Power BI tooltip support unless the user says not to.
7. Prefer Power BI-native tooltips over HTML tooltip patterns.
8. When using `pbiFormat`, always include a Power BI format string in `format`. See the cheat sheet above.
9. Use `pbiColor()` or Deneb Power BI colour schemes when a theme-aware visual is requested. Respect a brand palette if the user has one.
10. Keep the visual maintainable. Avoid clever transforms unless they are needed.
11. Explain which fields the user must add to the Deneb visual's Values role.
12. Add `width: "container"` and `height: "container"` alongside `autosize: { type: "fit", contains: "padding" }` for responsive sizing.
13. Apply the `__selected__` opacity pattern consistently across every mark layer (see Critical gotchas).
14. Include a short "How to use this in Deneb" section after the JSON.
15. Include a short "Debug if it fails" section for anything beyond a simple chart.

## Output format

For normal visual generation:

1. One-sentence summary of what the visual does.
2. "Add these fields to Deneb" list (Values well setup).
3. "Paste this into the Deneb specification editor" JSON block.
4. "Notes" with any important assumptions, especially format-string choices and any layers without cross-highlight.
5. "Debug if it fails" checklist.

Do not over-explain Vega-Lite unless the user asks.

## Hard rules for Deneb-safe specs

### Dataset binding

Always:
```json
"data": { "name": "dataset" }
```

Do not use inline `values` unless the user is asking for a standalone example outside Power BI. Do not use remote `url` data sources for Deneb visuals intended for certified Power BI use.

### Field names

Use the exact field names the user provides. Spaces and symbols allowed:
```json
{ "field": "Average Overdue Days", "type": "quantitative" }
```

If a spec fails in Deneb, advise checking Deneb's dataset/debug pane for the actual field names — they sometimes differ from the model property name (e.g., spaces collapsed, or renamed via `displayName`).

### Measures

Deneb needs fields in the Values role. For reliable persistence and Power BI behaviour, include at least one measure in the visual dataset.

### Tooltips

Prefer Power BI-native tooltips:
```json
"mark": { "type": "bar", "tooltip": true }
```

Or an explicit tooltip encoding with correct format pairing:
```json
"tooltip": [
  { "field": "Category", "type": "nominal" },
  { "field": "Sales", "type": "quantitative", "format": "$#,##0", "formatType": "pbiFormat" }
]
```

Do not recommend `vega-tooltip` HTML handlers for Deneb unless the user is building a standalone web embed outside Power BI.

### Row context

If the user wants Power BI report-page tooltips, click selection, drillthrough, or context menu behaviour, avoid transforms that destroy row context.

Transforms to use sparingly (or scope to a single label/annotation layer only):
- `aggregate`
- `fold`
- `pivot`
- `joinaggregate`
- `window`
- `lookup`
- `calculate` that replaces key fields
- synthetic rows

If a transform is required for an annotation, keep the base mark layers free of it so PBI selection and tooltips still work on the underlying rows.

### Cross-highlighting

For Power BI cross-highlighting, use Deneb's highlight pattern:

- A base layer encodes the original measure.
- A highlight layer encodes the auto-generated `Measure__highlight` field.
- Selection/highlight state controls opacity.

Apply the `__selected__` opacity condition to every interactive mark layer (bars, dots, labels, annotations). Inconsistent application looks like a bug.

```json
"opacity": {
  "condition": { "test": { "field": "__selected__", "equal": "off" }, "value": 0.25 },
  "value": 1
}
```

### Interactivity

Vega-Lite parameters work fine inside a visual. For advanced outbound Power BI cross-filtering driven by custom events, use Vega, not Vega-Lite.

### Formatting

Recap of the rule (see Critical gotchas for details):

- d3 format strings → no `formatType`, Vega handles formatting.
- Power BI format strings → set `format` AND `formatType: "pbiFormat"` (or `"pbiFormatAutoUnit"`).
- Never mix: a d3 string with `formatType: "pbiFormat"` renders `undefined`.

Use auto units for axes that need K/M/B abbreviations:
```json
"axis": {
  "format": "$#,##0",
  "formatType": "pbiFormatAutoUnit"
}
```

### Colour and theme

For theme-aware single colour:
```json
"color": { "value": { "expr": "pbiColor(0)" } }
```

For categorical scales, prefer Deneb's Power BI schemes when no brand palette is required:
```json
"scale": { "scheme": "pbiColorNominal" }
```

If the user has a brand palette, hardcode it. Flag explicitly that the visual will not follow theme changes.

For quantitative colour scales over measures that cluster in one range (e.g., gross margin usually 20%–40%), anchor with an explicit `domain` so the gradient stays meaningful:
```json
"color": {
  "field": "GrossMarginPct",
  "type": "quantitative",
  "scale": {
    "domain": [0, 0.25, 0.5],
    "range": ["#FF6F61", "#F0B35A", "#00A6A6"]
  }
}
```

### Layout

For simple single-view or layered visuals:
```json
"width": "container",
"height": "container",
"autosize": { "type": "fit", "contains": "padding" }
```

For faceted, repeated, concatenated, or step-sized visuals, warn about scrolling/overflow and avoid pretending the visual will always resize perfectly.

For donut/arc visuals with text labels at `radius`, leave a margin between `outerRadius` and label `radius` — `radius` greater than `outerRadius` + ~15px risks clipping under `autosize: fit` at tight viewport widths.

### Performance

Warn the user when a design may produce too many marks:

- dense scatter plots
- large text tables
- high-cardinality facets
- many labels
- many layered marks
- SVG-heavy visuals

Suggest:

- reduce fields in the Values role
- filter upstream in Power BI (especially TopN — see Critical gotchas)
- aggregate in the semantic model where possible
- use Canvas renderer for high mark counts
- avoid Auto Apply while editing complex specs

---

## Reviewing existing Deneb visuals

When the user asks for a review or audit of existing visuals, work through this checklist for each:

1. **Schema & dataset binding** — Vega-Lite version, `data: { "name": "dataset" }` present, no inline `values` or remote URLs.
2. **Format/formatType pairing** — every `formatType: "pbiFormat"` has a matching Power BI format string. Flag any `undefined`-risk fields.
3. **Cross-highlight consistency** — `__selected__` opacity applied to every interactive layer, not just one.
4. **Transforms** — flag `window`, `joinaggregate`, `aggregate`, `lookup` that destroy row context for base mark layers (annotation-only layers are fine).
5. **TopN scope** — `window`+`filter` for TopN should usually be a PBI visual filter instead.
6. **Quantitative colour scales** — explicit `domain` set, or flagged as risk.
7. **Layout** — `width`/`height: "container"` set; donut label radius doesn't clip.
8. **Theme awareness** — categorical scales use `pbiColorNominal` OR the user has a documented brand palette.
9. **Sizing-sensitive layers** — text labels, legends with `columns`, faceted/repeated views warned about.
10. **Performance** — mark count and renderer mode appropriate for the data volume.

When fixing issues in `visual.json` files, see the next section.

---

## Editing `visual.json` files in PBIR projects

PBIR `visual.json` files store the Vega spec as a stringified JSON value, doubly-escaped:

```json
"jsonSpec": {
  "expr": {
    "Literal": {
      "Value": "'{...spec...}'"
    }
  }
}
```

- The outer wrapping is single quotes (`'…'`) inside the JSON string.
- Inside the spec string, double quotes are escaped as `\"`.
- Some files (often hand-edited) use literal `\r\n` escape sequences (4 chars: backslash, r, backslash, n) for newlines. Others store the spec as one minified line with no `\r\n` at all.
- When editing with Edit/Write, match the existing escape style. A `\"foo\"` in a multi-line spec is the same value as `\"foo\"` in a minified one, but surrounding whitespace differs.

Practical tips:

- Use the Read tool on the file first to confirm whether the spec is multi-line (`\r\n` escapes between properties) or single-line minified.
- When using `replace_all`, anchor on something unique to the target (e.g., `"title":"Revenue"`) — bare format strings can repeat across multiple fields.
- For the "add a property before existing X" pattern, include the line before X in the match string so you replace `existing_line\nX` with `existing_line\nNEW\nX`.
- The supporting `filterConfig` block at the bottom of `visual.json` controls Power BI's filter pane. Moving a TopN out of the Vega spec into the filter pane means editing both: remove the `window`+`filter` transform AND add a TopN filter object to `filterConfig`. Doing this safely usually means opening the report in Desktop and editing the filter pane visually — the JSON schema for the TopN filter is non-trivial.
- After edits, the spec must still parse. A truncated or unbalanced JSON string in `Value` will silently fail to render in Desktop.

---

## Common visual patterns

Use `resources/visual-patterns.md` for ready-made scaffolds and decision rules.

Use `resources/debugging-checklist.md` when the user reports a broken visual.

Use `resources/prompt-patterns.md` when converting vague user requests into precise Deneb build prompts.

## Quality bar

A good answer is:

- paste-ready
- Power BI-aware
- field-specific
- explicit about `format` + `formatType` pairing
- consistent in cross-highlight behaviour across layers
- clear about assumptions
- concise enough to use while building
- honest about limitations

A poor answer:

- gives generic Vega-Lite with inline sample data
- forgets `dataset`
- pairs d3 format strings with `formatType: "pbiFormat"` (renders `undefined`)
- assumes web-based Vega tooltip handlers work in Deneb
- ignores Power BI formatting altogether
- applies `__selected__` opacity to only some mark layers
- destroys row context without warning
- gives no Values well setup
- gives no debugging advice
