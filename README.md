# Clover Infotech – Proposal Generator

Static, browser-based proposal generator. Fill a form, click **Generate**, get a `.docx` matching the Clover format with constant T&C baked in. Deploys via GitHub Actions to GitHub Pages.

## Repo layout

```
proposal-generator/
├── .github/workflows/
│   └── deploy.yml             # GitHub Actions → Pages deployment
├── lib/
│   ├── docx.umd.js            # docx-js 9.5.1, vendored (no CDN dependency)
│   └── FileSaver.min.js       # FileSaver 2.0.5, vendored
├── index.html                 # The whole app (HTML + CSS + JS)
├── README.md                  # This file
├── .nojekyll                  # Tells Pages to skip Jekyll processing
└── .gitignore
```

Why vendored libs? No CDN means it works on locked-down corporate networks, behind firewalls, and offline. Trade-off: the repo is ~800 KB instead of ~10 KB. Worth it.

---

## Deploy: step-by-step

### 1. Create the GitHub repo

```bash
# Create new empty repo on GitHub (any name, e.g. clover-proposal-gen)
# Then locally:
unzip proposal-generator.zip
cd proposal-generator

git init
git add .
git commit -m "Initial commit: proposal generator"
git branch -M main
git remote add origin https://github.com/<your-username>/clover-proposal-gen.git
git push -u origin main
```

### 2. Enable Pages with "GitHub Actions" as source

This is the **critical step** people miss. Without it, the workflow runs but nothing is published.

1. Go to your repo on github.com
2. **Settings** → **Pages** (left sidebar)
3. Under **Build and deployment** → **Source**, pick **GitHub Actions** (not "Deploy from a branch")
4. Save. The dropdown closes — that's it.

> Why not "Deploy from a branch"? That mode just serves files straight from the branch — no logs, no environments, no concurrency control. The Actions mode gives you proper deployment runs with logs, the green/red status on PRs, and lets you add validation/preview steps later if you want.

### 3. Push triggers the deploy

The workflow runs on every push to `main`. Watch it under the **Actions** tab. After ~30–60 seconds you'll see:

- A green check ✅ on the workflow run
- A "Deployment" link in the run summary, and at **Settings → Pages** → "Your site is live at ..."

URL pattern:
- User/Org site (repo named `<user>.github.io`): `https://<user>.github.io/`
- Project site (any other repo name): `https://<user>.github.io/<repo-name>/`

### 4. Manual re-deploy

Go to **Actions** → pick the workflow on the left → **Run workflow** button (top right). Useful if Pages cache is stale or you want to verify a deploy without pushing a code change.

---

## What the workflow does

```yaml
# .github/workflows/deploy.yml
on:
  push: { branches: [main] }     # auto on push to main
  workflow_dispatch: {}          # manual trigger

permissions:
  contents: read                 # checkout the repo
  pages: write                   # publish to Pages
  id-token: write                # OIDC auth for deploy-pages

concurrency:
  group: pages
  cancel-in-progress: false      # don't kill an in-flight prod deploy
```

Five steps:

1. **Checkout** — pull repo into the runner.
2. **Sanity check** — fail fast if `index.html`, `lib/docx.umd.js`, or `lib/FileSaver.min.js` is missing.
3. **`actions/configure-pages@v5`** — tells Pages this repo will be publishing.
4. **`actions/upload-pages-artifact@v3`** — tarballs the repo and uploads it as the deployment artifact.
5. **`actions/deploy-pages@v4`** — Pages picks up the artifact and serves it.

---

## Common deployment failures (and fixes)

| Symptom in Actions log | Cause | Fix |
|---|---|---|
| `Error: Get Pages site failed` / 404 | Pages not enabled, OR enabled with "Deploy from branch" instead of "GitHub Actions" | Settings → Pages → Source = **GitHub Actions** |
| `Resource not accessible by integration` | Workflow lacks the `pages: write` or `id-token: write` permission | Already set in `deploy.yml`. If you forked and changed it, restore the `permissions:` block |
| Run gets queued forever | A previous deploy is still running, concurrency is gating it | Wait for it. The `cancel-in-progress: false` is intentional — don't kill a live deploy |
| Site is live but shows old content | Pages CDN cache | Hard refresh (Ctrl+F5). Edge cache clears in ~30 seconds |
| Site is live but `lib/docx.umd.js` 404s | The lib/ folder wasn't committed | `git ls-files lib/` to confirm; if empty, `git add lib/ && git commit && git push` |
| Action fails on "Sanity check" step | One of the required files is missing | Check `git ls-files` includes `index.html`, `lib/docx.umd.js`, `lib/FileSaver.min.js`, `.nojekyll` |

---

## Usage workflow (after deployment)

1. Open the deployed URL.
2. Click **Load KBL Example** to see what a filled form looks like.
3. Edit fields for the new client (project title, client name, creators, inventory tables, milestones, etc.).
4. Click **💾 Save Draft (browser)** at any time to persist locally to `localStorage`.
5. Click **⬇ Generate & Download .docx** — downloads a file named like `KBL_Alfresco_Upgrade_Proposal.docx`.
6. To reuse data on another machine, click **Export JSON**, transfer the file, then **Import JSON** there.

---

## Customizing the constants

Constants live near the top of `index.html` in a JS object called `CONSTANT_TC`:

| What | Where in `index.html` |
|---|---|
| Payment terms (validity, taxes, lead time, leave, YoY %, etc.) | `CONSTANT_TC.paymentTerms` |
| GST / SAC / PO address table | `CONSTANT_TC.gstInfo` |
| General T&C (Confidentiality, Non-Solicit, Liability, IP, Termination, OHS) | `CONSTANT_TC.generalTC` |
| IP / Confidentiality intro paragraphs | `CONSTANT_TC.ipStatement` |
| Clover address block on cover/IP page | `CONSTANT_TC.cloverAddress` |
| Brand green colour | `const GREEN = "4B9B3B"` (in the docx generation script) |

Edit, commit, push — Actions redeploys automatically.

---

## Form sections

| Section | Notes |
|---|---|
| Cover Page | Title, client, short name (used in footer), date, multiple creators |
| Executive Summary | Free text. Blank lines = paragraph breaks |
| Background | Requirement bg + scope intro + two dynamic inventory tables |
| Technical Proposal | Solution intro + bullet list (one per line). `**bold**` markdown supported |
| SOW | Five fixed fields rendered as a 2-column table |
| Deliverables | One bullet per line, supports `**bold**` for sub-headings |
| Team Comp / Skills | One item per line → bullet list |
| Escalation Matrix | Dynamic rows (Level / Name / Phone / Email) |
| Approach Methodology | Dynamic rows (Sr. / Activity / Duration / Description) |
| Schedule | Gantt-style. Format: `Phase Name \| week1,week2`. Set total weeks separately |
| Comm/Governance | Dynamic 3-column tables |
| Pre-reqs / Assumptions / Exclusions | One bullet per line |
| Commercials | Engagement model + total + dynamic milestone rows. Amounts auto-calculated from % |

---

## Common footguns

- **Forgetting to save draft before refresh.** Browser refresh loses unsaved fields. Use **Save Draft** or **Export JSON** before closing the tab.
- **Milestone percentages not summing to 100.** The generator does NOT validate — you'll see whatever you entered in the bottom row of the milestone table.
- **Brand colour value.** `const GREEN = "4B9B3B"` — no `#` prefix, docx-js wants raw hex.
- **Replacing vendored libs with CDN to save space.** Only do this if you're 100% sure the network where users browse the site can reach the CDN. The vendored copy is the safer default.

---

## Future enhancements (parking lot)

- Embed Clover logo as a base64 image in `ImageRun` on the cover page.
- Add a Markdown live-preview pane next to each long-form textarea.
- Add a "duplicate previous proposal" dropdown stocked from a folder of JSON files in the repo.
- PDF export step (call a converter in the Actions workflow or use a serverless function).
- Add a PR preview deployment using a separate workflow + `actions/deploy-pages` with a different environment name.
