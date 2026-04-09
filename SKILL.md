---
name: data-analytics-slides
description: Create modular, data-driven HTML slide decks for analytical reports. Supports data contracts, design system enforcement, structured revisions, and iterative stakeholder review. Use when the user wants to build a data-heavy presentation with charts, color swatches, interactive filtering, or any analytical report delivered as an HTML slide deck.
---

# Data Analytics Slides

Create modular, data-driven HTML slide decks for analytical reports — trend studies, competitive analyses, KPI dashboards — with embedded charts, interactive filtering, and structured data pipelines.

## How This Differs from `frontend-slides`

| | `frontend-slides` | `data-analytics-slides` |
|---|---|---|
| Content type | Text + images | Data charts + interactive filtering |
| Architecture | Single file, one-shot generation | Modular dev → build → merge |
| Data layer | None | Data contracts + auto conversion scripts |
| Design system | Mood → preset picker | .impeccable.md → tokens → components |
| Charts | None | CSS bar, Chart.js line, SVG scatter, swatch gallery |
| Iteration | Generate once, enhance | Multi-round stakeholder review + structured revision |
| Viewport | Same | Same (reuses viewport-base.css) |

Use `frontend-slides` for pitch decks, tutorials, conference talks.
Use `data-analytics-slides` for analytical reports with data pipelines.

---

## Non-Negotiable Rules

1. **File size limit**: No single source file exceeds 500 lines. Split if it does.
2. **Token-first styling**: Never hardcode `font-size`, `padding`, `border-radius`, or color values in page files. Use CSS variables from `tokens.css`.
3. **Data from JSON only**: Page data must come from `data/*.json` files, never hardcoded JS variables. The build script inlines JSON into the final HTML.
4. **Design freeze**: After Phase 2 sign-off, only minor adjustments — no direction-level changes to fonts, colors, or layout paradigm.
5. **One session, one task**: Don't mix design exploration and code implementation in the same AI session.
6. **Git versions, not filenames**: Use `git branch` / `git tag` for versioning. Never create `file-v1.html`, `file-v2.html`.
7. **Viewport fitting**: Every `.slide` must have `height: 100vh; height: 100dvh; overflow: hidden;`. All sizing uses `clamp()`.

---

## Phase 0: Project Kickoff

**Goal**: Establish project infrastructure and align requirements before writing any code.

### Step 0.1: Project Infrastructure

1. Confirm Git repo is initialized
2. Create/verify `.gitignore` (exclude node_modules, .DS_Store, data files, exports)
3. Create/verify `README.md` (project overview, directory structure, build commands)
4. Confirm `.cursor/rules/` constraints are in place

### Step 0.2: Requirements Alignment

Collect and document the following in `docs/project-brief.md`:

**Glossary** — Lock all key terms (these are expensive to change later):
```markdown
## Glossary
- [Term A]: [Definition]  ← use this exact term everywhere
- [Term B]: [Definition]
```

**Sitemap** — Chapter → Page → Core content:
```markdown
## Sitemap
- Cover
- 1. Chapter Name
  - 1.1 Page Title — [brief description, chart type]
  - 1.2 Page Title — [brief description, chart type]
- 2. Chapter Name
  - 2.1 Page Title — [brief description, chart type]
- End
```

**Per-page requirements**:
- Chapter eyebrow (top left)
- Page title
- Takeaway text (below title, summarizing the data insight)
- Main content (chart/table/gallery)
- Footer note (data source, methodology)

**Data sources**: Which notebook/CSV feeds which page.

**Interaction needs**: Which pages need filtering, drilldown, hover tooltips, or Chart.js.

### Step 0.3: Timeline

Apply the 1/3 exploration + 2/3 execution rule:

| Phase | Timing | Milestone |
|-------|--------|-----------|
| Week 0 | Day 1-2 | Kickoff complete, brief signed off |
| Week 1 | Day 3-8 | Data pipeline + design system |
| Week 1 checkpoint | **Day 8** | **Design Freeze** ← critical |
| Week 2-3 | Day 9-22 | Page implementation |
| Week 4 | Day 23-28 | Integration, audit, revision |

Plan Git tags at milestones: `v0.1-data-pipeline`, `v0.2-design-freeze`, `v0.5-first-review`, `v1.0-final`.

---

## Phase 1: Architecture Setup

**Goal**: Establish modular development structure so each page can be developed and previewed independently.

**Read [architecture.md](architecture.md) for the full specification and build script template.**

### Directory Structure

```
project/
├── src/
│   ├── pages/            # Each page is an independently previewable HTML
│   │   ├── cover.html
│   │   ├── summary.html
│   │   ├── 1.2-primary.html
│   │   └── ...
│   ├── styles/
│   │   ├── tokens.css    # Design tokens (colors, fonts, spacing, sizing)
│   │   ├── base.css      # Reset, viewport fitting, scroll-snap, reveal animation
│   │   ├── components.css # Shared UI components (nav, footer, card, filter)
│   │   └── nav.css       # Floating tabs + brand stamp
│   ├── scripts/
│   │   ├── presentation.js  # IntersectionObserver, keyboard nav, scroll handling
│   │   └── chart-utils.js   # Chart.js config helpers (if needed)
│   └── data/
│       ├── contracts/    # Data contract definitions (one per page)
│       └── *.json        # Script-generated page data
├── build.py              # Merge script (~100 lines Python)
└── dist/
    └── output.html       # Build artifact — single-file deliverable
```

### Single-Page Preview

Each page file is a complete, previewable HTML that references shared styles:

```html
<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="../styles/tokens.css">
    <link rel="stylesheet" href="../styles/base.css">
    <link rel="stylesheet" href="../styles/components.css">
</head>
<body>
    <section class="slide" id="slide-xxx">
        <!-- page content -->
    </section>
</body>
</html>
```

### Build Pipeline

`build.py` reads a template, injects each page's `<section>`, merges all CSS into one `<style>` block, merges all JS, inlines JSON data as JS variables, and outputs a single deliverable HTML.

Run: `python build.py` → `dist/output.html`

---

## Phase 2: Design System

**Goal**: Establish a complete token + component system before writing any page content.

**Read [design-system-template.md](design-system-template.md) for the full token skeleton and component patterns.**
**See [reference/](reference/) for the SP26 Color project's extracted design system.**

### Step 2.0: Style Baseline

Choose a starting point for `tokens.css`:

**Option A**: Use an existing style preset from [presets/styles/](presets/styles/) (if available).

**Option B**: Generate from `.impeccable.md` — use the `teach-impeccable` skill to create a design context file, then derive tokens from it.

**Option C**: Fork the [reference implementation](reference/tokens-reference.css) and customize colors/fonts.

### Step 2.1: Design Context

If no `.impeccable.md` exists, create one covering:
- Brand personality (3-4 adjectives)
- Aesthetic direction (warm/cool, minimal/rich, editorial/technical)
- Anti-patterns (what to avoid)
- Font combination (display + body + CJK if needed)
- Color system (brand accent, gender differentiation, semantic colors)

### Step 2.2: tokens.css

Must include ALL of these token categories (no exceptions):

```css
:root {
    /* === Colors === */
    --color-accent: ...;
    --color-ink: ...;
    --color-ink-muted: ...;
    --color-bg: ...;
    /* Gender / category differentiation */
    --color-primary: ...;    /* e.g., women's accent */
    --color-secondary: ...;  /* e.g., men's accent */

    /* === Typography === */
    --font-display: ...;
    --font-body: ...;
    --font-serif: ...;       /* if needed */
    --text-xs: clamp(...);
    --text-sm: clamp(...);
    --text-base: clamp(...);
    --text-lg: clamp(...);
    --text-xl: clamp(...);
    --text-2xl: clamp(...);

    /* === Spacing === */
    --space-1: clamp(...);   /* ~0.25rem */
    --space-2: clamp(...);   /* ~0.5rem */
    --space-3: clamp(...);   /* ~0.75rem */
    --space-4: clamp(...);   /* ~1rem */
    --space-6: clamp(...);   /* ~1.5rem */
    --space-8: clamp(...);   /* ~2rem */

    /* === Layout === */
    --slide-padding: clamp(...);
    --board-radius: clamp(...);
    --board-padding: clamp(...);

    /* === Animation === */
    --ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);
    --duration-normal: 0.6s;
    --duration-fast: 0.25s;
}
```

### Step 2.3: components.css

Extract shared UI elements. Minimum required components:

| Component | Class | Used by |
|-----------|-------|---------|
| Page header | `.slide-header` | Every content page |
| Page footer | `.slide-footer` | Every content page |
| Navigation button | `.nav-btn` | Every content page |
| Content card | `.card` | Chart panels, info cards |
| Chapter eyebrow | `.eyebrow` | Every content page |
| Takeaway block | `.takeaway` | Pages with conclusions |
| Filter select | `.filter-select` | Pages with gender/season filters |
| Reveal animation | `.reveal` | Entrance animations |

### Step 2.4: Skeleton Validation

Create a skeleton HTML with 2-3 empty pages:
- Verify scroll-snap page switching works
- Verify navigation components
- Verify tokens render correctly at different viewports (1280×720, 1920×1080, 768×1024)
- After passing → **Design Freeze** → `git tag vX.Y-design-freeze`

---

## Phase 3: Data Contract

**Goal**: Define the interface between data pipelines (notebooks/CSV) and visualization (HTML/JS), then automate the conversion.

**Read [data-contract-template.md](data-contract-template.md) for the contract format and script template.**

### Step 3.1: Define Per-Page Data Requirements

For each data-driven page, create `data/contracts/page-X.X.json`:

```json
{
  "page": "1.2",
  "description": "Primary Color Overview — bar chart comparing two data sources",
  "sort": "by womenPr descending",
  "fields": {
    "name":   "Color English name (string)",
    "hex":    "Hex code (string, e.g. #DBCCB7)",
    "srcAPr": "Source A Look Presence % (number, 2 decimals)",
    "srcAGrowth": "Source A YoY Growth % (number, 2 decimals)",
    "srcBPr": "Source B Look Presence % (number, 2 decimals)",
    "srcBGrowth": "Source B YoY Growth % (number, 2 decimals)"
  },
  "sources": {
    "srcA": "path/to/source_a.csv → column_name_1, column_name_2",
    "srcB": "path/to/source_b.csv → column_name_1, column_name_2"
  }
}
```

### Step 3.2: Write Conversion Script

`build_data.py`: reads source CSVs → outputs page JSON files per contract.

Requirements:
- Idempotent (same input → same output)
- Validates output against contract fields
- Reports any missing/extra fields
- Outputs to `data/page-X.X.json`

### Step 3.3: Verify

- Run `python build_data.py`
- Inspect output JSON — field names, types, sort order
- Git-diff the JSON to confirm data changes are visible

---

## Phase 4: Page Implementation

**Goal**: Build each page individually, using tokens + components + data contracts.

**Read [chart-patterns.md](chart-patterns.md) for data visualization patterns.**

### Development Order

Low complexity → high complexity:

1. **Cover / End** — Pure display, no data
2. **Summary** — Text only, placeholder content OK initially
3. **Simple data pages** — CSS bar charts, data tables
4. **Medium complexity** — SVG charts, color spectrum grids
5. **High complexity** — Chart.js interactive charts with drilldown
6. **Gallery / large data** — DOM-rendered swatch grids with filtering

### Per-Page Development Cycle

1. Read the page's data contract → confirm JSON is ready
2. Develop in `src/pages/xxx.html` (independently previewable)
3. Use ONLY values from `tokens.css` and classes from `components.css`
4. If a new token is needed → add to `tokens.css`, never hardcode in the page
5. When done → run `build.py` → verify navigation works in merged version
6. `git commit` with descriptive message

### Chart Type Selection Guide

| Need | Recommended | Why |
|------|-------------|-----|
| Static comparison (bar, heatmap) | CSS divs | Zero dependencies, full style control |
| Interactive line/area charts | Chart.js | Built-in hover, tooltip, legend |
| Multi-dimensional visualization | SVG manual | Precise coordinate/path control |
| Color gallery (many swatches) | DOM rendering via JS | Loop generation, efficient filtering |

### Interaction Complexity Budget

Each page gets at most **2 interaction layers**:

| Layer | Example |
|-------|---------|
| Layer 1 | Filter (gender/season dropdown) |
| Layer 2 | Hover highlight / tooltip |

If a 3rd layer is needed (e.g., click-to-drilldown), **write a Design Spec first** before implementing. Avoid using timers to disambiguate click vs double-click — use explicit UI elements (buttons, icons) for advanced actions.

---

## Phase 5: Integration & Audit

**Goal**: Merge all pages, verify the complete deck, and achieve quality targets.

### Step 5.1: Build & Verify

- Run `build.py` → generate single-file HTML
- Verify all page navigation (prev/next buttons, scroll-snap)
- Verify floating tab navigation (chapter bookmarks)
- Verify data renders correctly in merged context
- Verify brand stamp and footer notes

### Step 5.2: Quality Audit

Use the `audit` skill to evaluate. Target: **≥ 16/20**.

Checklist:
- [ ] Viewport fitting: all pages fit at 1280×720 and 1920×1080, no overflow
- [ ] Responsive: at least 3 breakpoints (980px, 760px, 600px)
- [ ] Accessibility: keyboard nav works, aria-labels present
- [ ] Performance: first paint < 2s, no layout shift
- [ ] Design consistency: no hardcoded values outside tokens.css
- [ ] Data accuracy: spot-check 3 data points against source CSV

### Step 5.3: Tag Milestone

```bash
git tag vX.Y-audit-pass -m "Audit score: XX/20"
```

---

## Phase 6: Revision & Delivery

**Goal**: Handle stakeholder feedback efficiently through structured revision.

**Read [revision-workflow.md](revision-workflow.md) for the full process.**

### Step 6.1: Classify Feedback

| Type | Example | Who does it |
|------|---------|-------------|
| **Deterministic** | "Change Color to Colorway everywhere" | Agent auto-executes |
| **Content** | "Add conclusion text below 1.2 title" | Human writes content, agent injects |
| **Directional** | "Redo the gallery layout" | Assess impact, discuss before proceeding |

### Step 6.2: Execute Revisions

- Modify in modular source files (never in the 4000-line merged file)
- Re-run `build.py` → verify → tag: `vX.Y-revision-N`
- For terminology changes: search-and-replace in display text only, skip JS variable names and CSS class names

### Step 6.3: Final Delivery

- Export PDF if needed (Playwright + pdf-lib)
- Tag: `v1.0-final`
- Clean up: remove any leftover archive/ or -vN.html files

---

## Extension Points

### Style Presets

Visual style presets live in [presets/styles/](presets/styles/).
Each preset provides a `tokens.css` starting point (colors, fonts, spacing) — NOT a complete page layout.
See [presets/styles/README.md](presets/styles/README.md) for how to add presets.

### Architecture Paradigms

Report architecture paradigms live in [presets/paradigms/](presets/paradigms/).
Each paradigm defines a page framework (sitemap + per-page component combination + data contract templates).
See [presets/paradigms/README.md](presets/paradigms/README.md) for how to add paradigms.

---

## Supporting Files

| File | Purpose | When to Read |
|------|---------|--------------|
| [architecture.md](architecture.md) | Modular structure, build script template, preview setup | Phase 1 |
| [design-system-template.md](design-system-template.md) | Token categories, component checklist, skeleton template | Phase 2 |
| [data-contract-template.md](data-contract-template.md) | Contract JSON format, conversion script template | Phase 3 |
| [chart-patterns.md](chart-patterns.md) | CSS bar, Chart.js line, SVG scatter, swatch gallery | Phase 4 |
| [revision-workflow.md](revision-workflow.md) | Structured feedback → classified execution | Phase 6 |
| [reference/tokens-reference.css](reference/tokens-reference.css) | SP26 Color project tokens (real example) | Phase 2 |
| [reference/components-reference.css](reference/components-reference.css) | SP26 Color project components (real example) | Phase 2 |
| [presets/styles/](presets/styles/) | Visual style presets (extensible) | Phase 2 Step 2.0 |
| [presets/paradigms/](presets/paradigms/) | Report architecture paradigms (extensible) | Phase 0 Step 0.2 |
