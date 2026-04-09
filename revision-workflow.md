# Revision Workflow

How to handle stakeholder feedback efficiently, avoiding the "5 versions in one day" anti-pattern.

## Core Principle

Not all feedback is the same. Classify it first, then execute appropriately.

## Step 1: Receive & Document Feedback

When stakeholder feedback arrives, capture it in a structured file:

```markdown
<!-- docs/revisions/round-1.md -->
# Revision Round 1
Date: YYYY-MM-DD
From: [Stakeholder name]

## Feedback Items

1. Change "Color" to "Colorway" throughout all pages
2. Add conclusion text below 1.2 page title: "Over the past year..."
3. Modify 2.2 cohort naming convention
4. Gallery layout feels too cramped, consider more spacing
5. Cover title needs updating to "SP26 Colorway Trend Study"
```

## Step 2: Classify Each Item

| Category | Definition | Example | Executor |
|----------|------------|---------|----------|
| **Deterministic** | Exact find-and-replace, zero ambiguity | "Color → Colorway" | Agent auto |
| **Content injection** | Human provides exact text, agent places it | "Add this conclusion paragraph below title" | Human writes → Agent injects |
| **Parameterized** | Change a token or variable | "Card corners rounder" | Agent changes token |
| **Judgment required** | Subjective, needs human decision | "Layout feels cramped" | Human decides → Agent implements |
| **Directional** | Major redesign or restructure | "Redo the gallery as a grid instead of columns" | Scope assessment first |

Annotate the feedback document:

```markdown
1. [DETERMINISTIC] Change "Color" to "Colorway" throughout
   Scope: display text only, skip JS variable names and CSS class names
   
2. [CONTENT] Add conclusion text below 1.2 page title
   Text: "Over the past year for women, Earth Tone kept dominant..."
   Position: between .title and .card in slide-1-2
   
3. [DETERMINISTIC] Modify 2.2 cohort naming
   Rule: remove words after dot separator
   
4. [JUDGMENT] Gallery spacing feels cramped
   → Need to see current state, propose 2 options
   
5. [DETERMINISTIC] Update cover title
   Find: "SP26 Color Trend Study"
   Replace: "SP26 Colorway Trend Study"
```

## Step 3: Execute by Category

### Deterministic Changes

Execute all at once. These are safe to batch:

```bash
# In each relevant source file (not the merged 4000-line file!)
# Agent can search-replace with confidence
```

Scope rules for text replacement:
- ✅ Replace in: page titles, eyebrows, takeaway text, footer notes, cover text
- ❌ Do NOT replace in: JS variable names, CSS class names, JSON field names, code comments
- ⚠️ Check: navigation tab labels (may or may not need changing — ask if unclear)

### Content Injection

Human provides the exact text. Agent injects at the specified position:

1. Human writes content in the revision document
2. Agent opens the relevant page file (e.g., `src/pages/1.2-primary.html`)
3. Agent adds the text element at the specified position
4. Agent ensures correct CSS class (usually `.takeaway`)

### Parameterized Changes

These map directly to design tokens:

| Feedback | Token to change |
|----------|----------------|
| "Rounder card corners" | `--board-radius` in tokens.css |
| "Bigger page titles" | `--text-2xl` in tokens.css |
| "More space between sections" | `--space-8` or `--space-12` in tokens.css |
| "Different accent color" | `--color-accent` in tokens.css |

One-line change, all pages update automatically.

### Judgment-Required Changes

1. Agent proposes 2-3 options (e.g., increased gap values)
2. Present options to human (if possible, generate quick previews)
3. Human selects
4. Agent implements the selected option

### Directional Changes

These need scope assessment before implementation:

1. **Impact analysis**: How many files are affected? Does it change the page structure?
2. **Effort estimate**: Quick (< 30 min) or significant (> 2 hours)?
3. **Risk assessment**: Could this break existing pages?
4. **Decision**: Proceed, defer to next version, or reject with explanation

If proceeding:
- Create a feature branch: `git checkout -b revision/gallery-redesign`
- Implement on branch
- Review merged result
- Merge to main if approved

## Step 4: Verify & Tag

After all revisions are applied:

1. Run `build.py` → generate updated merged HTML
2. Spot-check every changed page in the browser
3. For terminology changes: search the merged file to confirm no stale instances remain
4. Commit with descriptive message:
   ```
   content(revision-1): apply stakeholder feedback round 1
   
   - Terminology: Color → Colorway (display text only)
   - Added conclusion text to pages 1.1, 1.2, 1.3
   - Updated cover title and department line
   ```
5. Tag: `git tag v0.9-revision-1`

## Anti-Patterns to Avoid

| Anti-pattern | Why it's bad | What to do instead |
|-------------|-------------|-------------------|
| Editing the merged 4000-line file directly | AI loses context, changes wrong section | Always edit in `src/pages/` source files |
| Applying feedback item by item across multiple sessions | Each session re-reads the entire file | Batch all deterministic changes into one session |
| Mixing revision with design exploration | Scope creeps, creates new feedback loops | Revision is execution only — no new design decisions |
| Not classifying feedback first | Wastes time on items that need human input | Always classify → then execute in order of confidence |
| Creating file-v2.html, file-v3.html | Loses git history, confuses future sessions | Use git commits + tags |

## Template: Revision Execution Checklist

```markdown
## Revision Round [N] Execution

### Pre-flight
- [ ] Feedback document annotated with categories
- [ ] Source files up to date (git status clean)
- [ ] Current version tagged

### Deterministic
- [ ] Item 1: [description] — applied in [file]
- [ ] Item 5: [description] — applied in [file]

### Content
- [ ] Item 2: [description] — text provided, injected in [file]

### Parameterized
- [ ] Item X: [token change description]

### Judgment
- [ ] Item 4: [options presented, human selected option B]

### Post-flight
- [ ] build.py executed successfully
- [ ] All changed pages spot-checked in browser
- [ ] No stale terminology in merged file
- [ ] Git committed with descriptive message
- [ ] Tagged: vX.Y-revision-N
```
