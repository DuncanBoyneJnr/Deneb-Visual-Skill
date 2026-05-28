# Deneb Debugging Checklist

Use this when a Deneb visual does not render, renders incorrectly, or behaves differently from Power BI native visuals.

## First checks

1. Is Deneb using Vega-Lite or Vega?
2. Does the spec schema match the provider?
3. Is the data bound as `data: { "name": "dataset" }`?
4. Are all referenced fields added to the Deneb Values role?
5. Is at least one measure present?
6. Do the field names in the spec exactly match Deneb's dataset fields?
7. Are there JSON syntax errors?
8. Is the visual using a feature from a newer Vega-Lite version than Deneb embeds?

## If values display as "undefined"

Almost always a `formatType` / `format` mismatch.

Check:
- Every `"formatType": "pbiFormat"` and `"formatType": "pbiFormatAutoUnit"` has a matching Power BI `format` string. Bare `formatType` with no `format` renders `undefined`.
- The `format` value is a Power BI format string, NOT a d3 format string. `$,.0f` is d3; `$#,##0` is Power BI.
- The field is a real Power BI measure or column. Calculated/derived fields from Vega transforms can't use `pbiFormat`.

Quick conversion table:

| d3 string (no formatType) | Power BI string (`pbiFormat`) |
|---|---|
| `$,.0f` | `$#,##0` |
| `,.0f` | `#,##0` |
| `.0%` | `0%` |
| `.1%` | `0.0%` |
| `+.0%` | `+0%;-0%;0%` |
| `+.1%` | `+0.0%;-0.0%;0.0%` |

For SI auto-units (`$,.0s`), use `format: "$#,##0"` + `formatType: "pbiFormatAutoUnit"`.

## If nothing renders

Check:
- missing field names
- wrong data source name
- invalid JSON
- wrong mark type
- wrong data type
- transform references to fields that do not exist
- scale domain filtering out all data

## If axes are wrong

Check:
- `nominal`, `ordinal`, `quantitative`, `temporal`
- sort field availability
- date fields coming through as text
- aggregation happening twice
- Power BI summarisation settings

## If tooltips do not work

Check:
- `mark.tooltip` is true or tooltip encoding exists
- row context has not been destroyed by transforms
- the tooltip fields are in the Values role
- report-page tooltip is configured in Power BI
- the mark being hovered maps to a real dataset row

## If cross-highlighting does not work

Check:
- cross-highlight values are exposed in Deneb settings
- the highlight field exists, usually `[Measure]__highlight`
- the highlight layer references the correct field
- the visual is not using transformed data that loses row mapping

## If only some marks dim during selection

A layered visual where bars dim but dots/labels stay opaque usually means the `__selected__` opacity condition is missing from those layers.

Check:
- Every interactive mark layer has the opacity condition in its `encoding`, not just on `mark`:
  ```json
  "opacity": {
    "condition": { "test": { "field": "__selected__", "equal": "off" }, "value": 0.25 },
    "value": 1
  }
  ```
- A flat `mark.opacity: 0.95` ignores selection state — move it into `encoding.opacity` with the condition.
- Text/label layers also need it. Use a lower `value` (e.g., 0.2) for text so it dims more aggressively.

## If the visual is slow

Check:
- row count
- mark count
- text label count
- number of layers
- number of facets
- SVG vs Canvas renderer
- whether Auto Apply is enabled while editing

## If layout overflows

Check:
- explicit width/height
- `step` sizing
- facets/repeats/concat
- long labels
- too many categories
- Power BI visual container size

## Safe fallback spec

When debugging, reduce to:

```json
{
  "$schema": "https://vega.github.io/schema/vega-lite/v6.json",
  "data": { "name": "dataset" },
  "mark": { "type": "point", "tooltip": { "content": "data" } },
  "encoding": {
    "x": { "field": "Field1", "type": "nominal" },
    "y": { "field": "Measure1", "type": "quantitative" }
  }
}
```

Then build back up one layer at a time.
