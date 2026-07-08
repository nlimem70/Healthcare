---
description: Security-scan, commit & push, deploy GitHub Pages via Actions, refresh the README, and update the repo About/homepage link.
argument-hint: [optional commit message]
---

Run the full publish pipeline for this repo, in this exact order. Do not skip the security scan and do not push if it finds anything. This repo is a single self-contained static site (see [CLAUDE.md](../../CLAUDE.md)) deployed to GitHub Pages at `https://nlimem70.github.io/Healthcare/` from `https://github.com/nlimem70/Healthcare.git`. Git lives at `C:\Program Files\Git\cmd\git.exe` — if a PowerShell call reports `git` as unrecognized, prepend `$env:PATH = "C:\Program Files\Git\cmd;$env:PATH"` to that same command block (PATH doesn't persist across tool calls).

## 1. Security scan (do this first, before touching git)

Scan every tracked-or-about-to-be-tracked file in the repo (not just the diff) for anything that shouldn't be public:
- Secrets/credentials: API keys, tokens, passwords, connection strings, private key blocks (`-----BEGIN ... PRIVATE KEY-----`), `.env` files, AWS/GCP/Azure key patterns.
- Real patient/personal data: this is a healthcare-clinic site — check the enquiry form JS and any sample data for real names, emails, phone numbers, or addresses instead of placeholder/fictional ones.
- Anything matching entries in `.gitignore` that may have been accidentally force-added.

Use Grep across the repo for common patterns (`api[_-]?key`, `secret`, `password`, `BEGIN PRIVATE KEY`, `AKIA[0-9A-Z]{16}`, etc.) and manually eyeball any new/changed files with Read. Confirm `.claude/settings.local.json` is still gitignored and not staged (`git status`).

If you find anything sensitive: **stop immediately, do not commit/push, and report exactly what you found and where.** Only continue to step 2 if the scan is clean.

## 2. Commit and push

- `git status` to see what changed.
- `git add -A` (respecting `.gitignore`).
- Commit with a descriptive message — use `$ARGUMENTS` as the message if provided, otherwise write one summarizing the actual changes.
- Push to `origin main`. This is a real push to a public repo — if this command is being run interactively (not as a pre-authorized batch job), confirm with the user before pushing, per standard practice for shared-state changes. Skip the confirmation only if the user invoked this command specifically asking you to run it end-to-end unattended.

## 3. GitHub Pages via Actions

- Check `.github/workflows/deploy-pages.yml` exists and is correct (checkout → copy site files into `_site` → `actions/configure-pages` → `actions/upload-pages-artifact` → `actions/deploy-pages`, with `pages: write` / `id-token: write` permissions). Create or fix it if missing/broken.
- After pushing, verify the workflow run succeeded (WebFetch `https://github.com/nlimem70/Healthcare/actions` or use `gh run list` if `gh` is available).
- Verify Pages is actually enabled: WebFetch `https://api.github.com/repos/nlimem70/Healthcare/pages` — a 404 means the repo's Settings → Pages → Source is not yet set to "GitHub Actions". If `gh` CLI is authenticated, you can set this directly via `gh api -X POST repos/nlimem70/Healthcare/pages -f build_type=workflow`; otherwise tell the user to flip that one setting manually (there's no way to do it without either `gh` auth or a token).

## 4. Professional README

Write/update `README.md` at repo root to actually look professional, not a placeholder. Include: a one-line project description, the live site link (`https://nlimem70.github.io/Healthcare/`), what the project is (single-file static marketing/appointment site for a fictional clinic, per CLAUDE.md), how to run it locally (just open `index.html`), and a note that there's no build step/dependencies. Keep it accurate to what's actually in `index.html` — don't invent features that don't exist. Don't add license/contributing boilerplate unless asked.

## 5. Repo About + homepage link

Update the GitHub repo's description ("About") and homepage URL to point at the live Pages site. This needs `gh` CLI authenticated:
- Check with `Get-Command gh -ErrorAction SilentlyContinue`. If missing, install via `winget install --id GitHub.cli -e --source winget --accept-package-agreements --accept-source-agreements`, same pattern used to install git in this repo (refresh PATH in the same command block afterward, e.g. `$env:PATH = "C:\Program Files\GitHub CLI;$env:PATH"`).
- If `gh auth status` shows not logged in, tell the user you need them to run `gh auth login` interactively (this can't be done non-interactively) rather than trying to force it.
- Once authenticated: `gh repo edit nlimem70/Healthcare --description "<short description>" --homepage "https://nlimem70.github.io/Healthcare/"`.
- If `gh` auth isn't available and can't be set up in this session, tell the user exactly what to paste into Settings → About (description + homepage field) instead of skipping silently.

## 6. Report back

Summarize what actually happened at each numbered step (including anything skipped and why), not just "done" — especially call out if the security scan found something, if Pages needed a manual settings change, or if `gh` auth was unavailable.
