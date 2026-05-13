# 🔍 Tender Research Claude Skill

A Claude skill that automates the full tender/procurement research pipeline — from scraping portals to producing a structured Excel report.

---

## What It Does

Given a company profile and a list of portals, this skill:

1. **Scrapes** configured tender portals (aggregators, EU TED, SAM.gov, UN, World Bank, and more)
2. **Filters** results by AI/tech keywords, deadline rules, and supplier eligibility
3. **Scores** each opportunity against your company profile (1–5 fit score)
4. **Downloads** tender documents where possible (for high-scoring opportunities)
5. **Outputs** a formatted Excel spreadsheet with results, flags, and a portal log

---

## Installation

### Step 1 — Clone the skill

```bash
git clone https://github.com/sarkisova-creator/tender-research-claude-skill.git \
  /mnt/skills/user/tender-research
```

### Step 2 — Set up your company profile

```bash
mkdir -p /mnt/skills/user/tender-research/priming
cp /mnt/skills/user/tender-research/priming-template/company.md \
   /mnt/skills/user/tender-research/priming/company.md
```

Then open `/mnt/skills/user/tender-research/priming/company.md` and fill in your company details.

> ⚠️ **The skill will not run without `priming/company.md`.** It needs your profile to score tender fit accurately.
>
> The `priming/` folder is intentionally not committed to the repo — it stays local and contains your private company information.

### Step 3 — Run it

Tell Claude:
```
Search for tenders
```

That's it.

---

## Usage

Once installed, trigger the skill with prompts like:

```
Search for tenders
Find AI procurement opportunities
Research bids for this week
Check tender portals for new opportunities
Find RFPs matching our capabilities
```

Point it to a specific portal:
```
Search for tenders on ted.europa.eu
Check SAM.gov for AI opportunities
```

Or provide your own list of portals:
```
Search these portals for tenders: [paste URLs]
```

---

## Output

The skill saves a spreadsheet to:
```
/mnt/user-data/outputs/tender-research-YYYY-MM-DD.xlsx
```

The spreadsheet contains three sheets:

| Sheet | Contents |
|-------|----------|
| **Opportunities** | All matched tenders with fit scores, flags, links |
| **Skipped** | Filtered-out tenders with rejection reason |
| **Portal Log** | Every portal visited, status, and result counts |

Priority flags:

| Flag | Meaning |
|------|---------|
| ⭐ | High priority — fit score 4.0+ |
| ✓ | Good match — fit score 2.5–3.9 |
| ⚠ | Date or eligibility unclear — verify manually |

---

## Folder Structure

What's in the repo:
```
tender-research-claude-skill/
├── SKILL.md                        ← Main skill instructions (Claude reads this)
├── references/
│   ├── portals.md                  ← Portal list + URL patterns + rendering notes
│   └── filters.md                  ← Keywords, date rules, eligibility criteria
└── priming-template/               ← Template files — copy these to set up your priming folder
    ├── company.md                  ← Fill in with your company profile
    └── exclusions.md               ← Optional: tender types to always reject
```

Your priming folder (local only — created during setup, never committed to the repo):
```
/mnt/skills/user/tender-research/priming/
├── company.md          ← Required — copied from priming-template/ and filled in
├── exclusions.md       ← Optional
└── team/               ← Optional: CV files (.pdf or .md)
```

---

## Configuring Your Company Profile

Edit `priming/company.md` to describe:

- **What your company does** — capabilities, services, technologies
- **Sectors you serve**
- **Team size and key roles**
- **Minimum contract value and preferred duration**
- **Geographic eligibility** — which regions/countries you can bid in
- **What to always exclude**

The more specific you are, the more accurate the fit scoring will be.

---

## Configuring Portals

The default portal list (`references/portals.md`) covers:

- **Aggregators:** BidDetail, DgMarket, Global Tenders, Tendersinfo
- **EU:** TED (Official Journal)
- **US:** SAM.gov, Grants.gov
- **UK:** Find a Tender
- **Multilateral:** UNGM, World Bank, EBRD, ADB, IFC

To add your own portals, create:
```
/mnt/skills/user/tender-research/portals.txt
```
One URL per line. Lines starting with `#` are comments. This file overrides the default list at runtime.

---

## Requirements

- **Claude.ai** with Computer Use / bash tool enabled
- Python 3 with `openpyxl` (the skill installs it automatically if missing)
- Internet access from the Claude container

---

## How Portal Scraping Works

Many tender portals use JavaScript rendering, so direct HTML fetching returns an empty shell. The skill handles this with a three-tier strategy:

| Tier | Method | When Used |
|------|--------|-----------|
| 1 | `web_fetch` direct | First attempt on every portal |
| 2 | `web_search site:domain` | When Tier 1 returns no content |
| 3 | Aggregator portals | When Tiers 1 & 2 both yield < 3 results |

Login-gated portals are noted in the Portal Log and skipped — the skill does not attempt to bypass authentication.

---

## Fit Scoring

Each tender is scored 1–5 across four dimensions:

| Dimension | Description |
|-----------|-------------|
| Technical match | Do requirements align with your capabilities? |
| Team match | Do your team's skills cover the required areas? |
| Scope match | Is the project size/budget appropriate? |
| Risk flags | Any disqualifying conditions? |

**Inclusion threshold: 2.5+** — tenders below this go to the "Skipped" sheet with a reason noted.

---

## License

MIT — free to use, adapt, and share.
