# Deneb Visual Builder

A Claude Skill for generating, adapting, debugging, and reviewing **Deneb Vega-Lite visuals for Power BI**.

Designed to catch the gotchas that silently break Deneb visuals in Power BI — the ones that make values render as `undefined`, cross-highlighting feel half-broken, or TopN filters quietly hurt performance.

Repo: [github.com/DuncanBoyneJnr/Deneb-Visual-Skill](https://github.com/DuncanBoyneJnr/Deneb-Visual-Skill)

---

## Why this skill exists

Generic AI-generated Vega-Lite specs look right but break inside Deneb because Deneb sits on top of Power BI, not the open web. This skill encodes the rules that make a spec actually work:

- **`pbiFormat` requires a Power BI format string.** Pair `formatType: "pbiFormat"` with a d3 string like `$,.0f` and every value renders as `undefined`. The skill ships a d3 → Power BI conversion cheat sheet so this stops happening.
- **Cross-highlight opacity has to land on every interactive layer.** A bar chart with a dot overlay needs the `__selected__` condition on both, or selections look broken.
- **TopN belongs in Power BI's filter pane, not in a Vega `window`+`filter` transform.** The skill flags this on review and explains why.
- **PBIR `visual.json` files are doubly-escaped JSON.** The skill documents the escape pattern so direct edits don't break the file.

Plus the usual: dataset binding (`{ "name": "dataset" }`), row-context-safe transforms, theme-aware colours via `pbiColor()` / `pbiColorNominal`, container sizing, and performance tradeoffs.

---

## What you can ask Claude to do

Once installed, the skill activates on requests like:

- "Build a Deneb heatmap of revenue by region and month"
- "Review the Deneb visuals in this report"
- "Why is my Deneb tooltip showing undefined?"
- "Convert this Vega-Lite example to work in Deneb"
- "Add cross-highlighting to this Deneb bar chart"
- "Fix the formatting in `visual.json`"

The skill also handles direct edits to PBIR `*.Report/.../visuals/<name>/visual.json` files, including the `\r\n`-escape vs minified-single-line variants.

---

## Install

Two install paths depending on which Claude product you use.

### Claude Code (CLI / VS Code extension)

Copy the skill folder into your user-level skills directory:

```powershell
# Windows (PowerShell)
git clone https://github.com/DuncanBoyneJnr/Deneb-Visual-Skill.git
Copy-Item -Recurse .\Deneb-Visual-Skill\deneb-visual-builder $env:USERPROFILE\.claude\skills\
```

```bash
# macOS / Linux
git clone https://github.com/DuncanBoyneJnr/Deneb-Visual-Skill.git
cp -r Deneb-Visual-Skill/deneb-visual-builder ~/.claude/skills/
```

Restart Claude Code. The skill is now invocable.

### Claude.ai (web / desktop)

1. Download `deneb-visual-builder.zip` from the [Releases](https://github.com/DuncanBoyneJnr/Deneb-Visual-Skill/releases) page (or build it yourself — see *Building a release ZIP* below).
2. In Claude.ai → Settings → Skills, upload the ZIP.

The ZIP root contains the `deneb-visual-builder/` folder, which is the layout Claude.ai expects.

---

## What's in the package

```
deneb-visual-builder/
├── SKILL.md                          # Main skill instructions (entry point)
├── README.md                         # This file
└── resources/
    ├── visual-patterns.md            # Ready-made Deneb/Vega-Lite scaffolds
    ├── debugging-checklist.md        # Symptom → cause → fix
    └── prompt-patterns.md            # Vague request → precise build prompt
```

The skill loads `SKILL.md` automatically and references the `resources/` files when relevant (e.g., the debugging checklist on a "why is this broken?" question).

---

## Quick example — the `undefined` gotcha

Before (silently broken — every tooltip value shows `undefined`):

```json
{ "field": "Revenue", "type": "quantitative", "formatType": "pbiFormat" }
```

After (the skill writes this by default):

```json
{ "field": "Revenue", "type": "quantitative", "format": "$#,##0", "formatType": "pbiFormat" }
```

The skill's `SKILL.md` opens with five "Critical gotchas" — read first, before the visual patterns or the debugging checklist — covering this and four other failure modes that don't show up as errors, only as wrong output.

---

## Building a release ZIP

From the repo root:

```powershell
# Windows (PowerShell)
Compress-Archive -Path .\deneb-visual-builder -DestinationPath .\deneb-visual-builder.zip -Force
```

```bash
# macOS / Linux
zip -r deneb-visual-builder.zip deneb-visual-builder
```

Upload the resulting `deneb-visual-builder.zip` to Claude.ai.

---

## Updating the skill

This skill is designed to grow as new Deneb / Power BI gotchas surface. If you hit a failure mode this skill didn't catch:

1. Add the rule to the relevant section of `SKILL.md` (usually "Critical gotchas" or "Hard rules").
2. If it's a symptom-driven failure, add it to `resources/debugging-checklist.md`.
3. If it's a reusable spec scaffold, add it to `resources/visual-patterns.md`.

Open a PR or issue at [github.com/DuncanBoyneJnr/Deneb-Visual-Skill](https://github.com/DuncanBoyneJnr/Deneb-Visual-Skill).

---

## Credits

Maintained by [Duncan Boyne](https://github.com/DuncanBoyneJnr). Built on the work of the Deneb team ([Daniel Marsh-Patrick](https://deneb-viz.github.io/)) and the broader Power BI / Vega-Lite community.
