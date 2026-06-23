# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Calibre News Delivery** — uses GitHub Actions to run Calibre on a schedule,
convert news/magazine *recipes* into EPUBs, and email them to reading devices
(Kindle, iPad/Apple Books). Forked from a template and customized; see
`README.md` / `README.zh-CN.md`.

## Architecture

Delivery is split by cadence into two schedulers that both call one shared engine:

| File | Role |
|---|---|
| `.github/workflows/calibre-news.yml` | **Reusable engine** (`workflow_call` only — no schedule, no `workflow_dispatch`). Installs Calibre, converts the recipes in the given list, emails them. Never runs on its own. |
| `.github/workflows/daily.yml` | Cron `0 0 * * *`. Calls the engine with `recipe_list: recipe_daily.txt`, `audience: news`. |
| `.github/workflows/weekly.yml` | Cron `0 0 * * 1` (Mon). Calls the engine with `recipe_list: recipe_weekly.txt`, `audience: magazine`. |

### Recipe lists
- `recipe_daily.txt` → `guardian-custom.recipe`
- `recipe_weekly.txt` → `New York Review of Books (no subscription)`, `The Diplomat`,
  `Nautilus Magazine`, `economist-custom.recipe`

**Recipe discovery model** (in `calibre-news.yml`, "Checking Recipe" step): the engine
processes **only** the entries listed in the recipe-list file. Each line is either:
- a **local `.recipe` file** in the repo root (referenced by filename), or
- a **Calibre built-in recipe title** (resolved via `ebook-convert --list-recipes`).

It does **not** auto-scan the repo for `*.recipe` files (an earlier version did — that
broke the daily/weekly split, so it was removed). Add a recipe by listing it.

### Custom recipes (RSS-based, in repo root)
- `guardian-custom.recipe` — The Guardian, **today only** (`oldest_article = 1`),
  12 curated feeds, live-blog filtering. Routed to email (see E999 note).
- `economist-custom.recipe` — best-effort **free** Economist digest; most full text is
  paywalled, so expect headlines + summaries outside free sections (the world this week,
  leaders). Weekly.

## Delivery routing (env secrets, in the `calibre-news` GitHub Environment)

`audience` selects the recipient in `calibre-news.yml`:
`to: ${{ inputs.audience == 'news' && secrets.TO_NEWS || secrets.TO }}`

- **Daily / news → `TO_NEWS`** (a normal inbox, read on iPad in Apple Books).
- **Weekly / magazines → `TO`** (Kindle `@kindle.com`).

Other secrets: `FROM`, `SMTP`, `PORT`, `ENCRYPT`, `SECRET` (SMTP password),
optional `FORMAT` (default epub), `SIZE` (default 25MB), `DAYS` (artifact retention).

## Key decisions & gotchas (the "why")

- **Why Guardian goes to email, not Kindle:** Amazon's "Send to Kindle" server-side
  conversion fails with **E999** on large/image-heavy EPUBs. Newspapers read fine as
  plain EPUB on iPad/Apple Books, which skips Amazon conversion entirely. Magazines
  (smaller/less frequent) still go to Kindle.
- **Why NYT was dropped:** NYT's paywall is server-side and blocks datacenter IPs, so
  GitHub Actions gets paywall stubs — free scraping yields mostly empty articles.
- **Format stays EPUB:** Send to Kindle no longer accepts MOBI/AZW3 (since 2022). Do
  not switch formats to "fix" delivery.
- **DST:** crons are fixed UTC. `0 0 * * *` etc. shift by an hour between CET/CEST for
  the Sweden-based owner.

## Conventions

- Commit messages end with the `Co-Authored-By: Claude` trailer.
- Only commit/push when the user asks.
- Test a recipe's Python syntax locally with `python -m py_compile <file>.recipe`
  (full conversion needs Calibre installed).
