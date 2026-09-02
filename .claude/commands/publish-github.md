---
description: Publish this project to GitHub — secret scan, push, README, About section, and Pages via Actions
argument-hint: <owner/repo> [public|private]
---

# Publish to GitHub

Publish this project to a GitHub repository, end to end.

Repo details from the user: **$ARGUMENTS**

If `$ARGUMENTS` is empty or does not contain an `owner/repo` slug, **stop and ask** for:
the target `owner/repo`, and whether it should be public or private. Do not guess a repo
name from the folder name — this folder lives under a personal OneDrive path and its name
is not a safe default.

Default visibility if the user gives a slug but no visibility: **ask**. Never default to
public on your own.

Work through the steps below **in order**. Step 1 gates everything else: nothing is pushed
until the scan is clean or the user has explicitly accepted each finding. Report at the end
with the repo URL, the Pages URL, and anything you skipped.

---

## Step 0 — Preflight

Both tools are required and **neither was installed on this machine** when this command was
written. Check first, and stop with install instructions rather than half-completing:

```powershell
git --version
gh --version
gh auth status
```

- No `git` → tell the user to install it (`winget install Git.Git`) and re-run. Stop.
- No `gh` → tell the user to install it (`winget install GitHub.cli`) and re-run. Stop.
- `gh auth status` not logged in → tell them to run `gh auth login` themselves in their own
  terminal (it is interactive; it cannot be driven from here). Stop.

Remember: this shell is PowerShell 5.1. No `&&`, no `||`. Chain with `;` or
`cmd; if ($?) { next }`.

---

## Step 1 — Secret / PII scan (BEFORE anything is pushed)

Scan every file that would be uploaded. Read the actual matches — do not just count them.

```powershell
$files = Get-ChildItem -Recurse -File -Force |
  Where-Object { $_.FullName -notmatch '\\\.git\\' }
$files | Select-Object FullName, Length
```

Then grep the working tree for each of these classes and inspect every hit:

**Credentials and keys**
- `password`, `passwd`, `pwd\s*=`, `secret`, `api[_-]?key`, `apikey`, `token`, `bearer`
- `AKIA[0-9A-Z]{16}` (AWS), `ghp_`, `gho_`, `ghs_`, `github_pat_` (GitHub tokens)
- `sk-[A-Za-z0-9]`, `sk-ant-` (LLM API keys), `AIza[0-9A-Za-z_-]{35}` (Google)
- `-----BEGIN .*PRIVATE KEY-----`, `.pem` / `.pfx` / `.p12` / `.key` files
- Connection strings: `mongodb://`, `postgres://`, `mysql://`, `Server=.*Password=`
- `.env`, `.env.*`, `credentials`, `secrets.*`, `*.tfstate` files present at all

**PII and internal identity — this project is especially exposed to these**
- Email addresses: `[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}`
- The user's own account name, and the employer name embedded in this folder's path.
  Derive both at runtime rather than reading them from here, then grep for each:

  ```powershell
  $env:USERNAME
  ($PWD.Path -split '\\') | Where-Object { $_ -match ' - ' }   # "OneDrive - <Company>"
  ```

  Search case-insensitively for the username and for each word of the company name.
- **Absolute Windows paths**, which leak both of the above at once:
  `C:\\Users\\`, `OneDrive - `, `%USERPROFILE%` written out literally
- Phone numbers, national IDs, addresses, real customer or supplier names in seed data
- Internal hostnames / private IPs: `10\.`, `192\.168\.`, `\.local`, `\.internal`, VPN or
  jump-host names

**Project-specific note:** `index.html` ships seed inventory records with supplier and
product names. Confirm with the user that this seed data is invented, not exported from a
real system, before it goes to a public repo. `CLAUDE.md` is also uploaded — read it in
full and check it names no internal systems or people.

Handling findings:
- Real secret (a live credential) → **stop**. Tell the user to rotate it first. Do not
  push, and do not offer to push it "just privately".
- PII / internal identifiers → list each one with its file and line, and ask the user
  per-item: remove, redact, or accept. Apply their choice before continuing.
- False positive (e.g. the word "token" in a comment) → say why you judged it benign.

If the repo already exists and has history, also scan what is already committed:

```powershell
git log --oneline -20
```

Nothing already public can be un-published by deleting it later — say so plainly if you
find something there, and recommend rotation over deletion.

Only continue when the scan is clean or every finding has been explicitly accepted by the
user.

---

## Step 2 — Push the code

Create a `.gitignore` first if one does not exist. At minimum:

```
.env
.env.*
*.pem
*.key
*.pfx
*.p12
.DS_Store
Thumbs.db
desktop.ini
~$*
.claude/settings.local.json
```

Keep `.claude/commands/` tracked — those are shareable. Do **not** ignore `CLAUDE.md`; it
is intended documentation.

This folder is **not currently a git repository**. Initialise and push:

```powershell
git init -b main
git add -A
git status            # review exactly what is staged before committing
git commit -m @'
Initial commit: LogiTrack Inventory dashboard

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>
'@
```

Show the user `git status` output and confirm the file list before the first commit — this
is the last checkpoint before anything leaves the machine.

Then create or attach the remote:

```powershell
# New repo (substitute the visibility the user chose):
gh repo create <owner/repo> --private --source . --remote origin --push

# Or, if the repo already exists:
git remote add origin https://github.com/<owner/repo>.git
git push -u origin main
```

If the remote already has commits, do not force-push. Pull, reconcile, and tell the user
what diverged.

Note: this folder is inside OneDrive. If OneDrive file locking causes a git error, say so
rather than retrying blindly.

---

## Step 3 — README

Create `README.md` if absent, or update it if present (preserve anything the user wrote —
edit around it, don't overwrite).

Write it from what the code actually does, by reading `index.html` and `CLAUDE.md`. Cover:

- **What it is** — one paragraph. An internal warehouse stock dashboard, single-file, no
  build step.
- **Live demo** — link the Pages URL from Step 5 (add this after Pages is deployed).
- **Running it locally** — open `index.html` in a browser. That is the whole setup.
- **Features** — search, category filter, typed column sorting, derived In/Low/Out status,
  add and delete records, accessible form validation.
- **Design constraints** — one file; no frameworks, build step, or external requests; state
  is in-memory only so a reload restores seed data.
- **Data note** — state plainly that the inventory rows are sample data, not real stock.

Keep it honest: do not claim tests, CI, or a license that does not exist. Ask whether the
user wants a license file rather than adding one unprompted.

Commit and push the README.

---

## Step 4 — About section

Set the repo description, homepage and topics:

```powershell
gh repo edit <owner/repo> `
  --description "<one line, under 100 chars>" `
  --homepage "https://<owner>.github.io/<repo>/" `
  --add-topic inventory `
  --add-topic dashboard `
  --add-topic vanilla-javascript `
  --add-topic single-file `
  --add-topic accessibility
```

Draft the description yourself, then show it to the user for approval before applying —
the About blurb is the first thing anyone reads. Keep the description free of employer or
personal identifiers (see Step 1).

---

## Step 5 — GitHub Pages via GitHub Actions

Create `.github/workflows/pages.yml`. This project is plain static files at the repo root,
so the workflow uploads the root directly — no build:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      - id: deployment
        uses: actions/deploy-pages@v4
```

`index.html` at the root becomes the served page. Pin the action versions as written; check
they are still current if a workflow run fails on a deprecated action.

Set the Pages source to Actions (a repo with Pages already configured needs `PUT`):

```powershell
gh api --method POST /repos/<owner>/<repo>/pages -f build_type=workflow
# already configured -> 409; then:
gh api --method PUT /repos/<owner>/<repo>/pages -f build_type=workflow
```

Commit and push the workflow, then watch the run:

```powershell
gh run list --limit 3
gh run watch
```

**Publishing a private repo's Pages site makes it publicly readable** on GitHub Free. If
the user chose private in Step 1, confirm explicitly that they still want Pages before
enabling it — the privacy of the repo does not carry over to the site.

Once deployed, add the live URL to the README (Step 3) and verify the About homepage field
(Step 4) matches.

---

## Final report

Tell the user:

1. Scan result — what was found, what was removed, what they accepted.
2. Repo URL and visibility.
3. Pages URL, and whether the first deployment succeeded (`gh run list`).
4. Anything skipped or blocked, and why.

Do not report success for a step you did not actually verify.
