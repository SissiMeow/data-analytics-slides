# Architecture Paradigms

Report architecture paradigms define the **page framework** — what pages to include, what chart type each page uses, and what data contracts are needed.

A paradigm defines the **report structure**, not the visual style. Visual style is defined by [style presets](../styles/).

## How to Add a Paradigm

Create a new directory here named after the paradigm:

```
presets/paradigms/
├── trend-report/
│   ├── README.md              # Description, sitemap, when to use
│   ├── sitemap.md             # Page framework with chart types
│   ├── contracts/             # Data contract templates per page
│   │   ├── page-overview.json
│   │   ├── page-drilldown.json
│   │   └── page-gallery.json
│   └── page-templates/        # Starter HTML for each page type (optional)
│       ├── overview.html
│       └── gallery.html
├── competitive-deck/
│   └── ...
└── dashboard-deck/
    └── ...
```

### Paradigm README.md Template

```markdown
# [Paradigm Name]

**Best for**: [types of analysis]
**Typical page count**: [range]
**Data sources**: [what kind of input data]

## Sitemap

- Cover
- Summary (text, findings list)
- 1. Chapter: [Name]
  - 1.1 [Page] — [chart type], [interaction type]
  - 1.2 [Page] — [chart type], [interaction type]
- 2. Chapter: [Name]
  - 2.1 [Page] — [chart type], [interaction type]
- End

## Per-Page Components

| Page | Chart Pattern | Filter | Interaction |
|------|--------------|--------|-------------|
| 1.1  | CSS bar      | None   | Hover tooltip |
| 1.2  | Mirror bar   | Gender | Hover tooltip |
| ...  | ...          | ...    | ...          |

## Data Contract Summary

| Page | Input CSV(s) | Output JSON | Key fields |
|------|-------------|-------------|------------|
| 1.1  | source.csv  | page-1.1.json | name, value, growth |
| ...  | ...         | ...           | ...                 |
```

## Planned Paradigms

These are report architectures identified from project experience. Add them as needed:

### Trend Report
- Cover → Summary → Overview comparison → Drilldown details → Monthly trend line → Seasonal cohort → Color gallery → End
- For: fashion color/material trend analysis, seasonal reports
- Key features: mirror bar chart, Chart.js timeline, SVG cluster chart, filterable gallery

### Competitive Deck
- Cover → Executive Summary → Brand-by-Brand comparison → Price matrix → Market position scatter → End
- For: competitive intelligence, market analysis
- Key features: comparison tables, scatter plots, ranking cards

### Dashboard Deck
- Cover → KPI Scorecards → Trend sparklines → Regional heatmap → Detail table → End
- For: operational KPI reports, performance reviews
- Key features: big-number cards, sparkline charts, heatmaps
