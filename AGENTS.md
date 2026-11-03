# AGENTS.md

🚨 MANDATORY: YOU MUST CALL "learn_shopify_api" ONCE WHEN WORKING WITH LIQUID THEMES. Use the Shopify AI Toolkit for all Shopify API/platform work (install: `npx skills add Shopify/shopify-ai-toolkit --list`).

## What this is

- Shopify Skeleton Theme (`theme_name: Skeleton`, v0.1.0 in `config/settings_schema.json:4-10`), plain Liquid — no build step, no bundler.
- No `package.json`, no tests, no CI in this repo: do not run `npm test`/`npm run lint` here (that is the parent workspace's `../AGENTS.md`, not this theme). Keep changes lean/minimal per `CONTRIBUTING.md:13-16`; no legacy features.

## Commands (Shopify CLI 5.0.0 installed)

- Verify (only offline check): `shopify theme check` — uses `.theme-check.yml` (`theme-check:recommended`); currently passes: 39 files, no offenses. Run after every Liquid/JSON change.
- Preview: `shopify theme dev` — needs an authenticated store, uploads a dev theme and live-syncs; confirm the target store before running.
- `shopify theme push`/`pull`/`publish` overwrite remote/local state — never run without an explicit store/theme target from the user.

## Wiring

- Entry: `layout/theme.liquid` renders `snippets/css-variables.liquid`, preloads `assets/critical.css`, renders `snippets/meta-tags.liquid`, then `header-group` / `content_for_layout` / `footer-group`. `layout/password.liquid` only for `templates/password.json`.
- `templates/*.json` are thin: each maps `"main"` to the same-named section (`index.json` → `sections/hello-world.liquid`). Prefer generating sections/blocks/snippets; merchants assemble templates in the theme editor.
- Auto-generated JSONC — `templates/*.json`, `sections/header-group.json`, `sections/footer-group.json`, `config/settings_data.json` carry an "auto-generated" header and may be overwritten by the theme editor. They contain comments (`.gitattributes`: `*.json linguist-language=jsonc`), so strict `json.load`/`jq` fails on them; validate with `shopify theme check`, don't "fix" the comments.

## Authoring rules

- Sections/blocks: required `{% schema %}` (exposes editor settings). Snippets, and blocks rendered statically via `{% content_for 'block', ... %}`, require a `{% doc %}` header with `@param`/`@example`.
- CSS/JS per component via `{% stylesheet %}`/`{% javascript %}` (only in `snippets/`, `blocks/`, `sections/`; one each; no Liquid inside them). Only every-page CSS goes in `assets/critical.css`; component CSS stays in the component.
- Schema settings: single CSS property → CSS variable (`style="--gap: {{ block.settings.gap }}px"`); multiple properties → a CSS class stored as the setting value.
- User-facing text: `{{ 'key' | t }}`, add keys only to `locales/en.default.json` (editor labels via `t:general.`/`t:labels.`/`t:options.` in `locales/en.default.schema.json`); hierarchical `snake_case`, max 3 levels, sentence case. Existing `hello-world`/`header`/`product` sections hardcode text — don't copy that pattern.
- Liquid gotchas: no parentheses or ternaries — nest `{% if %}` when combining >1 operator; `for` caps at 50 iterations, use `{% paginate %}` beyond that; `{% render %}` snippets get isolated scope — pass values as parameters.

## Repo quirks

- `copilot-instructions.md` is a dangling symlink to a path outside this repo — ignore it; this file is the instruction source.
- `.shopifyignore` is all comments: nothing is excluded from CLI push/dev.
- Default branch: `main`; remote is a fork (`barbaracohenguilhem/tumblr`), not `Shopify/skeleton-theme` despite the README badge.
