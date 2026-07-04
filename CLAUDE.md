# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page static site for beachris.ink (Bea, tattoo artist — Old School / pet portraits), in Portuguese. No framework, no build tool, no package manager, no test suite. The entire site is `index.html`: all CSS lives in the `<style>` block in `<head>`, all behavior lives in the one `<script>` block at the end of `<body>`. Assets are plain files under `assets/`.

## Running it locally

There is no build step. Serve the directory with any static file server and open it, e.g.:

```
npx serve .
```

Opening `index.html` directly via `file://` mostly works but can be flaky for the `@font-face` load — prefer a local server.

## Repo layout

- `index.html` — the entire site (markup + CSS + JS).
- `assets/fonts/OldSchoolUnited-Regular.ttf` — the only font file actually loaded by the site (via `@font-face`, referenced with a relative path, so it must stay at that path).
- `assets/img/` — photos used in the hero, about, and portfolio sections.
- `refs/`, `old_school_united.zip`, `old_school_united/` — design reference screenshots and the original font family package (all weights/styles). Nothing in `index.html` points at these; they're kept for future edits but excluded from git via `.gitignore`. Don't "clean up" `assets/fonts/OldSchoolUnited-Regular.ttf` thinking it's an orphaned duplicate of the zip's contents — it's the one actually served.

## Architecture notes that aren't obvious from a single scroll through the file

**The portfolio gallery is a scroll-hijacked horizontal track, not a grid.** In `#portfolio`, `.gallery-pin` is an oversized container; `.gallery-sticky` pins inside it (`position: sticky; top: 61px`, height `calc(100vh - 61px)` to sit right below the fixed nav) and holds three stacked pieces: `.portfolio__head` (title/copy), `.gallery-middle > .gallery-track` (the images), and a `.marquee` strip. As the page scrolls through `.gallery-pin`'s extra height, the JS at the bottom (`updateGalleryHeight` / `updateGalleryScroll`) translates `.gallery-track` horizontally in proportion to scroll progress — so vertical scrolling drives horizontal image movement while the section stays pinned. Once the track is fully scrolled, normal vertical scrolling resumes. Below 900px this whole mechanism is disabled by media query (`.gallery-pin { height: auto !important }`, sticky turned off, transform forced to `none`) in favor of plain touch-scrollable `overflow-x: auto` with `scroll-snap`. If you change `.gallery-sticky`'s height or the fixed nav's height, update `NAV_HEIGHT` in the script and the `top`/`height` values together — they're derived from the same 61px nav height and will desync silently otherwise.

**The marquee bands are rebuilt in JS, not just looped in CSS.** `.marquee__track`'s `@keyframes marquee-scroll` moves by a CSS custom property (`--marquee-distance`), not a hardcoded `-50%`. `setupMarquee()` in the script measures the real rendered width of one `.marquee__row`, clones it enough times to cover at least 2x the container width, and sets `--marquee-distance`/`--marquee-duration` so the loop is seamless and runs at a constant speed (`MARQUEE_SPEED`, px/second) regardless of viewport width, locale text length, or webfont metrics. It reruns on resize and once `document.fonts.ready` resolves. There are two independent `.marquee` instances on the page (the standalone strip after the tour banner, and the one nested inside `.gallery-sticky`); `document.querySelectorAll('.marquee').forEach(setupMarquee)` wires up both.

**`overflow-x: hidden` must stay on `body`, not on `.shell`.** Putting it on the inner `.shell` wrapper instead turns that div into an implicit scroll container in some browsers, which breaks `position: sticky` for any descendant (including the gallery above) — the sticky element ends up pinned relative to `.shell` instead of the viewport. This was a real bug once; don't move the property back onto `.shell`.

**The WhatsApp quote form doesn't submit to a server.** `#quote-form`'s submit handler in the script serializes the fields into a formatted message and opens a `wa.me` deep link. `WHATSAPP_NUMBER` (currently a placeholder, `5511900000000`) must be replaced with the real number before this goes live.

**Design tokens** are CSS custom properties in `:root` (`--ink`, `--paper`, `--brass`, `--muted`, `--dim`, etc.) — color edits should go through these rather than new hardcoded hex values, since the palette is reused across every section.

**Naming convention:** BEM-ish, scoped per section (`.hero__*`, `.about__*`, `.portfolio__*` / `.gallery-*`, `.cta__*`, `.footer__*`), with a few shared primitives reused across sections (`.btn`, `.btn--brass` / `.btn--outline`, `.chip`, `.eyebrow`). Follow this pattern for new sections rather than introducing inline styles.
