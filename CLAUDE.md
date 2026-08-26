# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A static German-language film review blog ("Filmtagebuch") by Mathis Worthmann. No build tools, no package manager, no JavaScript — plain HTML and CSS deployed to GitHub Pages at https://filmblog.mathisworthmann.de/.

The blog focuses on in-depth, analytical reviews of niche films — especially Kammerspiele, hard sci-fi, and found footage — not simple star ratings. Mathis attends Fantasy Filmfest and logs films on Letterboxd (handle: XerxesIV).

## Editorial Voice & Content Principles

All published content is written **in German**. The voice is defined and should be matched exactly when drafting or editing reviews:

- **Casual, personal, first person.** Directly addresses readers with "ihr/euch".
- **Honest.** Names a film's weaknesses openly, then argues why they do or don't matter.
- **Analytical, not just descriptive.** Goes beyond plot summary: uses frameworks (e.g. Vogler's hero's journey), makes original arguments (e.g. Marty as catalyst vs. George as the truly transformed hero in *Back to the Future*), and embeds production history / trivia ("Hintergrund").
- **Sourced.** Enriches claims with primary sources such as director interviews; verify facts rather than inventing them.
- Each review ends with a short personal verdict and a numeric/qualitative rating.

When writing about legal/privacy topics, always note that Mathis is not a lawyer and these are drafts to be checked.

## Working With Mathis

- **Execute directly.** He strongly prefers "just make the change" over being walked through how to do it himself.
- **Deliver complete, ready-to-commit files**, not fragments to assemble.
- **Iterate from his baseline.** When he shares existing HTML or an edited draft, match or improve from exactly that, don't start over.

## Development

No build step required. Open any `.html` file directly in a browser or serve with any static file server:

```bash
# Quick local server (Python)
python -m http.server 8080

# Or with Node.js
npx serve .
```

Deploy by pushing to `main` — GitHub Pages serves automatically (live after ~1 min).

## Architecture

Pure static site: one `.html` file per page, one `style.css` for all styling.

**Pages:**
- `index.html` — homepage, lists all reviews as cards
- `man-from-earth.html`, `deadstream.html`, `back-to-the-future.html` — individual review pages
- `kontakt.html` — contact page (mailto link; no form, no backend)
- `impressum.html`, `datenschutz.html` — German legal pages
- `404.html` — custom error page

**Assets:**
- `fonts/` — 8 self-hosted `.woff2` files (Playfair Display + Source Sans 3); no external font CDN
- `images/` — WebP for covers/photos, SVG for infographics, PNG for favicon

**SEO files:**
- `sitemap.xml` — lives in this repo at the root (`sitemap.xml`). **Not auto-generated** (no build step); must be updated by hand per new post.
- `robots.txt` — lives in this repo at the root (`robots.txt`); served at `filmblog.mathisworthmann.de/robots.txt`. It references the sitemap.
- `CNAME` — tells GitHub Pages to serve this repo under `filmblog.mathisworthmann.de`.

## CSS Design System

All theming lives in CSS custom properties at the top of `style.css` (after the `@font-face` block):

```css
--bg: #0c0a0e          /* dark page background */
--bg-card: #141218     /* card surfaces */
--accent: #e8b44d      /* gold highlight color */
--text: #e8e4ec        /* primary text */
--text-muted: #9a94a4  /* secondary text */
--font-display: 'Playfair Display', serif
--font-body: 'Source Sans 3', sans-serif
```

Responsive breakpoints: 768px (tablet) and 480px (mobile). Fluid typography via `clamp()`. Fonts are self-hosted via `@font-face` at the top of `style.css` — do not re-introduce the Google Fonts CDN (privacy reasons).

## Review Page Structure

Each review page follows a fixed template order:
1. Nav bar (`id="top"`) + back-to-overview link
2. Review metadata (number, date, star rating, "☕ Lesezeit" estimate)
3. Film details (year, director, genre, runtime)
4. Content sections with German `<h2>` headings (each has a left gold accent bar via CSS). Typical sections: "Worum geht's?", "Hintergrund", "Fazit", "Ähnliche Filme".
5. Images as `<figure>` with captions and copyright attribution
6. "Weitere Reviews" section + author bio card (with Letterboxd link) + footer

**Two different "related" sections — do not confuse them:**
- **"Ähnliche Filme"** links OUTWARD to three other films on Letterboxd (external recommendations).
- **"Weitere Reviews"** links INTERNALLY to Mathis's own other review pages (small cards, thumbnail + title). **Always exactly 3 cards**, showing the **3 reviews that came before this one**, descending (on Review #5: #4, #3, #2). If fewer than 3 predecessors exist, top the list up with the newest reviews that came *after*, appended at the end (on #3: #2, #1, #5; on #2: #1, #5, #4; on #1: #5, #4, #3). The cards are stacked in a single column (`.more-reviews-grid` is `grid-template-columns: 1fr`), not side by side.

All review pages include Schema.org JSON-LD `Review` data, a `<link rel="canonical">`, and full OpenGraph + Twitter Card meta tags.

## Adding a New Review

1. Copy an existing review page (e.g. `back-to-the-future.html`) as a template.
2. Write the review in the German voice described above.
3. Add the poster as a WebP to `images/`. Image files are named `<FilmInPascalCase>_<Motiv>.webp` — `Strafpark_Cover.webp`, `BackToTheFuture_DeLorean.webp`, `Flipped_Eggs.webp`. The prefix follows the film, not the page slug (the German-titled *Verliebt und ausgeflippt* uses `Flipped_*`), and every review needs a `_Cover.webp`.
4. Update the `<head>`: `<title>`, `meta description`, canonical, OG and Twitter tags, and the **JSON-LD block** (film name, director, year, rating).
5. Add a review card to `index.html`. **The reading time on the index card must match the detail page** (this once drifted — BttF said 15 min on the card, 12 in the page).
6. Rebuild the "Weitere Reviews" block on **every** review page per the rule above. Adding a review changes the top-up cards on the earliest pages (#1–#3), so re-check all of them, not just the neighbours.
7. Add a `<url>` entry (with `<lastmod>`) for the new page to `sitemap.xml`.
8. Use `loading="lazy" decoding="async"` on all `<img>` tags except the first hero/cover image.

## SEO & Metadata Conventions

- Canonical, OG, `og:locale=de_DE`, and Twitter Card tags on every page. Keep `og:url`/canonical pointing at the **real filename** (a past bug had `og:url` pointing at a non-existent `zurueck-in-die-zukunft.html`).
- JSON-LD `Review` schema is a **second source of truth** for rating/film data. When a rating or reading time changes, update BOTH the visible HTML and the JSON-LD.

## Accessibility & Performance Conventions

- Back-to-top is a real anchor (`<a href="#top">`), and the nav carries `id="top"`. Do not revert to an `onclick`-only link.
- `:focus-visible` outlines exist for keyboard navigation — keep them.
- Images below the fold are lazy-loaded; the hero/cover stays eager.

## Legal & Privacy (Germany)

- `impressum.html` covers § 5 DDG and names the § 18 Abs. 2 MStV content-responsible person. `datenschutz.html` is a DSGVO privacy policy covering GitHub Pages hosting (IP logs, US transfer, Data Privacy Framework), and explicitly states: no cookies, no tracking, no third-party fonts.
- These are **non-lawyer drafts** flagged for review (eRecht24 / IHK / a lawyer).
- If analytics, a contact form, comments, or any third-party script is ever added, the Datenschutzerklärung MUST be extended accordingly. Self-hosting the fonts was a deliberate privacy choice — keep it that way.

## Footer & Nav Consistency

Nav and footer are duplicated across every page (no includes/build step). A change to either must be replicated to ALL pages.
- **Nav links:** Reviews · Kontakt · ← Portfolio
- **Footer links:** Impressum · Datenschutz · Portfolio →

## Outstanding TODOs

- Submit the site + sitemap to Google Search Console (and Bing Webmaster Tools).
- Add intrinsic `width`/`height` to `<img>` tags (real pixel dimensions needed) to reduce layout shift.
- Audit which font weights are actually used and drop unused ones.
- Review color contrast of `--text-muted` on dark backgrounds (near WCAG AA edge).
- If posting frequency increases: consider migrating to Jekyll (`jekyll-sitemap` would auto-generate the sitemap and `_includes` would solve the duplicated nav/footer).
