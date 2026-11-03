# AGENTS.md

🚨 MANDATORY: Call `learn_shopify_api` exactly once per conversation, before the first read or edit of any `.liquid` or `.json` file in this theme. Use the Shopify AI Toolkit for all Shopify API/platform work. Install: `npx skills add Shopify/shopify-ai-toolkit`. To only view available skills, run the same command with `--list`.

If `learn_shopify_api` is unavailable or returns an error, tell the user, do not retry more than once, and continue only with the offline checks described in this file. If the Shopify AI Toolkit is not available, state that limitation to the user before proceeding with any API or platform change.

## What this is

- Shopify Skeleton Theme (`theme_name: Skeleton`, v0.1.0 in `config/settings_schema.json:4-10`), plain Liquid — no build step, no bundler.
- No `package.json`, no tests, no CI in this repo: do not run `npm test`/`npm run lint` here (that is the parent workspace's `../AGENTS.md`, not this theme). Keep changes lean/minimal per `CONTRIBUTING.md:13-16`; no legacy features.

## Commands (Shopify CLI 5.0.0 installed)

- Verify (only offline check): `shopify theme check` — uses `.theme-check.yml` (`theme-check:recommended`); currently passes: 39 files, no offenses. Run after every Liquid/JSON change. If `shopify theme check` reports offenses, fix them in the changed files and rerun the check. If an offense is in an auto-generated JSONC file, report it to the user instead of editing the file's comments.
- Preview: `shopify theme dev` — needs an authenticated store, uploads a dev theme and live-syncs. Before running `shopify theme dev`, ask the user to state the store domain and wait for confirmation. Do not infer the store from config files. If the CLI is not authenticated or the user has not named a store, do not run `shopify theme dev`. Ask the user to run `shopify auth login` and provide the store domain.
- `shopify theme push`/`pull`/`publish` overwrite remote/local state — never run without an explicit store/theme target from the user.

## Wiring

- Entry: `layout/theme.liquid` renders `snippets/css-variables.liquid`, preloads `assets/critical.css`, renders `snippets/meta-tags.liquid`, then `header-group` / `content_for_layout` / `footer-group`. `layout/password.liquid` only for `templates/password.json`.
- `templates/*.json` are thin: each maps `"main"` to the same-named section (`index.json` → `sections/hello-world.liquid`). Prefer generating sections/blocks/snippets; merchants assemble templates in the theme editor.
- Auto-generated JSONC — `templates/*.json`, `sections/header-group.json`, `sections/footer-group.json`, `config/settings_data.json` carry an "auto-generated" header and may be overwritten by the theme editor. They contain comments (`.gitattributes`: `*.json linguist-language=jsonc`), so strict `json.load`/`jq` fails on them; validate with `shopify theme check`, don't "fix" the comments.

## Authoring rules

- Sections/blocks: required `{% schema %}` (exposes editor settings). Snippets, and blocks rendered statically via `{% content_for 'block', ... %}`, require a `{% doc %}` header with `@param`/`@example`.
- CSS/JS per component via `{% stylesheet %}`/`{% javascript %}` (only in `snippets/`, `blocks/`, `sections/`; one each; no Liquid inside them). Only every-page CSS goes in `assets/critical.css`; component CSS stays in the component.
- Schema settings: single CSS property → CSS variable (`style="--gap: {{ block.settings.gap }}px"`); multiple properties → a CSS class stored as the setting value.
- User-facing text: `{{ 'key' | t }}`, add keys only to `locales/en.default.json` (editor labels via `t:general.`/`t:labels.`/`t:options.` in `locales/en.default.schema.json`). Translation keys: hierarchical `snake_case`, max 3 levels. Translation values: sentence case. Existing `hello-world`/`header`/`product` sections hardcode text — don't copy that pattern.
- Liquid gotchas: no parentheses or ternaries — nest `{% if %}` when combining >1 operator; `for` caps at 50 iterations, use `{% paginate %}` beyond that; `{% render %}` snippets get isolated scope — pass values as parameters.

## Liquid constraints (verify before finishing)

- No parentheses or ternaries; nest `{% if %}` when combining multiple operators.
- `for` loops cap at 50 iterations; use `{% paginate %}` beyond that.
- `{% render %}` snippets have isolated scope; pass required values as parameters.
- Sections and blocks require `{% schema %}`; snippets and statically-rendered blocks require `{% doc %}` headers with `@param`/`@example`.
- CSS/JS scoping: `{% stylesheet %}`/`{% javascript %}` only in `snippets/`, `blocks/`, `sections/`; one each, no Liquid inside them.
- Only every-page CSS goes in `assets/critical.css`; component CSS stays in the component.
- Schema settings: single CSS property → CSS variable; multiple properties → CSS class stored as setting value.

## Repo quirks

- `copilot-instructions.md` is a dangling symlink to a path outside this repo — ignore it; this file is the instruction source.
- `.shopifyignore` is all comments: nothing is excluded from CLI push/dev.
- Default branch: `main`; remote is a fork (`barbaracohenguilhem/tumblr`), not `Shopify/skeleton-theme` despite the README badge.
