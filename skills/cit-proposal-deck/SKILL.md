---
name: cit-proposal-deck
description: Use this skill whenever the user wants to create, draft, edit, or assemble an HTML presentation, slide deck, proposal, or pitch deck for CI&T — or whenever they mention "CI&T deck", "CI&T presentation", "CI&T proposal", "Proposal Hub", "Proposal Hub upload", or any request to author a presentation that follows CI&T branding. Triggers even when the user does not explicitly say "HTML" — if the destination is Proposal Hub or the brand context is CI&T, use this skill. Produces a single self-contained .html file at 1280×720 with the CI&T palette (coral / navy / sky / burgundy / pink), Inter typography, and exactly the structure Proposal Hub's parser expects so the deck uploads with no manual adjustments.
---

# CI&T Proposal Hub — Deck Authoring Skill

Use this skill to author HTML presentations that upload cleanly into CI&T's Proposal Hub. Following the conventions below produces decks that parse, render, and export identically in the browser and inside the tool.

## How to use this skill

When this skill triggers:

1. **Confirm scope quickly.** Before generating slides, confirm with the user: deck topic, target audience, approximate slide count, and any content they want included. Don't go deeper than ~3 questions — the goal is just to avoid producing the wrong deck.
2. **Start from the minimal template** in the "Minimal working template" section below. Copy it, then add slides by duplicating the slide pattern.
3. **Use brand tokens, not hardcoded colors.** Always reference `var(--coral)`, `var(--navy)`, etc. — defined once in `:root`.
4. **One file, no external assets.** The deliverable is a single `.html` file. No external CSS, no external image URLs, no JavaScript dependencies. Inline SVG and base64 data URIs only.
5. **Run the pre-upload checklist** at the end of this skill before handing the file to the user.
6. **Deliver the file.** Save the `.html` to disk and present it to the user. Do not paste 700 lines of HTML into chat — the file IS the artifact.

---

## How Proposal Hub reads the HTML

Proposal Hub parses the uploaded HTML file to extract:

- **Slides** — every slide element becomes an independently editable, individually stored slide.
- **Global styles** — all CSS is captured and applied inside an isolated rendering context (Shadow DOM) so styles never conflict with the app.
- **Canvas dimensions** — detected from the CSS rule that defines the slide class width and height.

The parser uses a priority cascade to find slides:

| Priority | Pattern | Example |
|---|---|---|
| 1 | Reveal.js `.reveal .slides > section` | Reveal decks work out of the box |
| 2 | `<section>` siblings | Standard semantic HTML |
| 3 | Elements with class `slide` or `page` at the top two levels of the body | **Recommended pattern** |
| 4 | Largest group of siblings sharing any common class | Fallback for custom class names |
| 5 | `<h1>`/`<h2>` heading splits | Document-style HTML |
| 6 | Entire `<body>` as one slide | Last resort |

**Always use the recommended pattern (priority 3).** It is explicit, predictable, and removes guesswork from the parser. Don't try to be clever with priority 1 or 4 unless the user specifically asks.

---

## Required HTML structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Your Presentation Title · CI&T</title>

  <!-- Fonts must be loaded via <link> — @import inside <style> is not supported -->
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&display=swap" rel="stylesheet" />

  <style>
    /* All CSS goes here — no external .css files */

    :root {
      /* Brand tokens — define once, use everywhere */
    }

    .slide {
      /* REQUIRED: explicit pixel dimensions so the tool detects the design canvas */
      width: 1280px;
      height: 720px;

      /* Required layout */
      position: absolute;
      top: 0; left: 0;
      overflow: hidden;
    }

    /* Per-slide rules use the slide ID as a scope: #s1 .title { ... } */
  </style>
</head>
<body>

  <div id="deck">
    <div class="slide" id="s1"><!-- slide 1 content --></div>
    <div class="slide" id="s2"><!-- slide 2 content --></div>
    <div class="slide" id="s3"><!-- slide 3 content --></div>
  </div>

</body>
</html>
```

### Rules that must be followed

| Rule | Why |
|---|---|
| All slide elements must be **siblings sharing one class** | The parser finds slides by detecting the largest sibling group with a common class |
| The slide class must define **`width` and `height` in px** | Used to auto-detect the design canvas size for scaling |
| Each slide should have a **unique `id`** (`s1`, `s2`, …) | Lets you scope CSS with `#s1 .heading { }` without affecting other slides |
| All CSS goes inside **`<style>` tags in `<head>`** | External `.css` files are not fetched |
| Images must be **inline `<svg>` or base64 `data:` URIs** | External image URLs are not guaranteed to be reachable after upload |
| **No JavaScript needed** — scripts are stripped | The tool renders slides statically; JS slideshow runners are ignored |
| **Do not use `display: none`** on slide elements | The tool strips it to show all slides simultaneously in the editor |

---

## Minimal working template

Copy this as the starting point. Every slide is self-contained with a background layer, a logo, a slide number, and a content area.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Presentation Title · CI&T</title>
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&display=swap" rel="stylesheet" />
<style>

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --coral:  #F05535;
  --navy:   #0B1461;
  --sky:    #AED6F1;
  --burg:   #820047;
  --pink:   #F2A0CC;
  --white:  #FFFFFF;
  --light:  #F5F5F5;
  --muted:  rgba(11, 20, 97, 0.12);
}

/* ── Slide canvas ── */
.slide {
  position: absolute;
  top: 0; left: 0;
  width: 1280px;
  height: 720px;
  overflow: hidden;
  font-family: 'Inter', sans-serif;
}

/* ── Shared components ── */
.bg      { position: absolute; inset: 0; }
.logo    { position: absolute; top: 36px; right: 48px; height: 22px; width: auto;
           color: var(--coral); display: block; }
.logo svg { height: 100%; width: auto; display: block; }
.snum    { position: absolute; bottom: 36px; right: 48px; font-size: 11px; font-weight: 600;
           letter-spacing: 0.12em; text-transform: uppercase; opacity: 0.4; }

/* ── Slide 1 ── */
#s1 .bg { background: var(--navy); }
#s1 .content {
  position: absolute;
  left: 120px; top: 50%;
  transform: translateY(-50%);
  max-width: 760px;
}
#s1 .eyebrow {
  font-size: 12px; font-weight: 700; letter-spacing: 0.18em;
  text-transform: uppercase; color: rgba(255,255,255,0.5); margin-bottom: 20px;
}
#s1 h1 { font-size: 72px; font-weight: 900; line-height: 1; color: var(--white); margin-bottom: 20px; }
#s1 h1 em { font-style: normal; color: var(--coral); }
#s1 .sub { font-size: 20px; color: rgba(255,255,255,0.65); line-height: 1.5; max-width: 560px; }

/* ── Slide 2+ ── */
#s2 .bg { background: var(--light); }
#s2 .content {
  position: absolute;
  left: 120px; right: 120px; top: 100px;
}
#s2 h2 { font-size: 40px; font-weight: 800; color: var(--navy); margin-bottom: 32px; }
#s2 p  { font-size: 18px; color: rgba(11,20,97,0.65); line-height: 1.6; }

</style>
</head>
<body>
<div id="deck">

  <!-- SLIDE 1 — COVER -->
  <div class="slide" id="s1">
    <div class="bg"></div>
    <div class="content">
      <p class="eyebrow">CI&amp;T · Your Division</p>
      <h1>Your Proposal<br><em>Title Here</em></h1>
      <p class="sub">A one-liner description of what this deck is about.</p>
    </div>
    <div class="logo">
      <svg viewBox="0 0 772.67 248.96" xmlns="http://www.w3.org/2000/svg" aria-label="CI&amp;T">
        <g fill="currentColor">
          <path d="m0,125.12C0,47.38,55.29.63,130.17.63c60.66,0,108.38,30.02,122.28,89.41h-58.13c-6.63-36.33-28.13-62.55-63.19-62.55-42.65,0-68.25,40.12-68.25,90.05,0,56.87,28.44,92.57,77.09,92.57,43.93,0,72.36-28.44,80.25-72.35h33.81c-9.47,68.25-57.19,111.21-127.96,111.21C49.93,248.96,0,198.42,0,125.12Z"/>
          <path d="m266.73,242.65c11.38-8.84,13.91-23.07,13.91-48.03V54.03c0-25.59-2.53-39.18-13.91-48.03v-1.27h87.52v1.27c-11.36,8.84-13.9,22.43-13.9,48.03v140.59c0,24.95,2.53,39.18,13.9,48.03v1.26h-87.52v-1.26Z"/>
          <path d="m561.94,4.73h210.73v44.23c-34.75-13.59-52.76-19.9-73.93-19.9h-1.58v165.56c0,24.95,2.53,39.18,13.91,48.03v1.26h-87.21v-1.26c11.38-8.84,13.9-23.07,13.9-48.03V29.07h-1.26c-21.48,0-39.49,6.31-74.56,19.9V4.73Z"/>
          <path d="m366.94,179.93c0-33.58,22.18-57.02,56.71-70.32-15.21-17.43-24.08-32.95-24.08-51.96,0-36.11,30.1-57.65,69.37-57.65,43.09,0,70.34,27.56,73.82,69.69h-44.35c-.31-26.61-9.5-48.15-30.1-48.15-14.89,0-24.08,9.82-24.08,25.35s9.19,29.46,35.48,56.07l53.22,53.54c3.17-13.94,5.07-29.14,5.39-44.35l47.52-.32v7.6c-10.45,20.59-20.92,40.55-32.31,58.28l61.13,61.45v4.76h-60.19l-29.14-29.79c-20.58,20.92-45.61,34.22-78.56,34.22-46.89,0-79.83-26.62-79.83-68.43Zm102.33,36.42c18.06,0,31.68-6.33,42.13-16.78l-74.45-75.71c-13.93,9.82-20.6,24.71-20.6,41.18,0,29.77,20.6,51.31,52.91,51.31Z"/>
        </g>
      </svg>
    </div>
    <div class="snum" style="color:rgba(255,255,255,0.3)">01 / 02</div>
  </div>

  <!-- SLIDE 2 — CONTENT -->
  <div class="slide" id="s2">
    <div class="bg"></div>
    <div class="content">
      <h2>Section Title</h2>
      <p>Your content goes here. Use absolute positioning for precise layout control.</p>
    </div>
    <div class="logo">
      <svg viewBox="0 0 772.67 248.96" xmlns="http://www.w3.org/2000/svg" aria-label="CI&amp;T">
        <g fill="currentColor">
          <path d="m0,125.12C0,47.38,55.29.63,130.17.63c60.66,0,108.38,30.02,122.28,89.41h-58.13c-6.63-36.33-28.13-62.55-63.19-62.55-42.65,0-68.25,40.12-68.25,90.05,0,56.87,28.44,92.57,77.09,92.57,43.93,0,72.36-28.44,80.25-72.35h33.81c-9.47,68.25-57.19,111.21-127.96,111.21C49.93,248.96,0,198.42,0,125.12Z"/>
          <path d="m266.73,242.65c11.38-8.84,13.91-23.07,13.91-48.03V54.03c0-25.59-2.53-39.18-13.91-48.03v-1.27h87.52v1.27c-11.36,8.84-13.9,22.43-13.9,48.03v140.59c0,24.95,2.53,39.18,13.9,48.03v1.26h-87.52v-1.26Z"/>
          <path d="m561.94,4.73h210.73v44.23c-34.75-13.59-52.76-19.9-73.93-19.9h-1.58v165.56c0,24.95,2.53,39.18,13.91,48.03v1.26h-87.21v-1.26c11.38-8.84,13.9-23.07,13.9-48.03V29.07h-1.26c-21.48,0-39.49,6.31-74.56,19.9V4.73Z"/>
          <path d="m366.94,179.93c0-33.58,22.18-57.02,56.71-70.32-15.21-17.43-24.08-32.95-24.08-51.96,0-36.11,30.1-57.65,69.37-57.65,43.09,0,70.34,27.56,73.82,69.69h-44.35c-.31-26.61-9.5-48.15-30.1-48.15-14.89,0-24.08,9.82-24.08,25.35s9.19,29.46,35.48,56.07l53.22,53.54c3.17-13.94,5.07-29.14,5.39-44.35l47.52-.32v7.6c-10.45,20.59-20.92,40.55-32.31,58.28l61.13,61.45v4.76h-60.19l-29.14-29.79c-20.58,20.92-45.61,34.22-78.56,34.22-46.89,0-79.83-26.62-79.83-68.43Zm102.33,36.42c18.06,0,31.68-6.33,42.13-16.78l-74.45-75.71c-13.93,9.82-20.6,24.71-20.6,41.18,0,29.77,20.6,51.31,52.91,51.31Z"/>
        </g>
      </svg>
    </div>
    <div class="snum" style="color:rgba(11,20,97,0.3)">02 / 02</div>
  </div>

</div>
</body>
</html>
```

---

## CI&T brand tokens

### Colours

Define these CSS variables in `:root`. They are the official CI&T palette — don't approximate them.

| Token | Hex | Usage |
|---|---|---|
| `--coral` | `#F05535` | Primary accent — CTAs, highlights, logo `&`, key numbers |
| `--navy` | `#0B1461` | Primary dark — cover backgrounds, dark-theme slides |
| `--sky` | `#AED6F1` | Secondary accent — decorative shapes, dividers |
| `--burg` | `#820047` | Alternate dark — section covers, contrast headers |
| `--pink` | `#F2A0CC` | Soft accent — supporting visuals, light-theme highlights |
| `--white` | `#FFFFFF` | Text on dark backgrounds |
| `--light` | `#F5F5F5` | Light slide backgrounds |
| `--muted` | `rgba(11,20,97,0.12)` | Subtle dividers, card borders on light backgrounds |

### Typography

**Font family:** Inter (Google Fonts). Load via `<link>`, never via `@import` inside `<style>`.

```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&display=swap" rel="stylesheet" />
```

| Role | Size | Weight | Notes |
|---|---|---|---|
| Cover headline | 64–80px | 900 | Line-height 1.0 |
| Section title | 40–48px | 800 | Line-height 1.1 |
| Body heading | 28–32px | 700 | |
| Body text | 16–20px | 400 | Line-height 1.5–1.6 |
| Eyebrow / label | 11–13px | 700 | `letter-spacing: 0.14–0.18em`, `text-transform: uppercase` |
| Footnote / caption | 11–12px | 400–500 | Opacity 0.5–0.6 |

### Logo

The CI&T logo is the **official inline SVG logotype** — always coral (`var(--coral)`), on every slide, on every background. Do not type "CI&T" as styled text and do not split the colors (white "CI" + coral "&" + white "T" is wrong). Do not import the logo as an external image either — embed the SVG inline so it travels with the file.

The SVG below uses `fill="currentColor"`, so the wrapper's `color: var(--coral)` paints the entire mark coral. To recolor for special slides, change the wrapper's `color`. The mark stays coral on both dark (navy) and light (`--light`) backgrounds — that is intentional.

```html
<div class="logo">
  <svg viewBox="0 0 772.67 248.96" xmlns="http://www.w3.org/2000/svg" aria-label="CI&amp;T">
    <g fill="currentColor">
      <path d="m0,125.12C0,47.38,55.29.63,130.17.63c60.66,0,108.38,30.02,122.28,89.41h-58.13c-6.63-36.33-28.13-62.55-63.19-62.55-42.65,0-68.25,40.12-68.25,90.05,0,56.87,28.44,92.57,77.09,92.57,43.93,0,72.36-28.44,80.25-72.35h33.81c-9.47,68.25-57.19,111.21-127.96,111.21C49.93,248.96,0,198.42,0,125.12Z"/>
      <path d="m266.73,242.65c11.38-8.84,13.91-23.07,13.91-48.03V54.03c0-25.59-2.53-39.18-13.91-48.03v-1.27h87.52v1.27c-11.36,8.84-13.9,22.43-13.9,48.03v140.59c0,24.95,2.53,39.18,13.9,48.03v1.26h-87.52v-1.26Z"/>
      <path d="m561.94,4.73h210.73v44.23c-34.75-13.59-52.76-19.9-73.93-19.9h-1.58v165.56c0,24.95,2.53,39.18,13.91,48.03v1.26h-87.21v-1.26c11.38-8.84,13.9-23.07,13.9-48.03V29.07h-1.26c-21.48,0-39.49,6.31-74.56,19.9V4.73Z"/>
      <path d="m366.94,179.93c0-33.58,22.18-57.02,56.71-70.32-15.21-17.43-24.08-32.95-24.08-51.96,0-36.11,30.1-57.65,69.37-57.65,43.09,0,70.34,27.56,73.82,69.69h-44.35c-.31-26.61-9.5-48.15-30.1-48.15-14.89,0-24.08,9.82-24.08,25.35s9.19,29.46,35.48,56.07l53.22,53.54c3.17-13.94,5.07-29.14,5.39-44.35l47.52-.32v7.6c-10.45,20.59-20.92,40.55-32.31,58.28l61.13,61.45v4.76h-60.19l-29.14-29.79c-20.58,20.92-45.61,34.22-78.56,34.22-46.89,0-79.83-26.62-79.83-68.43Zm102.33,36.42c18.06,0,31.68-6.33,42.13-16.78l-74.45-75.71c-13.93,9.82-20.6,24.71-20.6,41.18,0,29.77,20.6,51.31,52.91,51.31Z"/>
    </g>
  </svg>
</div>
```

```css
.logo {
  position: absolute;
  top: 36px; right: 48px;
  height: 22px;          /* default scale; bump to 26–28px for cover slides if needed */
  width: auto;
  color: var(--coral);   /* SVG fills inherit via currentColor */
  display: block;
}
.logo svg { height: 100%; width: auto; display: block; }
```

Standard position: `top: 36px; right: 48px;` on every slide. Standard size: `height: 22px`. Cover slides may use `height: 26–32px` for more presence.

#### BAD — do not do this

```html
<!-- Wrong: typed text. Inter glyphs are not the brand logotype. -->
<div class="logo">CI<span>&amp;</span>T</div>
```

```css
/* Wrong: split colors. The official logo is fully coral, not white CI + coral & + white T. */
.logo { color: var(--white); }
.logo span { color: var(--coral); }
```

### Slide dimensions

| Property | Value |
|---|---|
| Width | `1280px` |
| Height | `720px` |
| Aspect ratio | `16: