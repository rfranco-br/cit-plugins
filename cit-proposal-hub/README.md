# CI&T Proposal Hub Plugin

A Claude plugin that turns Claude into a co-author for CI&T Proposal Hub presentations. Ask Claude to draft a deck and it follows CI&T's official structure, brand tokens, and pre-upload checklist — producing a single self-contained `.html` file that uploads cleanly into Proposal Hub with no manual adjustments.

## What's inside

- **Skill: `cit-proposal-deck`** — The full Proposal Hub authoring guide, packaged so Claude consults it whenever a CI&T deck/proposal/presentation comes up. Covers required HTML structure, the minimal working template, brand colors and typography, layout patterns, image embedding, common pitfalls, and a pre-upload checklist.

## Install

### Claude Code / Cowork

1. Place this folder (or the `.zip`) somewhere Claude Code can read it.
2. From the Claude Code CLI, run:
   ```
   /plugin install ./cit-proposal-hub
   ```
   …or use your plugin marketplace's install flow.
3. The skill auto-loads. No further configuration.

### Verifying the install

Ask Claude:

> Make me a 5-slide CI&T proposal deck for a new fintech client.

If the plugin is installed correctly, Claude consults the `cit-proposal-deck` skill, asks 1–2 scoping questions, and produces a single `.html` file using the brand tokens and structure required by Proposal Hub.

## What the skill does for you

- Forces the recommended slide structure (`<div class="slide" id="sN">` siblings) — the parser pattern Proposal Hub trusts most.
- Pins the canvas to `1280×720` so the tool detects the right design size.
- Always uses the official palette: `--coral #F05535`, `--navy #0B1461`, `--sky #AED6F1`, `--burg #820047`, `--pink #F2A0CC`, plus `--white`, `--light`, `--muted`.
- Loads Inter via `<link>` (never `@import`).
- Embeds images as inline SVG or base64 — no external URLs.
- Emits no JavaScript and no `display: none` (both cause upload glitches).
- Walks a pre-upload checklist before handing the file over.

## Files

```
cit-proposal-hub/
├── .claude-plugin/
│   └── plugin.json
├── README.md
└── skills/
    └── cit-proposal-deck/
        └── SKILL.md
```

## License & ownership

This plugin packages the CI&T Proposal Hub authoring conventions for use with Claude. The brand tokens, structural rules, and palette belong to CI&T. Redistribute internally as needed.
