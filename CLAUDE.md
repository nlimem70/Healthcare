# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

This repo is a single self-contained file, [index.html](index.html) — a one-page marketing/appointment-request site for a fictional clinic ("Harborview Family Clinic"). There is no build step, no package manager, and no dependencies. HTML, CSS (`<style>` in `<head>`), and JavaScript (`<script>` at the end of `<body>`) all live inline in that one file.

## Commands

There is no build, lint, or test tooling in this repo. To view changes, open `index.html` directly in a browser:

```powershell
Start-Process "index.html"
```

There is no automated test suite — verify changes manually in a browser (resize the viewport for mobile layout, exercise the form, tab through with keyboard).

## Architecture

Everything is organized as sequential sections inside `index.html`, each marked with an HTML comment banner (`<!-- ===== ... ===== -->`):

1. **Navbar** (`.navbar`) — sticky header with a mobile hamburger toggle (`#navToggle` / `#navLinks`) driven by inline JS.
2. **Hero** (`#home`) — gradient banner, headline, two CTAs, and a trust-indicator strip.
3. **Services** (`#services`) — card grid; not in the original section list but added because the nav, hero CTA, and footer all link to it.
4. **Testimonials** (`#testimonials`) — card grid with CSS-only initials avatars (no image assets).
5. **Enquiry form** (`#enquiry`) — `#enquiryForm`, validated and submitted entirely client-side (see below).
6. **Footer** (`#contact`) — clinic info, quick links, placeholder social icons, JS-populated copyright year.

Key conventions to preserve when editing:

- **CSS custom properties** are defined once on `:root` at the top of the `<style>` block (`--primary`, `--accent`, `--text`, `--radius`, etc.). Change the palette/spacing there rather than hardcoding new values inline.
- **Mobile-first CSS**: base styles target small screens; `@media (min-width: ...)` blocks layer on wider-screen layout (nav, grids, form columns). Keep new responsive rules in the same pattern.
- **Scroll behavior**: navigation relies on `scroll-behavior: smooth` plus `scroll-margin-top` on `section` (to clear the sticky navbar) rather than JS-driven scrolling.
- **Scroll-triggered fade-in**: elements tagged `.fade-in` are revealed via a single `IntersectionObserver` in the script block; it short-circuits (no animation, immediate `.visible`) when `prefers-reduced-motion` is set or `IntersectionObserver` is unavailable. Add `.fade-in` to new section content rather than writing bespoke animation code.
- **Form handling is client-side only**: `#enquiryForm`'s submit handler in the script block does its own validation (name required, `EMAIL_RE`, `PHONE_RE`), toggles `aria-invalid` and per-field `.error-msg` elements, and on success builds a plain `formData` object and `console.log`s it. There is a comment marking exactly where a real `fetch(...)` POST to a backend would go — no network call is made. Don't wire up a real backend without being asked.
- **No external JS/CSS frameworks or icon libraries** — icons are inline `<svg>`, avatars are CSS-styled `<div>`s with initials. Keep new UI elements dependency-free and inline, consistent with the rest of the file.
