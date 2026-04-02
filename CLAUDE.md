# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UI/UX Pro Max (v2.5.0) is an AI-powered design intelligence toolkit providing searchable databases of UI styles, color palettes, font pairings, chart types, UX guidelines, and stack-specific recommendations. It works as a skill/workflow for 19 AI coding platforms (Claude Code, Cursor, Windsurf, Copilot, Kiro, Roocode, KiloCode, Codex, Qoder, Gemini, Trae, OpenCode, Continue, CodeBuddy, Droid, Warp, Augment, Agent, Antigravity).

**Data at a glance:** ~84 UI styles, 161 color palettes, 57 font pairings, 161 product types, 99 UX guidelines, 25 chart types, 16 tech stacks.

## Search Command

```bash
python3 src/ui-ux-pro-max/scripts/search.py "<query>" --domain <domain> [-n <max_results>]
```

**Domain search:**
- `product` — Product type recommendations (SaaS, e-commerce, portfolio)
- `style` — UI styles (glassmorphism, minimalism, brutalism) + AI prompts and CSS keywords
- `typography` — Font pairings with Google Fonts imports
- `color` — Color palettes by product type
- `landing` — Page structure and CTA strategies
- `chart` — Chart types and library recommendations
- `ux` — Best practices and anti-patterns
- `google-fonts` — Google Fonts database (~1,900 entries)
- `icon` — Icon set recommendations

**Stack search:**
```bash
python3 src/ui-ux-pro-max/scripts/search.py "<query>" --stack <stack>
```
Available stacks (16): `html-tailwind` (default), `react`, `nextjs`, `astro`, `vue`, `nuxtjs`, `nuxt-ui`, `svelte`, `angular`, `swiftui`, `react-native`, `flutter`, `shadcn`, `jetpack-compose`, `threejs`, `laravel`

**Design system generation:**
```bash
python3 src/ui-ux-pro-max/scripts/search.py "<query>" --design-system [--persist] [--page <page>]
```

## Architecture

```
src/ui-ux-pro-max/                  # Source of Truth
├── data/                           # Canonical CSV databases (14 core files)
│   ├── products.csv, styles.csv, colors.csv, typography.csv, ...
│   ├── google-fonts.csv            # ~1,900 Google Fonts entries
│   ├── design.csv, draft.csv       # Design reasoning data
│   ├── ui-reasoning.csv            # 161 reasoning rules
│   ├── _sync_all.py                # Sync utility
│   └── stacks/                     # Stack-specific guidelines (16 CSVs)
├── scripts/
│   ├── search.py                   # CLI entry point (domain, stack, design-system modes)
│   ├── core.py                     # BM25 + regex hybrid search engine
│   └── design_system.py            # Design system generation (Master + Overrides pattern)
└── templates/
    ├── base/                       # Base templates
    │   ├── skill-content.md        # Common SKILL.md content
    │   └── quick-reference.md      # Quick reference section (Claude only)
    └── platforms/                  # Platform-specific configs (18 JSON files)
        ├── claude.json, cursor.json, windsurf.json, copilot.json, ...
        └── (agent, augment, codebuddy, codex, continue, droid,
             gemini, kilocode, kiro, opencode, qoder, roocode, trae, warp)

cli/                                # CLI installer (uipro-cli on npm, v2.5.0)
├── src/
│   ├── index.ts                    # Entry point
│   ├── commands/
│   │   ├── init.ts                 # Install command with template generation
│   │   ├── uninstall.ts            # Uninstall command
│   │   ├── update.ts               # Update command
│   │   └── versions.ts             # Version info command
│   ├── utils/
│   │   ├── template.ts             # Template rendering engine
│   │   ├── detect.ts               # Platform detection
│   │   ├── extract.ts              # Asset extraction
│   │   ├── github.ts               # GitHub integration
│   │   └── logger.ts               # Logging utilities
│   └── types/index.ts              # TypeScript type definitions
├── assets/                         # Bundled assets (~564KB, copies of src/)
│   ├── data/                       # Copy of src/ui-ux-pro-max/data/
│   ├── scripts/                    # Copy of src/ui-ux-pro-max/scripts/
│   └── templates/                  # Copy of src/ui-ux-pro-max/templates/
└── package.json                    # ES Module, builds with Bun

.claude/skills/                     # Claude Code skills
├── ui-ux-pro-max/                  # Main skill (symlinks data/ and scripts/ to src/)
│   ├── SKILL.md
│   ├── data -> ../../../src/ui-ux-pro-max/data
│   └── scripts -> ../../../src/ui-ux-pro-max/scripts
├── banner-design/                  # Banner design skill
├── brand/                          # Brand identity skill (with Node.js scripts)
├── design/                         # Comprehensive design skill (logo, CIP, icon generation)
├── design-system/                  # Design tokens & component specifications
├── slides/                         # HTML presentation skill
└── ui-styling/                     # shadcn/ui + Tailwind styling (includes 54 canvas fonts)

.claude-plugin/                     # Claude Marketplace publishing
├── plugin.json                     # Skill manifest for marketplace
└── marketplace.json                # Publishing configuration

.github/workflows/
├── claude.yml                      # Claude Code action (issue/PR assistance)
└── claude-code-review.yml          # Automated PR code review
```

The search engine uses BM25 ranking combined with regex matching. Domain auto-detection is available when `--domain` is omitted.

## Sync Rules

**Source of Truth:** `src/ui-ux-pro-max/`

When modifying files:

1. **Data & Scripts** — Edit in `src/ui-ux-pro-max/`:
   - `data/*.csv` and `data/stacks/*.csv`
   - `scripts/*.py`
   - Changes automatically available via symlinks in `.claude/skills/ui-ux-pro-max/`

2. **Templates** — Edit in `src/ui-ux-pro-max/templates/`:
   - `base/skill-content.md` — Common SKILL.md content
   - `base/quick-reference.md` — Quick reference section (Claude only)
   - `platforms/*.json` — Platform-specific configs (18 platforms)

3. **CLI Assets** — Run sync before publishing:
   ```bash
   cp -r src/ui-ux-pro-max/data/* cli/assets/data/
   cp -r src/ui-ux-pro-max/scripts/* cli/assets/scripts/
   cp -r src/ui-ux-pro-max/templates/* cli/assets/templates/
   ```

4. **Other Skills** — Skills in `.claude/skills/` other than `ui-ux-pro-max/` have their own independent data, scripts, and references. Edit them directly.

## CLI Development

```bash
cd cli
bun run build          # Build: bun build src/index.ts --outdir dist --target node
bun run dev            # Dev mode
npm publish            # Publishes as uipro-cli (runs prepublishOnly → build)
```

Usage: `npx uipro-cli init --ai <platform>`

## Prerequisites

Python 3.x (no external dependencies required for search engine)

## Testing

Tests exist for the `ui-styling` skill only:
```bash
cd .claude/skills/ui-styling/scripts/tests
python3 -m pytest test_shadcn_add.py test_tailwind_config_gen.py
```

## Git Workflow

Never push directly to `main`. Always:

1. Create a new branch: `git checkout -b feat/...` or `fix/...`
2. Commit changes
3. Push branch: `git push -u origin <branch>`
4. Create PR
