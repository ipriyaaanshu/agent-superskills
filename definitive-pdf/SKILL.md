---
name: definitive-pdf
description: Production pipeline for executive-grade PDF proposals and reports — plain white, print-perfect output using WeasyPrint.
version: 1.0.0
author: Hermes Agent
---

# Definitive PDF — Proposal & Report Pipeline

Use this for any PDF that needs to look professional, print cleanly, and avoid formatting disasters. **Default: plain white background.** Dark themes are supported — see the Dark Theme section below.

## Stack

| Component | Tool |
|-----------|------|
| Authoring | HTML + CSS (hand-written) |
| Conversion | WeasyPrint (not Chrome/Playwright) |
| Venv | `/tmp/pdfenv` — activate with `source /tmp/pdfenv/bin/activate` |

If WeasyPrint is not installed: `python3 -m venv /tmp/pdfenv && source /tmp/pdfenv/bin/activate && pip install weasyprint`

## Prerequisites

### System packages (install once)

| Package | Provides | Why |
|---------|----------|-----|
| `python3-venv` or `python3-virtualenv` | `venv` module | Create `/tmp/pdfenv` |
| `poppler-utils` | `pdfinfo`, `pdftotext` | Verification step (# pages, searchable text, file size) |

**Debian / Ubuntu:**
```bash
sudo apt-get update && sudo apt-get install -y python3-venv poppler-utils
```

**macOS:**
```bash
brew install poppler
```

**Arch:**
```bash
sudo pacman -S python-virtualenv poppler
```

### WeasyPrint setup (one-liner)

```bash
python3 -m venv /tmp/pdfenv
source /tmp/pdfenv/bin/activate
pip install --upgrade pip
pip install weasyprint
```

> WeasyPrint also needs GTK (Pango/Cairo). On most Linux desktops these are already present. On headless servers:
> ```bash
> # Debian/Ubuntu headless
> sudo apt-get install -y libpango-1.0-0 libcairo2 libgdk-pixbuf2.0-0
> ```

### Verification that everything works

```bash
source /tmp/pdfenv/bin/activate
python -c "import weasyprint; print(weasyprint.__version__)"
which pdfinfo
which pdftotext
```

All three commands must succeed before generating PDFs.

## CSS Baseline (copy this)

```css
*, *::before, *::after { box-sizing: border-box; }
html, body {
  margin: 0; padding: 0;
  font-family: system-ui, -apple-system, 'Segoe UI', sans-serif;
  font-size: 10.5pt; line-height: 1.6;
  color: #1e293b;
}

@page {
  size: A4;
  margin: 2.2cm 1.8cm;
  @top-center { content: string(doc_title); font-size: 8.5pt; color: #94a3b8; }
  @bottom-center { content: counter(page); font-size: 8.5pt; color: #94a3b8; }
}
@page :first { margin: 2.8cm 2.4cm; @top-center { content: none; } @bottom-center { content: none; } }
@page contents { @top-center { content: none; } @bottom-center { content: none; } }

body { string-set: doc_title ""; }
h1 { string-set: doc_title content(); }

/* Cover page — PLAIN WHITE */
.cover-page { page-break-after: always; padding: 4cm 2cm 3cm 2cm; text-align: center; background: #fff; }
.cover-badge { display: inline-block; border: 1px solid #7c3aed; color: #7c3aed; padding: 4px 16px; border-radius: 99px; font-size: 10pt; letter-spacing: 0.06em; margin-bottom: 1.5rem; text-transform: uppercase; font-weight: 600; }
.cover-title { color: #0f172a; font-size: 26pt; font-weight: 800; line-height: 1.15; margin: 0 0 0.8rem 0; }
.cover-subtitle { color: #64748b; font-size: 12pt; font-weight: 400; margin: 0 0 2.5rem 0; line-height: 1.45; max-width: 75%; margin-left: auto; margin-right: auto; }
.cover-meta { color: #94a3b8; font-size: 9pt; margin-top: 2rem; }
.cover-line { width: 40px; height: 3px; background: #7c3aed; margin: 0 auto 1.5rem auto; }

/* TOC page — PLAIN WHITE */
.toc-sheet { page: contents; page-break-after: always; }
.toc-sheet h2 { font-size: 18pt; color: #1e293b; margin: 0 0 1.2rem 0; }
.toc-list { list-style: none; margin: 0; padding: 0; }
.toc-list li { margin: 0.5em 0; font-size: 10.5pt; }
.toc-list a { display: flex; gap: 0.4em; color: #1e293b; text-decoration: none; }
.toc-list a::after { margin-left: auto; content: target-counter(attr(href url), page); color: #94a3b8; }

/* Content */
h1 { font-size: 18pt; color: #0f172a; margin: 0 0 0.7rem 0; font-weight: 700; padding-bottom: 0.35rem; border-bottom: 2px solid #e2e8f0; page-break-before: always; }
h1:first-of-type { page-break-before: avoid; }
h2 { font-size: 13pt; color: #1e293b; margin: 1.3rem 0 0.5rem 0; font-weight: 600; }
h3 { font-size: 11.5pt; color: #334155; margin: 1rem 0 0.3rem 0; font-weight: 600; }
p { margin: 0 0 0.6rem 0; orphans: 3; widows: 3; }

pre { overflow: auto; white-space: pre-wrap; background: #f8fafc; border: 1px solid #e2e8f0; border-radius: 5px; padding: 0.7rem 0.9rem; font-size: 8.5pt; line-height: 1.45; margin: 0.8rem 0; }
code { font-family: 'SF Mono', 'Cascadia Code', monospace; font-size: 9pt; background: #f1f5f9; padding: 1px 4px; border-radius: 3px; }
pre code { background: none; padding: 0; }

figure { max-width: 100%; margin: 1rem 0; page-break-inside: avoid; }
figure img, figure svg { max-width: 100%; max-height: 35vh; height: auto; display: block; margin: 0 auto; }
figcaption { text-align: center; font-size: 9pt; color: #64748b; margin-top: 0.35rem; }
figcaption::before { content: attr(data-caption) " "; font-weight: 700; }

table { width: 100%; border-collapse: collapse; margin: 0.8rem 0; font-size: 9.5pt; page-break-inside: avoid; }
th { background: #f8fafc; color: #1e293b; font-weight: 600; text-align: left; padding: 7px 10px; border-bottom: 2px solid #cbd5e1; font-size: 9pt; text-transform: uppercase; letter-spacing: 0.03em; }
td { padding: 7px 10px; border-bottom: 1px solid #e2e8f0; color: #334155; }
thead { display: table-header-group; }

blockquote { border-left: 3px solid #7c3aed; padding-left: 1rem; margin: 0.8rem 0; color: #475569; font-style: italic; page-break-inside: avoid; }

.highlight-box { border-left: 3px solid #0891b2; padding: 0.6rem 1rem; margin: 0.8rem 0; page-break-inside: avoid; }
.highlight-box strong { color: #0e7490; }
.exec-summary { border-left: 3px solid #059669; padding: 0.8rem 1rem; margin: 1rem 0; page-break-inside: avoid; }
.exec-summary strong { color: #065f46; }

.roadmap-phase { margin: 1rem 0; page-break-inside: avoid; }
.roadmap-badge { display: inline-block; background: #7c3aed; color: #fff; padding: 2px 10px; border-radius: 99px; font-size: 8.5pt; font-weight: 600; white-space: nowrap; }
.roadmap-desc { padding-left: 1.2rem; border-left: 2px solid #e2e8f0; margin-left: 0.2rem; }

h2, h3 { page-break-after: avoid; }
```

## Hard Rules

1. **White background everywhere.** No dark pages. No gradients. No colored `background` on any block-level element. Use `border-left` for visual differentiation.
2. **`page-break-after: always`** — not `break-after: page`. WeasyPrint handles the old property correctly.
3. **CSS logical properties banned.** No `inline-size`, `block-size`, `margin-inline`, `padding-inline-start`. Use `width`, `height`, `margin`, `padding`.
4. **No viewport units.** `vh`/`vw` don't work in paged media. Use `mm`, `pt`, or `%`.
5. **`page-break-inside: avoid`** on tables, figures, blockquotes, callout boxes. Never on large parent containers.
6. **`thead { display: table-header-group }`** — headers repeat on multi-page tables.

## Dark Theme (When Not White)

Dark backgrounds render perfectly in WeasyPrint. **Do NOT use `@page { background: ... }`** — that property is poorly supported and can produce unexpected color shifts. Instead, remove page margins entirely and use a centered `.page` container.

### Required for dark backgrounds

```css
@page {
  size: letter portrait;   /* or A4 */
  margin: 0;               /* remove white margin boxes */
}

html, body {
  background-color: #0E0D0B;   /* obsidian — set explicitly */
  color: #E8DDC4;
}

.page {
  position: relative;
  max-width: 520pt;        /* ~70% of letter width */
  margin: 0 auto;          /* center the content column */
  padding: 28pt 24pt 48pt; /* internal spacing */
}
```

### Why `margin: 0` is required

WeasyPrint's default `@page` margins render as **white boxes** around the content. Setting `background` on `body` only fills the content area, not the margin area. `@page { margin: 0 }` eliminates those boxes so the dark body background bleeds edge-to-edge.

### Pitfall: `margin-bottom` on `page-break-inside: avoid` blocks

If two consecutive elements both have `page-break-inside: avoid`, a `margin-bottom` on the first can force the second onto its own page — creating an orphan.

**Bad:**
```css
.cta-block {
  page-break-inside: avoid;
  margin-bottom: 24pt;   /* orphan risk */
}
.footer {
  page-break-inside: avoid;
}
```

**Good:**
```css
.cta-block {
  page-break-inside: avoid;
  /* no margin-bottom; let footer flow inline */
}
.footer {
  /* page-break-inside: avoid removed; allow natural flow */
}
```

### Break-avoid hierarchy for dark documents

| Element | `page-break-inside` | `page-break-before` |
|---------|--------------------|---------------------|
| Stat cards (grid of 3) | `avoid` on `.stat-card` | — |
| Path cards (grid of 3) | `avoid` on `.path-card` + `.path-card-header` + `.path-body` | `page-break` on parent `.section-block` |
| Case studies (grid rows) | `avoid` on `.case-study` + `.case-study-metric` + `.case-study-body` | — |
| Engagement table | `avoid` on `table`, `tr`, and `td` content | — |
| Narrative/pull-quote | `avoid` on container | — |
| CTA block | `avoid` on container | — |
| Footer | **remove** `avoid` — let it flow with CTA | — |

## Converting an Existing HTML Design

When converting a browser-designed HTML to PDF:

1. **Inspect in Chrome first** — Use DevTools to verify the actual layout: `max-width`, margins, background behavior, card spacing.
2. **Map px → pt** — Browser designs use `px`; PDF uses `pt`. Divide by 1.33 (e.g., `18px` ≈ `13.5pt`).
3. **Remove interaction CSS** — Strip `transition`, `hover`, `cursor`, `animation` — they have no effect in PDFs and bloat the stylesheet.
4. **Test with WeasyPrint early** — Don't perfect the HTML in Chrome; WeasyPrint's layout engine differs. Iterate with `weasyprint input.html output.pdf` and `pdftoppm` for visual checks.
5. **Verify no orphans** — Check page count (`pdfinfo`) and visually inspect each page for stranded headers/footers.

## Conversion

```bash
source /tmp/pdfenv/bin/activate
weasyprint /path/to/document.html /path/to/output.pdf
```

## Verification

```bash
pdfinfo output.pdf           # Check pages, page size (must be A4)
pdftotext output.pdf - | head -20  # Verify searchable text, TOC page numbers
ls -lh output.pdf            # File size
```

## Page Count Tuning

| Too many pages | Too few pages |
|---|---|
| Reduce heading sizes | Increase line-height slightly |
| Reduce line-height | Increase margins |
| Reduce pre/code font sizes | — |
