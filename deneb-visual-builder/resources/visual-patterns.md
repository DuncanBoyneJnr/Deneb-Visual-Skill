# Deneb Visual Patterns

## 1. Basic categorical bar chart

Use when the user wants a measure by category.

Required fields:
- category field
- measure

Pattern:

```json
{
  "$schema": "https://vega.github.io/schema/vega-lite/v6.json",
  "data": { "name": "dataset" },
  "width": "container",
  "height": "container",
  "autosize": { "type": "fit", "contains": "padding" },
  "mark": { "type": "bar", "tooltip": true },
  "encoding": {
    "y": {
      "field": "Category",
      "type": "nominal",
      "sort": "-x",
      "axis": { "title": null }
    },
    "x": {
      "field": "Measure",
      "type": "quantitative",
      "axis": {
        "title": null,
        "format": "#,##0",
        "formatType": "pbiFormat"
      }
    },
    "color": { "value": { "expr": "pbiColor(0)" } }
  },
  "config": {
    "view": { "stroke": null },
    "font": "Segoe UI, Arial, sans-serif"
  }
}
```

## 2. Time-series line chart

Use for trends over time.

Required fields:
- date field
- measure
- optional series field

Tips:
- Use `temporal` for true date fields.
- Use `ordinal` for pre-formatted year-month labels.
- If using YearMonth text, sort by a numeric sort field if the user provides one.

## 3. Label overlay

Use a `layer` with a bar layer and text layer.

Avoid labels when there are many categories.

## 4. KPI bullet chart

Use for actual vs target.

Required fields:
- category or KPI name
- actual measure
- target measure
- optional low/mid/high ranges

Usually use layered `bar`, `rule`, and `text`.

## 5. Heatmap

Use `rect` with `x`, `y`, and `color`.

Required fields:
- x category/date
- y category
- colour measure

Tips:
- Use `ordinal` for month labels unless using actual dates.
- Use explicit sort fields if available.
- Use tooltips because colour alone is not enough.

## 6. Scatter plot

Required fields:
- x measure
- y measure
- detail/category field for row identity
- optional colour field
- optional size measure

Important:
- Add a `detail` field to preserve point identity.
- Warn about dense datasets.

## 7. Faceted small multiples

Use when the user wants the same chart split by a category.

Required fields:
- facet field
- measure
- x/y field depending on chart type

Warn:
- Facets can overflow in Deneb.
- Limit panels or apply filters.
- Consider `columns`.

## 8. Cross-highlight bar chart

Use only when the user asks for Power BI cross-highlighting.

Required fields:
- category
- measure
- generated Deneb highlight measure field, normally `Measure__highlight`

Concept:

```json
{
  "$schema": "https://vega.github.io/schema/vega-lite/v6.json",
  "data": { "name": "dataset" },
  "layer": [
    {
      "mark": { "type": "bar", "opacity": 0.3, "tooltip": true },
      "encoding": {
        "x": { "field": "Measure", "type": "quantitative" }
      }
    },
    {
      "mark": { "type": "bar", "tooltip": true },
      "encoding": {
        "x": { "field": "Measure__highlight", "type": "quantitative" }
      }
    }
  ],
  "encoding": {
    "y": { "field": "Category", "type": "nominal", "sort": "-x" },
    "color": { "value": { "expr": "pbiColor(0)" } }
  }
}
```

## 9. Debug datum tooltip

Use only while debugging:

```json
"tooltip": { "content": "data" }
```

Remove this before final delivery unless the user wants a diagnostic spec.

## 10. PBIR fragment support

If the user asks for PBIR output, provide:
- the Deneb visual type/GUID if known
- `jsonSpec`
- `jsonConfig`
- settings such as tooltip/context menu/highlight booleans

If uncertain, say the PBIR wrapper may need adjustment for their report version and provide the spec separately.
