# Tender Portal List & URL Patterns

This file lists the default tender portals to search. The user may override this with their own portal file.

---

## How to Use This File

For each portal below:
1. Use the **Search URL pattern** to construct a pre-filtered URL where possible.
2. If no search URL pattern is available, load the **Base URL** and navigate to the opportunities/tenders listing section.
3. Look for filter controls: keyword search, category filter, date range.

Replace `{KEYWORD}` in URL patterns with each AI/tech keyword from `filters.md`.

---

## User-Provided Portal File

If the user provides a file (e.g., `portals.txt` or `portals.xlsx`), read that file first. It overrides this list. Each URL in the user's file should be treated as a base URL to the opportunities listing page.

For user-provided portals with no known URL pattern, use this heuristic:
- Look for a "Search" or "Find opportunities" input field
- Try appending `?q=artificial+intelligence` or `?keyword=AI` to the URL
- If that fails, load the base page and extract all visible opportunity listings

---

## Portal Rendering Types — Read This First

Portals fall into three categories that determine your access strategy:

| Type | Behaviour | Strategy |
|------|-----------|----------|
| ✅ **Direct fetch** | Returns full HTML with tender listings | `web_fetch` → extract directly |
| ⚠ **JS-rendered** | Returns nav shell only, no tender content | `web_search site:domain` fallback |
| 🔒 **Login-gated** | Returns login wall, no public listings | Note in Portal Log, skip |

Always try `web_fetch` first. If the response contains no tender titles or listing rows, switch to `web_search` immediately — do not retry `web_fetch`.

---

## Tier 1 — Aggregator Portals (fetch directly, no login required)

These portals aggregate tenders from many sources and return real HTML listings. **Start here** — they give the broadest coverage fastest.

| Portal | URL | Notes |
|--------|-----|-------|
| **BidDetail** | `https://www.biddetail.com/global-tenders/artificial-intelligence-tenders` | ✅ Direct fetch. 300–400 active AI tenders. Top 10 visible without login. Best starting point. |
| **DgMarket** | `https://www.dgmarket.com/tenders/searchTenders.do?keyword={KEYWORD}` | ✅ Usually fetchable. World Bank / bilateral donor projects. |
| **Global Tenders** | `https://www.globaltenders.com/tenders-for/{KEYWORD}` | ✅ Usually fetchable. Broad coverage. |
| **Tendersinfo** | `https://www.tendersinfo.com` | ✅ Partial — top listings visible without login. |

---

## Tier 2 — Major Portals (JS-rendered — use web_search fallback)

These portals are the authoritative sources but render with JavaScript. `web_fetch` returns an empty shell. Use `web_search` with `site:` operator.

### TED — EU Official Journal
- **Base URL:** `https://ted.europa.eu`
- **Rendering:** ⚠ JS-rendered — `web_fetch` returns nav shell only
- **Fallback search queries:**
  ```
  site:ted.europa.eu "artificial intelligence" 2026 active
  site:ted.europa.eu "machine learning" 2026 notice
  site:ted.europa.eu "NLP" OR "natural language processing" tender 2026
  site:ted.europa.eu "computer vision" OR "chatbot" 2026
  site:ted.europa.eu "IoT" OR "internet of things" 2026 procurement
  ```
- **Direct notice URLs** (from search results): `https://ted.europa.eu/en/notice/-/detail/{NOTICE-ID}`
- **PDF fallback:** `https://ted.europa.eu/en/notice/{NOTICE-ID}/pdf` — sometimes fetchable directly
- **CPV codes for AI work:** 72000000, 72200000, 72300000, 72610000

### SAM.gov — US Federal Procurement
- **Base URL:** `https://sam.gov/search/?index=opp`
- **Rendering:** ⚠ JS-rendered — `web_fetch` returns auth shell only
- **Fallback search queries:**
  ```
  site:sam.gov "artificial intelligence" opportunity 2026
  site:sam.gov "machine learning" contract solicitation 2026
  site:sam.gov "NLP" OR "natural language" 2026
  site:sam.gov "computer vision" OR "chatbot" 2026
  site:sam.gov "generative AI" OR "LLM" 2026
  ```
- **Direct opportunity URLs** (from search results): `https://sam.gov/opp/{ID}/view`
- **NAICS codes for AI:** 541511, 541512, 541519, 541715
- **Note:** All SAM.gov opportunities are open to US-based suppliers by default

### Find a Tender (UK)
- **Base URL:** `https://www.find-tender.service.gov.uk`
- **Rendering:** ✅ Usually fetchable
- **Search URL:** `https://www.find-tender.service.gov.uk/Search?Keywords={KEYWORD}&Status=live`

### eProcurement Ireland (eTenders)
- **Base URL:** `https://www.etenders.gov.ie`
- **Rendering:** ⚠ Partially JS-rendered
- **Fallback:** `site:etenders.gov.ie "artificial intelligence" 2026`

---

## Tier 3 — Multilateral / Development Banks

| Portal | URL | Rendering | Notes |
|--------|-----|-----------|-------|
| UN Global Marketplace | `https://www.ungm.org/Public/Notice?noticeType=1&keyword={KEYWORD}` | ✅ Usually fetchable | UN system procurement, open internationally |
| World Bank | `https://projects.worldbank.org/en/projects-operations/procurement` | ✅ Fetchable | Browse by sector: Information Technology |
| EBRD | `https://ecepp.ebrd.com/delta/search.html` | ⚠ JS-rendered | Use `site:ecepp.ebrd.com` fallback |
| ADB | `https://www.adb.org/projects` | ✅ Fetchable | Filter by sector: Information Technology |
| IFC / World Bank Group | `https://disclosures.ifc.org` | ✅ Fetchable | Private sector projects |
| Grants.gov (US) | `https://grants.gov/search-grants?keywords={KEYWORD}` | ✅ Fetchable | US federal grants, R&D |

---

## Heuristic for Unknown Portals (User-Provided URLs)

When the user provides a URL not in this list:

1. Try `web_fetch` on the URL
2. Check the response:
   - **Has tender listings** (titles, deadlines, authorities visible): extract directly
   - **Homepage / nav only**: look for links labelled "Opportunities", "Tenders", "Procurement", "Open calls", "Find contracts" — follow them
   - **Login wall**: note "Login required" in Portal Log, skip
   - **Blank / JS shell** (< 500 chars of content, no listings): switch to `web_search site:{domain} artificial intelligence 2026` immediately
3. Follow pagination up to 5 pages / 50 results
4. If both `web_fetch` and `web_search` return nothing useful: log as "Inaccessible — manual review needed"

---

## Adding Your Own Portals

Maintain your portal list at:
```
/mnt/skills/user/tender-research/portals.txt
```

One URL per line. Lines starting with `#` are comments. The skill reads this file at runtime — add or remove portals here without editing the skill.

Example:
```
# Aggregators (fast, no login)
https://www.biddetail.com/global-tenders/artificial-intelligence-tenders

# EU portals
https://ted.europa.eu

# US portals
https://sam.gov

# Client / sector-specific portals
https://procurement.someclient.com/opportunities
```
