# AGENTS.md

Context and rules for working on this repository. Read fully before changing anything.

## Project

Personal portfolio for Edel/Grey/Ray (GitHub: Reydeuss), a computer science student. A single-page static site built with Hugo and hosted on GitHub Pages. Audience: recruiters, researchers, and selection reviewers who should understand the work in under a minute.

Core principle: **as light as possible.** The author cares about the lightweight-web movement (1MB Club style).

## Hard constraints

- **No JavaScript** *by default*. No frameworks, no bundlers, no Node toolchain. Ask before adding any script. Prefer HTML/CSS solutions (`prefers-color-scheme`, `<details>`, `scroll-behavior`, `loading="lazy"`).
- Do not add dependencies, plugins, or Hugo modules without confirming with the user first.

## Preferences
- Semantic HTML for content is preferable; inform the user if this is not possible or better approaches exist.

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

`partials/project.html` receives the page and renders one `<article>` with an `<h3>` title, an uppercase `<small>` stack label from `.Params.stack` (comma-separated in front matter, interpuncts in output; a plain meta line, not a chip), the content, and a source link from `.Params.repo`.

`baseof.html` has a `<nav>` linking to the section anchors, then `<main>` containing the block. The nav is a sibling of `<main>`, not inside `<header>` — it overlays the hero. Include `<meta name="viewport">`, `<meta name="color-scheme" content="light">`, a description meta, and optionally `<meta name="theme-color" content="#000000">`.

## Design system (Swiss style + brutalist structure)

`assets/css/main.css` is the source of truth. Keep to these rules:

- **Layout:** 12-column grid. Section titles in the left rail (columns 1-3), content in columns 4-12. Single column below 48rem; gutters tighten below 30rem. Backgrounds run full width; content stays within 80rem via the `--inset` variable. The hero exposes its 12 columns as faint, hard-edged 1px lines (`repeating-linear-gradient`, equal twelfths of the field). They are texture, not track guides: they do not align with the grid gutters, and that is deliberate.
- **Type:** one sans family: `"Helvetica Neue"`, Helvetica, Arial, `"Liberation Sans"`, then generic `sans-serif`. No system-ui. Hierarchy from size, weight, and tight tracking. Flush-left, ragged-right. Body about 17px, 1.5 line height, 60ch max measure. Large header name (up to 11rem).
- **Structure is weight, not decoration:** slab rules (`--rule: 3px solid currentColor`) above every section and between projects; the first article drops its top rule, and the footer relies on the colour change instead of a rule. No cards, box-shadows, border radius, icons, or decorative gradients. No press/translate hovers.
- **Interaction is flat inversion, nothing else:** hover and `:focus-visible` swap a solid block — `background: var(--inv-bg); color: var(--inv-fg); text-decoration: none`. Each field declares its own pair: `nav` and `header` are white-on-green, `section` is black-on-white, `#contact` is white-on-near-black. Inline links also take a 3px `currentColor` outline at 3px offset when focused; where the inverted block already provides the contrast (white block on the green hero), the block is the focus indicator. Do not add shadows, opacity fades, or movement.
- **Colour (black on white; one green field and one reversed field):**
  - Hero: full-width deep-green field (`--title-bg: #123524`) with white ink (`--title-fg: #fff`), nav overlaid in white. This is the only loud surface.
  - Reading column: white paper (`--bg: #fff`) with black ink (`--fg: #000`) for the About, Projects, and CV sections. Section titles and slab rules are black.
  - Footer: reversed field, white ink on near-black (`--footer-bg: #121212`), with no rule above it; the colour change marks the end of the page.
  - Links and borders use `currentColor` so they take their field's ink: black on the white reading column, white on the hero and footer.
  - Colours are fixed: `<meta name="color-scheme" content="light">`, and no dark-mode overrides exist.
- **Nav:** absolutely positioned over the top of the hero; the header reserves `--nav-h` in its top padding so content never collides. Links inherit the hero's white ink and invert to a padded block on hover/focus: `padding: 0.5rem 0.75rem`, `margin-inline: -0.75rem` so the labels stay flush with the h1, `gap: 2rem` so the enlarged blocks stay distinct. The nav is outside `<header>`, so it declares its own inversion pair. Keep the header unpositioned — a positioned header would paint its field over the overlaid nav.
- **CV call to action:** the link inside `#cv p` is a solid inverted slab at rest (3px border in `--fg`, `padding: 0.5em 1.1em`), dropping to a transparent background with black ink on hover/focus — a flat two-state toggle.
- **Stack label:** `article small` is a plain uppercase meta line (0.75rem, `letter-spacing: 0.08em`) with no background or border.
- **Rejected:** shadows and offset/press hovers, bordered chips and inverted stack labels, cards, per-category colour coding of projects, numerals on projects (they are not a sequence), system-ui fonts, dark mode.
- To retheme, change the custom properties in `:root` only: `--bg`, `--fg`, `--title-bg`, `--title-fg`, `--footer-bg`, `--footer-fg`, `--sans`, `--nav-h`, `--pad`, `--gap`, `--rule`, `--inset`. The per-field `--inv-bg` / `--inv-fg` pairs are declared on `nav`, `header`, `section`, and `#contact` and reference those tokens; never hard-code colours there.

## Commands

```
hugo                   # development, for previewing build result
hugo --gc --minify     # production build, same as CI
```

Verify before finishing a change: build passes with no warnings, `public/` contains only the expected files, `public/index.html` contains no `<script>` tags, no external URLs appear other than the author's own links, and the page is under the size budget. A running `hugo server` can inject `/livereload.js` into `public/` — after previewing, rebuild with `hugo --gc --minify` so the shipped page stays script-free.

## Working style

- Keep changes small and explain what changed and why.
- Match existing conventions rather than introducing new ones.
- When a request conflicts with a hard constraint above, say so and ask before proceeding.
