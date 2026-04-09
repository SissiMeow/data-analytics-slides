# Style Presets

Visual style presets provide a `tokens.css` starting point — colors, fonts, and spacing — for new projects.

A style preset defines the **visual feel**, not the page structure. Page structure is defined by [architecture paradigms](../paradigms/).

## How to Add a Style Preset

Create a new directory here named after the style:

```
presets/styles/
├── restrained-professional/
│   ├── README.md           # Description, mood, when to use
│   ├── tokens.css          # Complete tokens file
│   └── preview.html        # Single-slide preview (optional)
├── data-forward/
│   └── ...
└── editorial/
    └── ...
```

### Preset tokens.css Requirements

Must include ALL token categories from [design-system-template.md](../../design-system-template.md):
- Colors (accent, ink, bg, category differentiation, surface)
- Typography (font stacks, type scale with clamp())
- Spacing (space-1 through space-12)
- Layout (slide-padding, board-radius, board-padding)
- Animation (easing curves, durations)

### Preset README.md Template

```markdown
# [Preset Name]

**Mood**: [3-4 adjectives]
**Best for**: [types of reports]

## Colors
- Background: [description]
- Accent: [hex + description]
- Category differentiation: [how it handles A vs B]

## Typography
- Display: [font name] — [why this choice]
- Body: [font name]
- CJK: [font name, if applicable]

## Example Use Cases
- [Report type 1]
- [Report type 2]
```

## Planned Presets

These are style baselines identified from project experience. Add them as needed:

### Restrained Professional
- Warm light background + dark text + 1-2 strong accent colors
- Serif headings + sans-serif data
- For: internal analysis reports, trend studies

### Data-Forward
- Dark background + bright chart colors + high contrast
- Monospace or geometric sans-serif throughout
- For: technical dashboards, real-time data reports

### Editorial
- Generous whitespace + magazine-style typography + fine details
- Serif-dominant with decorative elements
- For: external insight reports, white papers, client deliverables
