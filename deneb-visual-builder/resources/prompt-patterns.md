# Prompt Patterns for Deneb Visual Generation

Use these patterns to turn vague user requests into reliable Deneb outputs.

## Basic build prompt

"Create a Deneb-safe Vega-Lite spec for Power BI using:
- Category: [field]
- Measure: [field]
- Visual type: [bar/line/scatter/etc.]
- Tooltip: Power BI native
- Formatting: [format]
- Theme: Power BI theme-aware"

## Clarifying questions

Ask only what is needed:
- What are the exact Power BI field names?
- Is the date field a true date or a text label?
- Do you need Power BI report-page tooltips or simple tooltips?
- Does this need cross-highlighting from other visuals?
- Should it match a brand style or the Power BI theme?

## Conversion prompt

When converting a Vega-Lite example to Deneb:

"Convert this Vega-Lite example into Deneb-safe Vega-Lite for Power BI. Replace inline/URL data with `data: { "name": "dataset" }`, map fields to my Power BI fields, use Power BI-native tooltips, and flag any transforms that may break row context."

## Debug prompt

When troubleshooting:

"Review this Deneb spec for Power BI. Check JSON validity, dataset binding, field names, data types, tooltip compatibility, row-context issues, version compatibility, and performance risks. Return a corrected spec and a short explanation."

## Style prompt

When styling:

"Keep this Deneb spec structurally the same, but improve the visual design. Use theme-aware colours where possible, reduce clutter, improve axis labels, add sensible formatting, and keep it Power BI-safe."

## Performance prompt

When optimizing:

"Optimize this Deneb visual for Power BI performance. Reduce unnecessary marks, avoid heavy transforms, recommend SVG vs Canvas, and keep tooltip/selection behaviour working where possible."

## Template prompt

When creating reusable templates:

"Turn this Deneb spec into a reusable pattern. Replace specific fields with clear placeholders, document required fields, list optional fields, and include usage notes for Power BI."
