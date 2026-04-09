# Chart Patterns

Data visualization patterns for analytical slide decks. Each pattern includes HTML structure, CSS, and JS where needed. All patterns are extracted from production implementations.

## Pattern Selection Guide

| Data type | Recommended pattern | Dependencies |
|-----------|-------------------|--------------|
| Two-source comparison | CSS Mirror Bar Chart | None |
| Ranking / distribution | CSS Horizontal Bar Chart | None |
| Time series / trend | Chart.js Line Chart | Chart.js CDN |
| Multi-dimensional clusters | SVG Coordinate Chart | None |
| Color palette display | DOM Swatch Gallery | None |
| Summary statistics | CSS Data Table | None |

**Rule of thumb**: Use CSS for static display, Chart.js for interactive time series, SVG for custom coordinate systems, DOM rendering for large filterable datasets.

---

## 1. CSS Mirror Bar Chart

Two-source comparison with bars growing from a center axis. Used for comparing e.g., Runway vs RedNote data side by side.

### HTML Structure

```html
<div class="card">
    <!-- Header row -->
    <div class="mirror-topline">
        <span class="panel-side-title" style="color: var(--color-primary)">SOURCE A</span>
        <span class="panel-center-title">COLOR</span>
        <span class="panel-side-title" style="color: var(--color-secondary)">SOURCE B</span>
    </div>

    <!-- Metric labels -->
    <div class="mirror-subline">
        <div class="metric-strip">
            <span>YOY%</span>
            <span>LOOK PRESENCE %</span>
        </div>
        <div></div>
        <div class="metric-strip metric-right">
            <span>LOOK PRESENCE %</span>
            <span>YOY%</span>
        </div>
    </div>

    <!-- Data rows -->
    <div class="mirror-grid" id="mirrorChart">
        <!-- JS generates rows here -->
    </div>
</div>
```

### CSS

```css
.mirror-topline,
.mirror-subline {
    display: grid;
    grid-template-columns: minmax(0, 1fr) var(--center-width) minmax(0, 1fr);
    align-items: center;
    gap: var(--space-4);
}

.mirror-topline {
    padding-bottom: var(--space-2);
    border-bottom: 1px solid var(--color-divider);
}

.mirror-grid {
    display: grid;
    grid-template-columns: minmax(0, 1fr) var(--center-width) minmax(0, 1fr);
    grid-template-rows: repeat(var(--row-count, 11), minmax(var(--row-height), 1fr));
    gap: var(--space-1) var(--space-4);
    min-height: 0;
    flex: 1;
}

.row-side {
    display: grid;
    align-items: center;
    min-width: 0;
}

.row-side.left {
    grid-template-columns: var(--growth-col) minmax(0, 1fr);
    gap: var(--space-3);
}

.row-side.right {
    grid-template-columns: minmax(0, 1fr) var(--growth-col);
    gap: var(--space-3);
}

.bar-track {
    position: relative;
    display: flex;
    align-items: center;
    width: 100%;
    height: var(--bar-height);
    border-radius: 999px;
    background: rgba(169, 153, 140, 0.15);
}

.bar-track.left { justify-content: flex-end; }

.bar-fill {
    height: 100%;
    border-radius: inherit;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 0 var(--space-3);
    font-size: var(--text-sm);
    font-weight: 700;
    white-space: nowrap;
    color: rgba(255, 253, 249, 0.85);
    transition: all 0.3s var(--ease-out-expo);
}

.bar-fill:hover {
    transform: scale(1.05);
    filter: brightness(1.1);
}

/* For light-colored bars, use dark text */
.bar-fill.is-light {
    color: rgba(21, 18, 16, 0.65);
    border: 1px solid rgba(71, 56, 43, 0.18);
}

.center-node {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: var(--space-2);
}

.center-swatch {
    width: clamp(1rem, 1.2vw, 1.2rem);
    height: clamp(1rem, 1.2vw, 1.2rem);
    border-radius: 0.24rem;
    border: 1px solid rgba(71, 56, 43, 0.18);
}
```

### JS (row generation)

```javascript
function renderMirrorChart(data, maxPr) {
    const grid = document.getElementById('mirrorChart');
    grid.innerHTML = '';

    data.forEach(item => {
        // Left side (Source A)
        const left = document.createElement('div');
        left.className = 'row-side left';
        left.innerHTML = `
            <span class="growth-value" style="color: ${item.srcAGrowth >= 0 ? 'var(--color-primary)' : 'inherit'}">
                ${item.srcAGrowth >= 0 ? '+' : ''}${item.srcAGrowth}%
            </span>
            <div class="bar-track left">
                <div class="bar-fill ${isLightColor(item.hex) ? 'is-light' : ''}"
                     style="width: ${(item.srcAPr / maxPr * 100)}%; background: ${item.hex}">
                    ${item.srcAPr}%
                </div>
            </div>`;

        // Center (color swatch + name)
        const center = document.createElement('div');
        center.className = 'center-node';
        center.innerHTML = `
            <div class="center-swatch" style="background: ${item.hex}"></div>
            <span class="center-label">${item.name}</span>`;

        // Right side (Source B)
        const right = document.createElement('div');
        right.className = 'row-side right';
        right.innerHTML = `
            <div class="bar-track">
                <div class="bar-fill ${isLightColor(item.hex) ? 'is-light' : ''}"
                     style="width: ${(item.srcBPr / maxPr * 100)}%; background: ${item.hex}">
                    ${item.srcBPr}%
                </div>
            </div>
            <span class="growth-value" style="color: ${item.srcBGrowth >= 0 ? 'var(--color-secondary)' : 'inherit'}">
                ${item.srcBGrowth >= 0 ? '+' : ''}${item.srcBGrowth}%
            </span>`;

        grid.append(left, center, right);
    });
}
```

---

## 2. Chart.js Line Chart

Interactive time-series with hover tooltips and legend filtering. Requires Chart.js CDN.

### Setup

```html
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.7/dist/chart.umd.min.js"></script>
```

### Standard Configuration

```javascript
function createLineChart(canvasId, datasets, labels) {
    const ctx = document.getElementById(canvasId).getContext('2d');

    return new Chart(ctx, {
        type: 'line',
        data: {
            labels: labels,
            datasets: datasets.map(ds => ({
                label: ds.name,
                data: ds.values,
                borderColor: ds.hex,
                backgroundColor: ds.hex + '20',
                borderWidth: 1.5,
                pointRadius: 2,
                pointHoverRadius: 5,
                tension: 0.3,
                fill: false,
            }))
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            interaction: {
                mode: 'index',
                intersect: false,
            },
            plugins: {
                legend: {
                    display: true,
                    position: 'bottom',
                    labels: {
                        usePointStyle: true,
                        pointStyle: 'circle',
                        padding: 12,
                        font: { size: 11 }
                    }
                },
                tooltip: {
                    backgroundColor: '#31302d',
                    titleFont: { size: 12 },
                    bodyFont: { size: 11 },
                    padding: 10,
                    cornerRadius: 6,
                }
            },
            scales: {
                x: {
                    grid: { display: false },
                    ticks: { font: { size: 10 } }
                },
                y: {
                    grid: { color: 'rgba(0,0,0,0.06)' },
                    ticks: {
                        font: { size: 10 },
                        callback: v => v + '%'
                    }
                }
            }
        }
    });
}
```

### Interaction Recommendations

- **Hover**: Use Chart.js built-in `interaction.mode: 'index'` — highlights all datasets at the same x position.
- **Legend click**: Default Chart.js behavior toggles dataset visibility. This is sufficient for most cases.
- **Drilldown**: If needed, use an explicit button or link below the chart — do NOT overload click/double-click on the chart itself.

---

## 3. SVG Coordinate Chart

For multi-dimensional data (e.g., hue × saturation × brightness clusters). Hand-drawn SVG with precise coordinate control.

### HTML Structure

```html
<div class="chart-svg-wrap" data-gender="women" data-season="ss">
    <svg viewBox="0 0 600 400" preserveAspectRatio="xMidYMid meet">
        <!-- Grid, axes, and data paths rendered by JS -->
    </svg>
</div>
```

### JS Pattern

```javascript
function renderCoordinateChart(container) {
    const ns = 'http://www.w3.org/2000/svg';
    const svg = container.querySelector('svg') || document.createElementNS(ns, 'svg');

    const W = 600, H = 400;
    const PAD = { t: 30, r: 20, b: 40, l: 50 };

    const mapX = (val) => PAD.l + (val / maxX) * (W - PAD.l - PAD.r);
    const mapY = (val) => H - PAD.b - (val / maxY) * (H - PAD.t - PAD.b);

    // Grid lines
    const gridG = document.createElementNS(ns, 'g');
    yTicks.forEach(tick => {
        const line = document.createElementNS(ns, 'line');
        line.setAttribute('x1', PAD.l);
        line.setAttribute('x2', W - PAD.r);
        line.setAttribute('y1', mapY(tick));
        line.setAttribute('y2', mapY(tick));
        line.setAttribute('stroke', 'rgba(93,78,63,0.12)');
        line.setAttribute('stroke-width', '0.5');
        gridG.appendChild(line);
    });
    svg.appendChild(gridG);

    // Data paths
    const linesG = document.createElementNS(ns, 'g');
    data.colors.forEach(color => {
        const path = document.createElementNS(ns, 'path');
        const d = color.points.map((p, i) =>
            `${i === 0 ? 'M' : 'L'}${mapX(p.x)},${mapY(p.y)}`
        ).join(' ');
        path.setAttribute('d', d);
        path.setAttribute('fill', 'none');
        path.setAttribute('stroke', color.hex);
        path.setAttribute('stroke-width', '0.7');
        path.setAttribute('opacity', '0.3');
        linesG.appendChild(path);
    });
    svg.appendChild(linesG);

    // Centroid (average path)
    const centroid = document.createElementNS(ns, 'path');
    centroid.setAttribute('d', centroidPathData);
    centroid.setAttribute('stroke', '#1a1613');
    centroid.setAttribute('stroke-width', '1.6');
    centroid.setAttribute('stroke-dasharray', '4,2.5');
    centroid.setAttribute('fill', 'none');
    svg.appendChild(centroid);
}
```

---

## 4. DOM Swatch Gallery

Large color palette display with filtering by gender/season/cohort. Rendered dynamically via JS for efficient filtering.

### HTML Structure

```html
<div class="card">
    <div class="gallery-topbar">
        <span class="panel-title">Color Gallery</span>
        <div class="filter-group">
            <div class="filter-select">
                <select id="genderFilter">
                    <option value="women">Women</option>
                    <option value="men">Men</option>
                </select>
            </div>
            <div class="filter-select">
                <select id="seasonFilter">
                    <option value="ss">Spring/Summer</option>
                    <option value="fw">Fall/Winter</option>
                </select>
            </div>
        </div>
    </div>
    <div class="gallery-container" id="galleryContainer">
        <!-- JS renders cohort columns here -->
    </div>
</div>
```

### CSS

```css
.gallery-container {
    display: flex;
    gap: var(--space-4);
    flex: 1;
    min-height: 0;
    overflow: hidden;
}

.cohort-column {
    flex: 1;
    display: flex;
    flex-direction: column;
    gap: var(--space-2);
    min-height: 0;
    overflow-y: auto;
}

.primary-group {
    margin-bottom: var(--space-2);
}

.pg-label {
    display: flex;
    align-items: center;
    gap: var(--space-2);
    font-size: var(--text-xs);
    font-weight: 600;
    margin-bottom: var(--space-1);
}

.pg-dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    flex-shrink: 0;
}

.swatch-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(18px, 1fr));
    gap: 2px;
}

.gal-swatch {
    aspect-ratio: 1;
    border-radius: 2px;
    border: 1px solid rgba(0, 0, 0, 0.08);
    cursor: pointer;
    transition: transform 0.2s ease;
}

.gal-swatch:hover {
    transform: scale(1.3);
    z-index: 5;
    box-shadow: 0 3px 10px rgba(0, 0, 0, 0.2);
}
```

### JS Pattern

```javascript
function renderGallery(data, gender, season) {
    const container = document.getElementById('galleryContainer');
    container.innerHTML = '';

    const seasonData = data[gender][season];
    if (!seasonData) return;

    seasonData.cohorts.forEach(cohort => {
        const col = document.createElement('div');
        col.className = 'cohort-column';

        // Cohort header
        const header = document.createElement('div');
        header.className = 'col-header';
        header.innerHTML = `<div class="col-title">${cohort.name}</div>
            <div class="col-subtitle">${cohort.n} colors</div>`;
        col.appendChild(header);

        // Group by primary color
        const groups = groupByPrimary(cohort.colors);
        groups.forEach(group => {
            const groupEl = document.createElement('div');
            groupEl.className = 'primary-group';

            const label = document.createElement('div');
            label.className = 'pg-label';
            label.innerHTML = `<span class="pg-dot" style="background:${group.hex}"></span>
                <span>${group.name}</span>`;
            groupEl.appendChild(label);

            const grid = document.createElement('div');
            grid.className = 'swatch-grid';
            group.colors.forEach(color => {
                const swatch = document.createElement('div');
                swatch.className = 'gal-swatch';
                swatch.style.background = color.hex;
                swatch.title = `${color.name} — ${color.pr}%`;
                grid.appendChild(swatch);
            });
            groupEl.appendChild(grid);
            col.appendChild(groupEl);
        });

        container.appendChild(col);
    });
}
```

---

## 5. CSS Data Table

Summary statistics in a clean table format. Pure CSS, no JS needed.

### HTML Structure

```html
<div class="card">
    <div class="data-table">
        <div class="table-head">
            <span class="table-head-label">Metric</span>
            <span class="col-header" style="color: var(--color-primary)">Women</span>
            <span class="col-header" style="color: var(--color-secondary)">Men</span>
        </div>
        <div class="data-row">
            <span class="row-label">Total Looks</span>
            <span class="big-num" style="color: var(--color-primary)">8,239</span>
            <span class="big-num" style="color: var(--color-secondary)">5,117</span>
        </div>
        <!-- more rows -->
    </div>
</div>
```

### CSS

```css
.data-table {
    border-radius: var(--board-radius);
    background: var(--color-surface);
    border: 1px solid var(--color-surface-border);
    box-shadow: 0 12px 36px var(--color-surface-shadow);
    overflow: hidden;
}

.table-head {
    display: grid;
    grid-template-columns: clamp(6rem, 14vw, 9rem) 1fr 1fr;
    padding: var(--space-4) var(--space-8);
    border-bottom: 1.5px solid var(--color-divider);
    align-items: end;
}

.data-row {
    display: grid;
    grid-template-columns: clamp(6rem, 14vw, 9rem) 1fr 1fr;
    align-items: center;
    padding: var(--space-6) var(--space-8);
    border-bottom: 1px solid rgba(102, 88, 75, 0.06);
}

.data-row:last-child { border-bottom: none; }

.big-num {
    font-family: var(--font-display);
    font-size: var(--text-3xl);
    font-weight: 700;
    line-height: 1;
    text-align: center;
}
```

---

## Color Utility: Light Color Detection

Many chart patterns need to switch text color for light-colored bars/swatches:

```javascript
function isLightColor(hex) {
    hex = hex.replace('#', '');
    const r = parseInt(hex.substring(0, 2), 16);
    const g = parseInt(hex.substring(2, 4), 16);
    const b = parseInt(hex.substring(4, 6), 16);
    const luminance = (0.299 * r + 0.587 * g + 0.114 * b) / 255;
    return luminance > 0.65;
}
```
