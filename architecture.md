# Modular Architecture

## Directory Structure

```
project-root/
├── .gitignore
├── README.md
├── docs/
│   ├── project-brief.md        # Glossary, sitemap, per-page requirements
│   └── session-handoff.md      # AI session context transfer
├── src/
│   ├── pages/
│   │   ├── cover.html          # Cover slide
│   │   ├── summary.html        # Summary / TOC
│   │   ├── 1.1-page-name.html  # Chapter 1 pages
│   │   ├── 1.2-page-name.html
│   │   ├── 2.1-page-name.html  # Chapter 2 pages
│   │   └── end.html            # End slide
│   ├── styles/
│   │   ├── tokens.css          # Design tokens only (colors, fonts, spacing)
│   │   ├── base.css            # Reset, scroll-snap, viewport, reveal animation
│   │   ├── components.css      # Shared UI components
│   │   └── nav.css             # Floating tabs + brand stamp
│   ├── scripts/
│   │   ├── presentation.js     # IntersectionObserver, keyboard/touch nav
│   │   └── chart-utils.js      # Chart.js config helpers (optional)
│   └── data/
│       ├── contracts/           # Data contract definitions
│       │   ├── page-1.1.json
│       │   └── page-1.2.json
│       └── page-1.1.json       # Generated JSON data (from build_data.py)
├── build.py                    # Merge modules → single HTML
├── build_data.py               # CSV → JSON conversion (data contracts)
└── dist/
    └── output.html             # Build artifact
```

## Page File Template

Each page file is independently previewable in the browser:

```html
<!-- src/pages/1.2-primary-overview.html -->
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>1.2 Primary Overview — Dev Preview</title>
    <!-- Shared styles -->
    <link rel="stylesheet" href="../styles/tokens.css">
    <link rel="stylesheet" href="../styles/base.css">
    <link rel="stylesheet" href="../styles/components.css">
    <!-- Page-specific fonts (also declared in tokens.css) -->
    <link href="https://fonts.googleapis.com/css2?family=..." rel="stylesheet">
</head>
<body>
    <section class="slide" id="slide-1-2">
        <div class="slide-content">
            <header class="slide-header reveal">
                <div>
                    <span class="eyebrow">1. Chapter Name</span>
                    <h2 class="title">Page Title</h2>
                    <p class="takeaway">Key insight text goes here.</p>
                </div>
                <div class="header-actions">
                    <button class="nav-btn" onclick="console.log('navigate to 1.1')" title="Previous">
                        <span class="material-symbols-outlined">arrow_back</span>
                    </button>
                    <button class="nav-btn" onclick="console.log('navigate to 1.3')" title="Next">
                        <span class="material-symbols-outlined">arrow_forward</span>
                    </button>
                </div>
            </header>

            <div class="card reveal">
                <!-- Chart / data content -->
            </div>

            <footer class="slide-footer reveal">
                <div class="footer-note">
                    <strong>Source:</strong> Data source description
                </div>
            </footer>
        </div>
    </section>

    <!-- Dev-only: load data for preview -->
    <script>
        // In dev mode, load JSON directly
        // In production, build.py inlines this as a JS variable
        fetch('../data/page-1.2.json')
            .then(r => r.json())
            .then(data => renderChart(data));
    </script>
</body>
</html>
```

During development, navigation buttons log to console. The build script replaces them with real `scrollIntoView` calls.

## Build Script Template

```python
#!/usr/bin/env python3
"""
build.py — Merge modular pages into a single deliverable HTML.

Usage: python build.py [--output dist/output.html]
"""

import os
import re
import json
import argparse
from pathlib import Path

SRC = Path("src")
DIST = Path("dist")

# Page order — defines the slide sequence
PAGE_ORDER = [
    "cover.html",
    "summary.html",
    "1.1-page-name.html",
    "1.2-page-name.html",
    # ... add pages here
    "end.html",
]

def read_file(path):
    return Path(path).read_text(encoding="utf-8")

def extract_section(html):
    """Extract the <section class='slide'> block from a page file."""
    match = re.search(
        r'(<section\s+class="slide"[^>]*>.*?</section>)',
        html, re.DOTALL
    )
    return match.group(1) if match else ""

def extract_page_script(html):
    """Extract page-specific <script> content (skip dev-only fetch blocks)."""
    scripts = re.findall(r'<script>(.*?)</script>', html, re.DOTALL)
    page_scripts = []
    for s in scripts:
        if 'fetch(' not in s and 'Dev-only' not in s:
            page_scripts.append(s.strip())
    return "\n\n".join(page_scripts)

def collect_styles():
    """Read and concatenate all CSS files."""
    style_order = ["tokens.css", "base.css", "components.css", "nav.css"]
    css_parts = []
    for name in style_order:
        path = SRC / "styles" / name
        if path.exists():
            css_parts.append(f"/* === {name} === */\n{read_file(path)}")
    return "\n\n".join(css_parts)

def collect_scripts():
    """Read and concatenate all JS files."""
    js_parts = []
    for js_file in sorted((SRC / "scripts").glob("*.js")):
        js_parts.append(f"// === {js_file.name} ===\n{read_file(js_file)}")
    return "\n\n".join(js_parts)

def inline_data():
    """Read JSON data files and convert to JS variable declarations."""
    data_dir = SRC / "data"
    js_lines = []
    for json_file in sorted(data_dir.glob("page-*.json")):
        var_name = json_file.stem.replace("-", "_").replace(".", "_").upper()
        data = json.loads(read_file(json_file))
        js_lines.append(f"const {var_name} = {json.dumps(data, ensure_ascii=False)};")
    return "\n".join(js_lines)

def collect_font_links():
    """Extract Google Fonts <link> tags from tokens.css comments or a config."""
    tokens = read_file(SRC / "styles" / "tokens.css")
    match = re.search(r'/\*\s*FONT_LINKS:\s*(.*?)\*/', tokens, re.DOTALL)
    if match:
        return match.group(1).strip()
    return '<!-- Add font links here -->'

def build(output_path):
    DIST.mkdir(exist_ok=True)

    css = collect_styles()
    js_shared = collect_scripts()
    js_data = inline_data()

    sections = []
    page_scripts = []
    for page_name in PAGE_ORDER:
        page_path = SRC / "pages" / page_name
        if not page_path.exists():
            print(f"⚠️  Missing page: {page_name}")
            continue
        html = read_file(page_path)
        section = extract_section(html)
        if section:
            sections.append(f"    <!-- === {page_name} === -->\n    {section}")
        ps = extract_page_script(html)
        if ps:
            page_scripts.append(f"    // === {page_name} ===\n{ps}")

    font_links = collect_font_links()

    template = f"""<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Report Title</title>
    {font_links}
    <style>
{css}
    </style>
</head>
<body>
    <!-- Floating tabs — chapter navigation -->
    <nav class="float-tabs" aria-label="Chapter navigation">
        <!-- build.py can auto-generate these from PAGE_ORDER -->
    </nav>

    <!-- Brand stamp -->
    <div class="brand-stamp">Brand Name</div>

{"chr(10)".join(sections)}

    <script>
    // === Data ===
    {js_data}

    // === Shared Scripts ===
    {js_shared}

    // === Page Scripts ===
    {"chr(10)".join(page_scripts)}
    </script>
</body>
</html>"""

    Path(output_path).write_text(template, encoding="utf-8")
    line_count = template.count('\n') + 1
    print(f"✅ Built {output_path} ({line_count} lines, {len(sections)} pages)")

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--output", default="dist/output.html")
    args = parser.parse_args()
    build(args.output)
```

## Navigation Injection

In dev mode, page files use placeholder navigation:
```javascript
onclick="console.log('navigate to 1.1')"
```

The build script can post-process navigation buttons to inject real scroll targets:
```javascript
onclick="document.getElementById('slide-1-1').scrollIntoView({behavior:'smooth'})"
```

Alternatively, use `data-target` attributes and let `presentation.js` handle all navigation via event delegation:
```html
<button class="nav-btn" data-target="slide-1-1">
```

## File Size Monitoring

After building, check the output size:

```bash
wc -l dist/output.html
```

If the merged file exceeds ~5000 lines, review whether:
- CSS has redundant per-page definitions that should be components
- JS has duplicated chart setup that should be utility functions
- Data JSON is unnecessarily verbose (trim decimal precision, remove unused fields)
