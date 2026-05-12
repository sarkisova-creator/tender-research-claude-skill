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

## Folder Structure

```
tender-research-claude-skill/
├── SKILL.md                        ← Main skill instructions (Claude reads this)
├── references/
│   ├── portals.md                  ← Portal list + URL patterns + rendering notes
│   └── filters.md                  ← Keywords, date rules, eligibility criteria
└── priming-template/
    ├── company.md                  ← Fill this in with your company profile
    └── exclusions.md               ← Optional: tender types to always reject
```

---

## Installation

### 1. Copy the skill into your Claude skills directory

Place the folder at:
```
/mnt/skills/user/tender-research/
```

So the full path to the skill file is:
```
/mnt/skills/user/tender-research/SKILL.md
```

### 2. Set up your priming folder

Create a folder at:
```
/mnt/skills/user/tender-research/priming/
```

Copy `priming-template/company.md` into it and fill in your details:
```
/mnt/skills/user/tender-research/priming/company.md       ← Required
/mnt/skills/user/tender-research/priming/exclusions.md    ← Optional
/mnt/skills/user/tender-research/priming/team/            ← Optional: CV files (.pdf or .md)
```

> ⚠️ **The skill will not run without `company.md`.** It needs your profile to score tender fit.

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

You can also point it to a specific portal:
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

### Priority flags in the spreadsheet:

| Flag | Meaning |
|------|---------|
| ⭐ | High priority — fit score 4.0+ |
| ✓ | Good match — fit score 2.5–3.9 |
| ⚠ | Date or eligibility unclear — verify manually |

---

## Configuring Your Company Profile

Edit `priming/company.md` to describe:

- **What your company does** (capabilities, services, technologies)
- **Sectors you serve**
- **Team size and key roles**
- **Minimum contract value and preferred duration**
- **Geographic eligibility** (which regions/countries you can bid in)
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

You can add your own portals by creating a file at:
```
/mnt/skills/user/tender-research/portals.txt
```
One URL per line. Lines starting with `#` are comments. This file overrides the default list at runtime.

---

## Requirements

- **Claude.ai** with Computer Use / bash tool enabled
- Python 3 with `openpyxl` installed (the skill installs it automatically if missing)
- Internet access from the Claude container

---

## How Portal Scraping Works

Many tender portals use JavaScript rendering, which means direct HTML fetching returns an empty shell. The skill handles this with a three-tier access strategy:

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

**Inclusion threshold: 2.5+**

Tenders scoring below 2.5 are moved to the "Skipped" sheet with a reason noted.

---

## License

MIT — free to use, adapt, and share.
