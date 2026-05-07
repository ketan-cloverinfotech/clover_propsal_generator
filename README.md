# Clover Infotech – Proposal Generator

Static, browser-based proposal generator. Fill a form, click **Generate**, get a `.docx` matching the Clover format with constant T&C baked in.

## What it does

- Single-page app, runs entirely in the browser (no backend, no server).
- Generates `.docx` client-side using [`docx-js`](https://github.com/dolanmiu/docx) loaded from CDN.
- Cover page, IP/Confidentiality, Executive Summary, Background (with current + target inventory tables), Technical Proposal, SOW, Team/Skills/Escalation, Approach Methodology, Schedule (Gantt), Pre-reqs/Assumptions/Exclusions, Commercials with auto-calculated milestone amounts, payment T&C, GST info, and General T&C.
- All Clover legal text (Payment Terms, GST, General T&C) is hardcoded in `index.html` — edit there if policy changes.
- Save draft to browser (localStorage), Export/Import as JSON for reuse across machines.
- "Load KBL Example" button pre-fills form with the original KBL proposal data so you can see the format end-to-end.

## File layout

```
proposal-generator/
├── index.html      # The whole app (HTML + CSS + JS in one file)
├── .nojekyll       # Tells GitHub Pages to skip Jekyll processing
└── README.md
```

## Deploy to GitHub Pages

### Option A — User/Org site (root repo)

```bash
# Create repo named <username>.github.io (replace <username>)
git init
git add .
git commit -m "Initial commit: proposal generator"
git branch -M main
git remote add origin https://github.com/<username>/<username>.github.io.git
git push -u origin main
```

Live at `https://<username>.github.io/` within a minute.

### Option B — Project site (any repo name)

```bash
# Create any repo, e.g. clover-proposal-gen
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<username>/clover-proposal-gen.git
git push -u origin main
```

Then on GitHub:
1. Go to **Settings → Pages**.
2. Under **Source**, pick **Deploy from a branch**.
3. Branch: `main`, folder: `/ (root)`. Save.
4. Wait ~30–60 seconds, refresh. URL shown at the top: `https://<username>.github.io/clover-proposal-gen/`.

## Usage workflow

1. Open the deployed URL.
2. Click **Load KBL Example** to see what a filled form looks like.
3. Edit fields for the new client (project title, client name, creators, inventory tables, milestones, etc.).
4. Click **💾 Save Draft (browser)** at any time to persist locally.
5. Click **⬇ Generate & Download .docx** — downloads a `.docx` with auto-named file (e.g. `KBL_Alfresco_Upgrade_Proposal.docx`).
6. To reuse data on another machine, click **Export JSON**, transfer the file, then **Import JSON** there.

## Customizing the constants

The constants live near the top of `index.html` in a JS object called `CONSTANT_TC`. Edit:

| What | Where |
|---|---|
| Payment terms (validity, taxes, lead time, leave, YoY %, etc.) | `CONSTANT_TC.paymentTerms` |
| GST / SAC / PO address table | `CONSTANT_TC.gstInfo` |
| General T&C (Confidentiality, Non-Solicit, Liability, IP, Termination, OHS) | `CONSTANT_TC.generalTC` |
| IP / Confidentiality intro paragraphs | `CONSTANT_TC.ipStatement` |
| Clover address block on cover/IP page | `CONSTANT_TC.cloverAddress` |
| Brand green colour | `const GREEN = "4B9B3B"` (further down, in the docx generation script block) |

After editing, commit + push — GitHub Pages redeploys automatically.

## Form sections explained

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
| Schedule | Gantt-style. Format: `Phase Name | week1,week2`. Set total weeks separately |
| Comm/Governance | Dynamic 3-column tables |
| Pre-reqs / Assumptions / Exclusions | One bullet per line |
| Commercials | Engagement model + total + dynamic milestone rows. Amounts auto-calculated from % |

## Common mistakes / footguns

- **Forgetting to save draft before refresh.** Browser refresh loses unsaved work. Use **Save Draft** or **Export JSON** before closing the tab.
- **Milestone percentages not summing to 100.** The generator does not block this — it just shows whatever you entered. Check the bottom row of the milestone table in the generated `.docx`.
- **Special characters in project title / client name.** They're sanitized for the filename only; document content keeps them as-is.
- **CDN blocked at client site.** This app loads `docx-js` and `FileSaver` from `cdnjs.cloudflare.com`. If you're behind a corporate proxy that blocks it, host both libs locally next to `index.html` and update the `<script src=...>` paths.
- **Old browser support.** Tested on Chrome/Edge/Firefox 2023+. Old IE will not work — uses `async/await` and modern Blob APIs.

## Troubleshooting

| Problem | Fix |
|---|---|
| Blank `.docx` opens with error | Open browser DevTools → Console, check for JS errors. Most common: a typo in JSON import or missing CDN |
| Tables overflow page | Reduce text in long cells; column widths in the generator are tuned for US Letter with 1″ margins |
| Bullets not rendering | Make sure each item is on its own line; don't paste rich-text bullets — the `•` is added by Word's numbering |
| Brand colour not green | Confirm `const GREEN = "4B9B3B"` (no `#` prefix — docx-js wants raw hex) |

## Why this stack

- **No backend** = no auth, no hosting cost, no maintenance. GitHub Pages is free and serves it forever.
- **`docx-js` in browser** = the same proven library used in the Clover team's existing tooling, just loaded via CDN. The generated file opens cleanly in MS Word, LibreOffice, and Google Docs.
- **localStorage drafts + JSON export** = lets the same engineer or different engineers reuse data across browsers/machines without a database.

## Future enhancements (parking lot)

- Embed Clover logo as a base64 image in `ImageRun` on the cover page.
- Add a Markdown live-preview pane next to each long-form textarea.
- Add a "duplicate previous proposal" dropdown stocked from a folder of JSON files in the repo.
- PDF export (call `docx → pdf` server-side or use a converter API).
