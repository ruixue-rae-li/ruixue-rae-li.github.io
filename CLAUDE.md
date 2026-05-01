# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Site Overview

Academic portfolio for Ruixue (Rae) Li, PhD Candidate at Columbia University. Built on the **Academic Pages** Jekyll theme, hosted at `ruixue-rae-li.github.io`. It is a fully static site — no backend, no database.

## Development Commands

### Local Development (Docker — preferred)
```bash
docker compose up          # Serves on http://localhost:4000
```

### Local Development (direct Ruby/Node install)
```bash
bundle install
npm install
npm run build:js           # Bundle JS into assets/js/main.min.js
jekyll serve -l -H localhost   # -l enables live reload
```

### JavaScript rebuild (after editing assets/js/_main.js)
```bash
npm run build:js           # or: npm run watch:js for continuous watching
```

Config changes (`_config.yml`) require restarting Jekyll — live reload won't pick them up.

## Architecture

### Content Model

Content lives in Jekyll **collections**. Each item is a Markdown file with YAML frontmatter:

| Directory | URL prefix | Layout used |
|---|---|---|
| `_publications/` | /publications/ | single |
| `_talks/` | /talks/ | talk |
| `_teaching/` | /teaching/ | single |
| `_portfolio/` | /portfolio/ | single |

The **homepage** is `_pages/about.md`. It uses anchor IDs (`#research`, `#teaching`, `#contact`) that the navigation links in `_data/navigation.yml` point to.

### Layout Hierarchy

```
_layouts/compress.html         ← HTML minifier (outermost wrapper)
└── _layouts/default.html      ← master layout (head, masthead, footer)
    ├── _layouts/single.html   ← individual pages/publications
    ├── _layouts/talk.html     ← talks with venue metadata
    ├── _layouts/archive.html  ← collection listing pages
    └── _layouts/cv-layout.html
```

Includes in `_includes/` are partials injected by layouts. Key ones:
- `author-profile.html` — sidebar with avatar + social links
- `masthead.html` — top navbar with theme toggle
- `head.html` — `<head>` meta, CSS, analytics

### CSS/SCSS

All SCSS lives under `_sass/`. Entry point: `assets/css/main.scss`. The build pipeline is:

```
_sass/main.scss
├── _sass/include/_mixins.scss, _utilities.scss
├── _sass/layout/*.scss        ← page sections (_masthead, _sidebar, _page, etc.)
├── _sass/theme/default_light.scss + default_dark.scss  ← active theme colors
└── _sass/vendor/              ← Breakpoint, Susy, Font Awesome
```

To change the active theme, set `site_theme` in `_config.yml` (options: `default`, `air`, `contrast`, `dirt`, `mint`, `sunrise`). Each theme has a `_light.scss` and `_dark.scss` pair under `_sass/theme/`.

### JavaScript

`assets/js/_main.js` is the source. It handles:
- Dark/light theme toggle (persisted in `localStorage`)
- Plotly chart rendering (reads `language-plotly` fenced code blocks)
- Smooth scroll with `-70px` offset to clear the 70px fixed masthead

After editing `_main.js`, run `npm run build:js` — the site loads `assets/js/main.min.js` (the bundled output), not the source file.

### Configuration

All site-level settings are in `_config.yml`. Commonly edited fields:

```yaml
author:
  name, avatar, bio, email, github, ...  # sidebar profile
site_theme: "default"                    # visual theme
publication_category:                    # custom publication groupings
  books / manuscripts / conferences
```

Navigation links: `_data/navigation.yml`

Social link labels/icons: `_data/ui-text.yml`

### CV

The CV PDF is at `files/ruixue_rae_li_cv.pdf`, sourced from `files/ruixue_rae_li_cv.tex`. The nav links directly to the PDF. To update the CV, recompile the `.tex` file and replace the PDF.

### Talkmap

`talkmap/talkmap.py` (and the `.ipynb` notebook) geocodes talk locations from `_talks/*.md` and writes `talkmap/org-locations.js`. A GitHub Action (`.github/workflows/scrape_talks.yml`) runs this automatically on pushes that touch `_talks/`.

### Markdown Generators

`markdown_generator/` contains Python scripts and Jupyter notebooks that convert TSV or BibTeX input into correctly formatted Markdown files for `_publications/` and `_talks/`. Use these rather than hand-writing entries when importing many items at once.
