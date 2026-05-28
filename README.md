# Deneb Visual Builder

A Claude Skill for generating, adapting, debugging, and reviewing **Deneb Vega-Lite visuals for Power BI**.

Once installed, Claude writes Deneb specs that drop into Power BI without rework — correctly bound data, Power BI-native tooltips, theme-aware colours, working cross-highlighting, and consistent formatting.

Repo: [github.com/DuncanBoyneJnr/Deneb-Visual-Skill](https://github.com/DuncanBoyneJnr/Deneb-Visual-Skill)

---

## What the skill gives you

A Claude that knows how Deneb actually behaves inside Power BI, not just generic Vega-Lite:

- **Power BI–shaped specs by default.** Dataset binding (`{ "name": "dataset" }`), Power BI tooltips, container sizing, and `__selected__` cross-highlight wiring applied consistently across every layer.
- **Power BI format strings done right.** Currency, percent, signed percent, and auto-units (K/M/B) all use the correct `format` + `formatType` pairing so values render properly first time.
- **Theme-aware colours.** `pbiColor()` and `pbiColorNominal` for visuals that follow the report theme; explicit brand palettes when you want fixed colours.
- **Row-context-safe transforms.** Knows which transforms break Power BI selection, drillthrough, and report-page tooltips — and which only need to be scoped to a label/annotation layer.
- **Direct edits to PBIR `visual.json`.** Understands the doubly-escaped spec format inside `Literal.Value` and the `\r\n`-escape vs minified-single-line variants.
- **Audit mode.** Run a 10-point review checklist over the Deneb visuals in an existing report and get back a prioritised fix list.

---

## What you can ask Claude to do

Once installed, the skill activates on requests like:

- "Build a Deneb heatmap of revenue by region and month"
- "Review the Deneb visuals in this report"
- "Convert this Vega-Lite example to work in Deneb"
- "Add cross-highlighting to this Deneb bar chart"
- "Format the tooltip on this measure as currency"
- "Fix the formatting in `visual.json`"

It works equally well for greenfield specs and for editing existing PBIR projects.

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
└── resources/
    ├── visual-patterns.md            # Ready-made Deneb/Vega-Lite scaffolds
    ├── debugging-checklist.md        # Symptom → cause → fix
    └── prompt-patterns.md            # Vague request → precise build prompt
```

The skill loads `SKILL.md` automatically and pulls in the `resources/` files when relevant (e.g., the debugging checklist on a "why is this broken?" question, the patterns on a "build me a…" request).

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

## Contributing

The skill is built to grow as new Deneb / Power BI patterns and edge cases come up. Useful additions:

- New scaffolds for `resources/visual-patterns.md` (new chart types, new design treatments).
- New symptom → cause → fix entries for `resources/debugging-checklist.md`.
- New rules for `SKILL.md` when something behaves differently inside Deneb than in raw Vega-Lite.

Open a PR or issue at [github.com/DuncanBoyneJnr/Deneb-Visual-Skill](https://github.com/DuncanBoyneJnr/Deneb-Visual-Skill).

---

## Credits

Maintained by [Duncan Boyne](https://github.com/DuncanBoyneJnr). Built on the work of the Deneb team ([Daniel Marsh-Patrick](https://deneb-viz.github.io/)) and the broader Power BI / Vega-Lite community.
