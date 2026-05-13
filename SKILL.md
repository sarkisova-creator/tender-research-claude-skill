---
name: tender-research
description: >
  Use this skill whenever the user wants to research, find, analyze, or track tenders, procurement opportunities, RFPs, or public contracts. Triggers include: "search for tenders", "find procurement opportunities", "check tender portals", "research bids", "find RFPs", "look for contracts", or any request to scan tender/procurement websites for relevant opportunities. Also triggers when the user provides tender portal URLs and asks to find relevant AI/tech opportunities. This skill handles the full pipeline: portal scraping → keyword filtering → date validation → company fit analysis → spreadsheet output.
---

# Tender Research Skill

This skill runs a full tender research pipeline: scraping configured portals → filtering by AI/tech keywords and date rules → analyzing fit against your company profile → saving results to a structured Excel spreadsheet.

---

## Before You Start — Read These Files

1. **`references/portals.md`** — The list of tender portal URLs to search. If the user provides a file of portals, read it first and use those URLs.
2. **`references/filters.md`** — Keyword list, date rules, and eligibility criteria.
3. **Priming folder** — `/mnt/skills/user/tender-research/priming/` — Contains your company description (`company.md`) and any CV/team files. Read ALL files in this folder before analyzing tenders.

If the priming folder does not exist or is empty, ask the user to set it up before proceeding. Do not guess at the company profile.

---

## Workflow Overview

```
Step 1: Load configuration
Step 2: Load priming folder (company profile + CVs)
Step 3: For each portal URL → scrape opportunity listings
Step 4: Filter by keywords + date rules
Step 5: For each filtered tender → fetch detail page + analyze fit
Step 6: For matched tenders → attempt document download
Step 7: Write results to Excel spreadsheet
Step 8: Report summary to user
```

Work through portals **one by one**. Do not skip portals.

---

## Step 1: Load Configuration

Read `references/filters.md` for:
- AI/tech keywords
- Date eligibility rules
- Supplier eligibility (US-based suppliers must be eligible)

Read the portal list (from `references/portals.md` OR from the user-provided file at the path they specify).

---

## Step 2: Load Priming Folder

```
/mnt/skills/user/tender-research/priming/
├── company.md          ← Required: company description, capabilities, past projects
├── team/               ← Optional: CV files (PDF or .md)
└── exclusions.md       ← Optional: types of tenders to always reject
```

Read `company.md` fully. If team CVs exist, extract key skills/domains from them. Build a mental profile: **what kinds of tenders is this company well-positioned to win?**

If `exclusions.md` exists, note what to always filter out.

---

## Step 3: Scrape Each Portal

Work through portals one by one using the **three-tier access strategy** below. Many major portals (TED, SAM.gov) are JavaScript-rendered and will return empty or near-empty HTML via `web_fetch` — this is expected. Always try all three tiers before marking a portal as inaccessible.

### Tier 1 — Direct fetch (try first)
1. Use `web_fetch` to load the portal's opportunity listing page.
2. If the portal has keyword search, construct a filtered URL (see `references/portals.md`).
3. If the response contains meaningful tender listings (titles, deadlines, authorities): extract them and move to Step 4.
4. If the response is empty, just a nav shell, or obviously JS-rendered (no tender content visible): immediately move to **Tier 2** — do not log this as a failure yet.

### Tier 2 — web_search fallback (use when Tier 1 returns no content)
Run targeted `web_search` queries against the portal domain. This reliably surfaces results even from JS-heavy portals.

Use these query patterns, substituting each AI/tech keyword from `filters.md`:

```
site:ted.europa.eu "artificial intelligence" 2026 active tender
site:ted.europa.eu "machine learning" 2026 notice
site:sam.gov "artificial intelligence" opportunity 2026
site:sam.gov "machine learning" contract 2026
site:ungm.org "AI" OR "machine learning" tender 2026
```

Run 3–5 queries per portal, covering the primary keywords. Collect all result URLs and snippets. Deduplicate by title. Proceed to Step 4 with the combined results.

### Tier 3 — Aggregator fallback (use when Tier 1 and Tier 2 both yield sparse results)
If a portal returns fewer than 3 results after both tiers, supplement with the aggregator portals listed in `references/portals.md` (BidDetail, DgMarket, etc.). These index thousands of tenders and are directly fetchable without login.

### Detail page fetching
After collecting the listing from any tier, attempt `web_fetch` on each individual tender detail URL. Many detail pages are also JS-rendered. If a detail page returns empty:
- Use the title + authority from the search snippet as the source of truth
- Mark "Detail page JS-rendered — verify manually" in the Documents Status column
- Do not discard the tender solely because its detail page is inaccessible

### Pagination
If results are paginated and fetchable, follow up to 5 pages per portal or 50 results, whichever comes first.

### Login-gated portals
If `web_fetch` returns a login/registration wall and `web_search` yields no results for that domain: note "Login required — manual access needed" in the Portal Log and move on.

---

## Step 4: Filter Opportunities

Apply ALL of the following filters. A tender must pass every filter to proceed to Step 5.

### 4a. Keyword Filter
At least one of these keywords must appear in the title or description (case-insensitive):
- Artificial Intelligence, AI
- Machine Learning, ML
- Computer Vision
- Chatbot
- NLP, Natural Language Processing
- Virtual Assistant
- IoT, Internet of Things
- Large Language Model, LLM
- Automation (only if paired with an AI/tech keyword)

### 4b. Date Filter
Apply ALL three rules (today's date = the actual current date when the skill runs):

| Rule | Requirement |
|------|-------------|
| **Open today** | Tender submission window must include today (published before today, deadline after today) |
| **Min deadline** | Deadline must be at least 30 days from today |
| **Active through year-end** | Contract/project period should extend through end of current year (if stated) |

If the deadline is not clearly stated, flag as "Date unclear — verify manually" and include it anyway with a yellow flag.

### 4c. Eligibility Filter
- US-based suppliers must be explicitly eligible, OR the tender must not restrict by geography, OR it must be an international/open procurement.
- Reject tenders explicitly limited to domestic (non-US) suppliers only.

---

## Step 5: Analyze Fit for Each Filtered Tender

For each tender that passes Step 4, attempt to fetch its full detail page with `web_fetch`. If the detail page is JS-rendered or empty, work from what is available (search snippet title, authority, and any partial data).

Extract as many of the following as possible — mark "Not available" for any field that cannot be retrieved:
- Full title
- Contracting authority / buyer name
- Country / region
- Deadline (exact date)
- Estimated value (if stated)
- Technical requirements summary
- Eligibility conditions
- Direct URL to tender
- Document download link (if visible)

**If the detail page is inaccessible:** Proceed with scoring using available data. Set Documents Status to "Detail page JS-rendered — verify manually" and flag with ⚠ if the deadline is also uncertain.

### Fit Scoring (internal — not shown in spreadsheet)
Rate 1–5 on each dimension:
- **Technical match**: Do the requirements align with the company's capabilities?
- **Team match**: Do the CVs cover the required skills?
- **Scope match**: Is the project size/budget appropriate?
- **Risk flags**: Any disqualifying conditions?

**Overall fit**: Average of above. Threshold for inclusion = 2.5+

Include ALL tenders scoring 2.5+ in the spreadsheet. Flag tenders scoring 4+ as **High Priority**.

---

## Step 6: Attempt Document Download

For tenders with fit score ≥ 3.5:

1. Look for a "Download all", "Download documents", "ZIP", or similar button/link on the tender detail page.
2. If a direct file URL is found, use `web_fetch` to download it (if accessible without login).
3. If documents are behind a login wall, note "Documents require login — download manually" in the spreadsheet.
4. If documents are downloaded, perform a deeper analysis: scope of work, evaluation criteria, required certifications, subcontracting rules, and format of bid.
5. Update the fit score and notes based on document contents.

---

## Step 7: Write Results to Excel

Use `openpyxl` to create a spreadsheet at:
```
/mnt/user-data/outputs/tender-research-YYYY-MM-DD.xlsx
```

### Sheet 1: "Opportunities" (main results)

| Column | Content |
|--------|---------|
| A | Priority flag (⭐ = High Priority ≥4, ✓ = Match, ⚠ = Date unclear) |
| B | Tender Title |
| C | Contracting Authority |
| D | Country / Region |
| E | Deadline |
| F | Estimated Value |
| G | Matched Keywords (comma-separated) |
| H | Relevant Technologies |
| I | Fit Score (1–5) |
| J | Fit Notes (1–2 sentence summary of why it matches or doesn't) |
| K | Documents Status |
| L | Direct Link to Tender |
| M | Portal Source |
| N | Date Found |

### Sheet 2: "Skipped" (filtered-out tenders)
Same structure but with a "Reason Skipped" column instead of Fit Score. Useful for audit trail.

### Sheet 3: "Portal Log"
| Portal URL | Status | Tenders Found | Tenders Matched | Notes |
Log every portal visited, even if no results.

### Formatting Rules
- Header row: bold, dark blue background (`1F4E79`), white text
- High priority rows: light yellow background (`FFFACD`)
- Date unclear rows: light orange background (`FFE4B5`)
- Freeze top row
- Auto-fit column widths
- Hyperlink column L (tender links)
- Use Arial 10pt throughout

---

## Step 8: Report to User

After saving the spreadsheet, report:

```
## Tender Research Complete — [Date]

**Portals searched:** X
**Total opportunities found:** X
**After keyword + date filter:** X
**Matched your company profile:** X (including X high-priority)

**Top 3 opportunities:**
1. [Title] — [Authority] — Deadline [Date] — Fit: X/5
2. ...
3. ...

📁 Spreadsheet saved: tender-research-YYYY-MM-DD.xlsx

**Portals with issues:** [list any that blocked access]
```

Then ask: *"Would you like me to do a deeper analysis on any specific tender, or try to download its documents?"*

---

## Important Notes & Limitations

- **Login-gated portals**: Many procurement portals require registration. If `web_fetch` returns a login page, note it and skip — do not attempt to bypass authentication.
- **Dynamic/JS-heavy portals**: Some portals render with JavaScript and `web_fetch` may return incomplete HTML. If results look empty or truncated, note "Portal may require browser rendering — check manually."
- **Rate limiting**: If a portal blocks requests, wait and retry once. If still blocked, skip and note it.
- **Date parsing**: Dates may appear in many formats (DD/MM/YYYY, YYYY-MM-DD, written out). Parse carefully. When uncertain, flag the entry.
- **Currency**: Note the currency of estimated values — do not convert.
- **Priming folder is required**: Never analyze tenders without reading the company profile first. If missing, halt and ask the user.

---

## File Structure

Repo contents (cloned to `/mnt/skills/user/tender-research/`):
```
tender-research/
├── SKILL.md                        ← This file
├── references/
│   ├── portals.md                  ← Default portal list + URL patterns
│   └── filters.md                  ← Keywords, date rules, eligibility
└── priming-template/               ← Template files only — do not read these at runtime
    ├── company.md                  ← Template for user to copy and fill in
    └── exclusions.md               ← Template for exclusions list
```

Priming folder (user-managed, local only — created by copying from `priming-template/`):
```
/mnt/skills/user/tender-research/priming/
├── company.md          ← REQUIRED — must exist before skill can run
├── exclusions.md       ← optional
└── team/               ← optional CVs
```
