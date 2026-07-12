# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single-file personal portfolio (`index.html`) for **허유경 (Heo Yugyeong)**, an AI · full-stack developer. The aesthetic is a clean, light **GENTLE MONSTER–style editorial** (white-tone, monochrome) with a fullscreen video hero and scroll-driven motion. It is built as a portfolio-grade artifact for an IICOMBINED application.

There is no build system, no package manager, and no test runner. The entire site lives in one HTML file that loads its dependencies from CDNs. The repo is a git repo intended for **GitHub Pages** deployment (push to `main`, enable Pages on root).

> History: an earlier version (`portfolio.html`, now deleted) was a dark/neon Three.js concept piece. It was replaced by this white-tone editorial redesign. See `docs/superpowers/specs/2026-05-31-portfolio-redesign-design.md` and `docs/superpowers/plans/2026-05-31-portfolio-redesign.md`.

## Running / previewing

Open `index.html` directly, or serve it (needed so the `<video>` and CDN assets load cleanly):

```bash
# from the repo root
python3 -m http.server 8000
# then open http://localhost:8000/
```

The hero video (`Soft_pink_flowers_and_grass_sw.mp4`) and `assets/` live at the repo root.

Quick visual check after edits: reload and scroll top→bottom. Expect: loader fades → fullscreen pink-flower video hero → INTRO → WORK (7 cinematic alternating image rows; click a row → fullscreen detail panel with animated metric count-up) → ABOUT → keyword marquee → CONTACT. Watch the console for errors.

## Architecture (one file, three layers)

`index.html` is organized top-to-bottom as:

1. **Head + inline `<style>`** — Tailwind Play CDN config (palette `paper #F5F3EF` / `snow #FFFFFF` / `ink #0A0A0A` / `ash #6B6B6B` / `line #E2DED7`; fonts `Urbanist` display + `Pretendard` Korean), then custom CSS for `.reveal`, `.label`, the nav transparent→solid (`#nav.solid`) transition, and the marquee keyframes.
2. **Body markup** — loader, scroll-progress bar, fixed `#nav`, five sections in `<main>` (`#hero`, `#intro`, `#work`, `#about`, marquee, `#contact`), and the fullscreen `#panel` (WORK detail overlay).
3. **Inline `<script>` IIFE** — one `(() => { 'use strict'; ... })()` block. Order matters:
   - `PROJECTS` data array (the 7 projects — single source of truth) and `SKILLS` array.
   - Renders the WORK rows (`#work-list`) and ABOUT skill chips (`#skills`) from those arrays. Each project's image is resolved by `imgPath(no)` via the `IMG` map (project `no` → `assets/*.jpg` filename); `THUMBS` gradients are the fallback, and a broken `<img>` removes itself via `onerror`. `data-pll` elements get a parallax tween.
   - **Lenis** inertia scroll wired into **GSAP ScrollTrigger** (`lenis.raf` driven by `gsap.ticker`).
   - `.reveal` scroll reveals, nav solid toggle, progress bar, smooth anchor scroll (`lenis.scrollTo`).
   - Hero scroll motion (video scale + copy fade via scrub ScrollTrigger).
   - WORK detail panel: `renderPanel`/`openPanel`/`closePanel`, metric **count-up** (GSAP `innerText` tween), row-click + Esc handlers, `lenis.stop()/start()` to lock scroll while open.
   - Loader intro timeline (ends with `ScrollTrigger.refresh()`).

Because everything is in one file, "where does X happen" is almost always answerable by searching `index.html` for the relevant id or symbol. The 7 projects' content lives **only** in `PROJECTS` — edit there, not in markup.

## Design system (do not drift)

The aesthetic is load-bearing. When adding or editing visuals, keep:

- **Color**: warm-white base (`paper #F5F3EF`), white (`snow`) for alternating sections, ink black (`#0A0A0A`) text, `ash` grey for secondary, `line` for hairlines. **Monochrome — no accent color.** Detail comes from whitespace, thin hairlines, and small uppercase tracked `.label`s.
- **Type**: `Urbanist` (display, light/wide for editorial headlines, `font-700` for emphasis) for English; `Pretendard` (`font-kr`) for Korean. Korean/English are intentionally mixed, GM-style.
- **Motion**: Lenis inertia scroll + GSAP/ScrollTrigger. Reveal-on-scroll (fade + rise, staggered via `data-delay`), scrubbed hero, metric count-up, sticky-nav transition, keyword marquee, load intro. No abrupt cuts.
- **Media**: fullscreen muted-loop video hero. Real images now live in `assets/` — `profile.jpg` plus per-project shots mapped in the `IMG` object (`aikey`, `cardnews`, `nexus`, `hover`, `football`, `fashion`, `sedaily`). Add a project image by dropping `assets/<name>.jpg` and pointing its `no` at it in `IMG`; missing images degrade to the `THUMBS` gradient automatically.

## Working on this repo

- **Single-file / CDN-only is a constraint.** No bundler, npm, or local deps. Need a library? Add a CDN `<script>` in the head.
- **Ambition is the default for visual work here** — see `~/.claude/projects/-Users-yugang-XENONIX-project-IICOMBINED/memory/feedback_creative_polish.md`. For visual changes, add complementary scroll/hover micro-interactions that reinforce the editorial aesthetic rather than shipping the minimum. Does **not** apply to config/refactor work — keep normal brevity there.
- **Prefer `Edit` against `index.html`**, not rewrites. Section banners (`<!-- ① HERO -->` … in markup, `/* ---------- ... ---------- */` in JS) make regions easy to locate.
- No tests. Verify by loading the page, scrolling through all sections, opening/closing each WORK panel, and checking the console (Lenis/ScrollTrigger init and the panel count-up are the likeliest error sources).
- Real assets (profile photo, project images) live in `assets/`; the user provides them. `.gitignore` excludes `node_modules/`, `.DS_Store`, and `*.HEIC` (the source `IMG_3486.HEIC` is not committed).
