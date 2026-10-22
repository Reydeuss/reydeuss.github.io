# AGENTS.md

Context and rules for working on this repository. Read fully before changing anything.

## Project

Personal portfolio for Edel/Grey/Ray (GitHub: Reydeuss), a computer science student. A single-page static site built with Hugo and hosted on GitHub Pages. Audience: recruiters, researchers, and selection reviewers who should understand the work in under a minute.

Core principle: **as light as possible.** The author cares about the lightweight-web movement (1MB Club style).

## Hard constraints

- **No JavaScript** *by default*. No frameworks, no bundlers, no Node toolchain. Ask before adding any script. Prefer HTML/CSS solutions (`prefers-color-scheme`, `<details>`, `scroll-behavior`, `loading="lazy"`).
- **Accessibility:** semantic HTML, one `h1`, `lang` attribute, AA contrast in light and dark, visible focus states, content readable with CSS disabled.
- Do not add dependencies, plugins, or Hugo modules without confirming with the user first.

## Stack and decisions already made

- Generator: **Hugo** (extended not required). Pin the exact version in CI.
- Hosting: GitHub Pages, deployed by a GitHub Actions workflow (Settings > Pages > Source: GitHub Actions). Workflow installs the pinned Hugo binary, runs `hugo --gc --minify`, uploads `public/`.
- **One page only, for now.** Sections are anchors: `#about`, `#projects`, `#cv`, `#contact`. No per-project pages, no list pages, no taxonomies.

## Repository layout

```
hugo.toml
content/
  _index.md                  # intro text, rendered in the header
  about.md                   # About section text, rendered by home.html
  projects/
    _index.md                # build settings that suppress per-project pages
    <project>.md             # one file per project
layouts/
  baseof.html                # <head>, nav, main block
  home.html                  # the only content template
  partials/project.html      # one project article: h3, uppercase stack label, content, source link
assets/css/main.css          # processed, minified, inlined into <head>
.github/workflows/hugo.yml
```

If project pages stop being available to the template, `"section"` must not be in `disableKinds`.

`content/projects/_index.md` must keep project files out of the output but available to templates:

```toml
+++
title = "Projects"
[build]
  render = "never"
  list = "local"
[cascade.build]
  render = "never"
  list = "local"
+++
```

CSS is inlined in `<head>`:

```go-html-template
{{ with resources.Get "css/main.css" | minify }}<style>{{ .Content | safeCSS }}</style>{{ end }}
```

After a build, `public/` should contain `index.html`, `404.html`, `sitemap.xml`, and little else. Verify this.

## Project content

Each file in `content/projects/` uses this front matter:

```toml
+++
title = ""
repo = ""
stack = ""
weight = 1
+++
Two or three sentences. Problem, approach, result.
```

Repo URL pattern: `https://github.com/Reydeuss/<name>`.

**Do not invent results, metrics, or claims.** Blurbs use only what the repo descriptions and READMEs state. Lines marked `TODO` in content files are placeholders for the author to fill in (a result, dataset size, metric, role, status). They must be replaced or removed before publishing; never ship a literal `TODO`.

## Templates

`home.html` renders, in order: a `<header>` with `<h1>` (site title) and `.Content` of `_index.md`; `<section id="about">` reading `content/about.md` via `site.GetPage`, rendering its title in the left rail and its content on the right; `<section id="projects">` looping `site.GetPage "/projects"` pages by weight and calling `partials/project.html` for each; a `<section id="cv">` linking to the CV; a `<footer id="contact">` with email and GitHub links.

`partials/project.html` receives the page and renders one `<article>` with an `<h3>` title, an uppercase `<small>` stack label from `.Params.stack` (comma-separated in front matter, interpuncts in output), the content, and a source link from `.Params.repo`.

`baseof.html` has a `<nav>` linking to the section anchors and `<main>` containing the block. Include `<meta name="viewport">`, `<meta name="color-scheme" content="light">`, a description meta, and optionally `<meta name="theme-color" content="#000000">`.

## Design system (Swiss style)

`assets/css/main.css` is the source of truth. Keep to these rules:

- **Layout:** 12-column grid. Section titles in the left rail (columns 1-3), content in columns 4-12. Single column below 48rem. Backgrounds run full width; content stays within 80rem via the `--inset` variable.
- **Type:** one sans family (Helvetica and its metric substitutes; no system-ui fallback). Hierarchy from size, weight, and tight tracking. Flush-left, ragged-right. Body about 17px, 1.5 line height, 60ch max measure. Large header name (up to 11rem).
- **No** cards, shadows, border radius, icons, gradients, or decorative elements. Rules (borders) and whitespace do the structuring.
- **Colour (black on white; one red field and one reversed field):**
  - Hero: full-width Swiss-red field (`#e30613`) with white ink, nav overlaid in white. This is the only red surface.
  - Reading column: white paper (`#fff`) with black ink for the About, Projects, and CV sections. Section titles and hairlines are black.
  - Footer: reversed field, white ink on near-black (`#121212`), with no rule above it; the colour change marks the end of the page.
  - Links/borders use `currentColor` so they take their field's ink: black on the white reading column, white on the hero and footer.
  - All outlines are hairlines: `1px solid currentColor` above sections and between projects. No heavy bars.
  - Colours are fixed; the page declares `<meta name="color-scheme" content="light">` and no dark-mode overrides exist.
- **Rejected:** per-category colour coding of projects. Do not colour projects by type. Do not add numerals to projects (they are not a sequence).
- Colours that appear as text must meet AA contrast. Yellow is not usable as text on light backgrounds.
- To retheme, change the custom properties in `:root` only.

## Commands

```
hugo server            # local dev
hugo --gc --minify     # production build, same as CI
```

Verify before finishing a change: build passes with no warnings, `public/` contains only the expected files, no external URLs appear in `public/index.html` other than the author's own links, and the page is under the size budget.

## Working style

- Keep changes small and explain what changed and why.
- Match existing conventions rather than introducing new ones.
- When a request conflicts with a hard constraint above, say so and ask before proceeding.
