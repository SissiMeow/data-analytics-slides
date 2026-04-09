# Design System Template

How to establish a complete design system for a data analytics slide deck.

## Prerequisites

- `.impeccable.md` or equivalent design context file
- Project brief with sitemap and per-page requirements

## Step 1: Token Categories Checklist

Your `tokens.css` MUST include all of these categories. No exceptions — missing categories lead to hardcoded values that break consistency later.

### Colors

```css
:root {
    /* Brand accent — used for eyebrows, highlights, active states */
    --color-accent: ;

    /* Ink — primary text colors */
    --color-ink: ;           /* darkest, headings */
    --color-ink-dark: ;      /* secondary headings */
    --color-ink-muted: ;     /* body text, descriptions */

    /* Background */
    --color-bg: ;            /* solid fallback */
    --color-bg-gradient: ;   /* preferred background */

    /* Category differentiation (e.g., gender, region, source) */
    --color-primary: ;       /* category A accent */
    --color-secondary: ;     /* category B accent */

    /* Surface (cards, panels) */
    --color-surface: ;       /* card background */
    --color-surface-border: ;
    --color-surface-shadow: ;

    /* Structural lines */
    --color-grid: ;          /* decorative grid lines */
    --color-divider: ;       /* content dividers */
}
```

**Decision guide**: Is the data categorized by a binary dimension (gender, source A vs B)? Define `--color-primary` and `--color-secondary`. If more than 2 categories, define `--color-cat-1` through `--color-cat-N`.

### Typography

```css
:root {
    /* Font stacks — always include CJK fallback if bilingual */
    --font-display: ;   /* headings, numbers, navigation */
    --font-body: ;      /* body text, descriptions */
    --font-serif: ;     /* decorative headings, if needed */

    /* Type scale — MUST use clamp() for every size */
    --text-xs:   clamp(0.5rem,  0.65vw, 0.64rem);  /* footnotes, labels */
    --text-sm:   clamp(0.6rem,  0.8vw,  0.76rem);  /* captions, metadata */
    --text-base: clamp(0.75rem, 1vw,    0.98rem);   /* body text */
    --text-lg:   clamp(0.85rem, 1.2vw,  1.1rem);   /* subheadings */
    --text-xl:   clamp(1rem,    1.45vw, 1.3rem);    /* section titles */
    --text-2xl:  clamp(1.4rem,  2.4vw,  2.2rem);   /* page titles */
    --text-3xl:  clamp(2rem,    4vw,    3.6rem);    /* hero numbers */
}
```

**Font selection tips**:
- Display font: needs to look good at large sizes and in data contexts. Manrope, DM Sans, Instrument Sans, Plus Jakarta Sans are good options.
- Body font: needs to be readable at small sizes. Noto Sans SC for CJK, DM Sans for Latin.
- Serif font: optional, for editorial feel. Cormorant Garamond, Noto Serif SC, Playfair Display.
- **Avoid**: Inter, Roboto, Arial, system fonts — they read as generic AI output.

### Spacing

```css
:root {
    --space-1:  clamp(0.15rem, 0.3vw,  0.25rem);   /* tight gaps */
    --space-2:  clamp(0.25rem, 0.5vw,  0.4rem);
    --space-3:  clamp(0.35rem, 0.7vw,  0.6rem);
    --space-4:  clamp(0.5rem,  1vw,    0.8rem);     /* standard gap */
    --space-6:  clamp(0.8rem,  1.5vw,  1.2rem);
    --space-8:  clamp(1rem,    2vw,    1.6rem);      /* section gap */
    --space-10: clamp(1.4rem,  2.8vh,  2.2rem);
    --space-12: clamp(1.5rem,  3vh,    2.5rem);      /* large gap */
}
```

### Layout

```css
:root {
    --slide-padding: clamp(0.8rem, 2.5vw, 2.2rem);
    --board-radius:  clamp(1rem, 2vw, 1.6rem);
    --board-padding: clamp(0.8rem, 1.8vw, 1.6rem);
}
```

### Animation

```css
:root {
    --ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);
    --duration-normal: 0.6s;   /* entrance, card transitions */
    --duration-fast: 0.25s;    /* hover, active states */
}
```

## Step 2: Component Extraction

Scan the sitemap and identify UI elements that appear on 2+ pages. Each becomes a component in `components.css`.

### Minimum Required Components

| Component | When to use | CSS class |
|-----------|-------------|-----------|
| Slide container | Every page | `.slide`, `.slide-content` |
| Page header | Pages with eyebrow + title + nav | `.slide-header`, `.header-actions` |
| Page footer | Pages with source notes | `.slide-footer`, `.footer-note` |
| Navigation button | Page-to-page navigation | `.nav-btn` |
| Content card | Wrapping chart panels | `.card` |
| Chapter eyebrow | Chapter label above title | `.eyebrow` |
| Takeaway text | Insight summary below title | `.takeaway` |
| Filter dropdown | Gender / season / region filtering | `.filter-group`, `.filter-select` |
| Reveal animation | Scroll-triggered entrance | `.reveal` |
| Floating tabs | Chapter bookmark navigation | `.float-tabs`, `.float-tab` |
| Brand stamp | Bottom-left brand watermark | `.brand-stamp` |

### Optional Components (add as needed)

| Component | When to use | CSS class |
|-----------|-------------|-----------|
| Color dot | Inline color swatch in text | `.cd` |
| Tooltip/popup | Hover data display | `.popup` |
| Rank badge | Top-N indicators | `.rank` |
| Metric strip | PR% / YoY% column headers | `.metric-strip` |

## Step 3: Skeleton Validation

Create a test HTML with 2-3 empty pages to validate the design system works:

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Design System Skeleton Test</title>
    <link rel="stylesheet" href="styles/tokens.css">
    <link rel="stylesheet" href="styles/base.css">
    <link rel="stylesheet" href="styles/components.css">
    <link rel="stylesheet" href="styles/nav.css">
</head>
<body>
    <nav class="float-tabs">
        <button class="float-tab active">Chapter 1</button>
        <button class="float-tab">Chapter 2</button>
    </nav>
    <div class="brand-stamp">Brand Name</div>

    <section class="slide" id="slide-test-1">
        <div class="slide-content">
            <header class="slide-header reveal">
                <div>
                    <span class="eyebrow">1. Chapter Name</span>
                    <h2 class="title">Test Page Title</h2>
                    <p class="takeaway">This is the takeaway text.</p>
                </div>
                <div class="header-actions">
                    <button class="nav-btn"><span class="material-symbols-outlined">arrow_forward</span></button>
                </div>
            </header>
            <div class="card reveal">
                <p>Card content placeholder</p>
            </div>
            <footer class="slide-footer reveal">
                <div class="footer-note"><strong>Source:</strong> Test data source</div>
            </footer>
        </div>
    </section>

    <section class="slide" id="slide-test-2">
        <div class="slide-content">
            <header class="slide-header reveal">
                <div>
                    <span class="eyebrow">2. Second Chapter</span>
                    <h2 class="title">Another Test Page</h2>
                </div>
            </header>
            <div class="card reveal">
                <p>Another card</p>
            </div>
        </div>
    </section>
</body>
</html>
```

### Validation Checklist

- [ ] Scroll-snap switches between pages smoothly
- [ ] Reveal animations trigger on scroll
- [ ] All text sizes scale with viewport
- [ ] Card panels have correct radius, padding, shadow
- [ ] Footer stays at bottom without overflowing
- [ ] Floating tabs show/highlight correctly
- [ ] Works at 1280×720, 1920×1080, and 768×1024
- [ ] `prefers-reduced-motion` disables animations

After all checks pass → **Design Freeze**.
