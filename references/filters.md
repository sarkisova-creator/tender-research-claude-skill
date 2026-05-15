# Tender Filters Reference

These are the mandatory filters applied in Step 4 of the tender research pipeline.

---

## Keyword Filter

A tender must contain at least ONE of the following keywords in its **title or description** (case-insensitive match):

### Primary Keywords (high confidence)
- `Artificial Intelligence`
- `AI` (must appear as a standalone term or in a tech context — avoid false matches like "AI" as a country code)
- `Machine Learning`
- `ML`
- `Computer Vision`
- `Chatbot`
- `NLP`
- `Natural Language Processing`
- `Virtual Assistant`
- `IoT`
- `Internet of Things`
- `Large Language Model`
- `LLM`
- `Generative AI`
- `GenAI`
- `Deep Learning`

### Secondary Keywords (include only if paired with a primary keyword or clear tech context)
- `Automation` — only if alongside AI/ML context
- `Smart systems` — only if alongside AI/ML context
- `Predictive analytics`
- `Data science`
- `Neural network`
- `Cognitive computing`
- `Intelligent systems`

### Exclusion Keywords (reject even if primary keyword present)
These terms indicate the tender is NOT relevant even if it mentions AI:
- `AI` as part of country/language code in isolation (e.g., "Artificial Insemination" in agriculture tenders)
- Pure hardware procurement with no software/AI component
- Tenders exclusively about physical infrastructure unless explicitly mentioning AI integration

---

## Date Filter Rules

All three rules must be satisfied. Use the **actual current date** when the skill runs (check with system date).

### Rule 1: Open Today
- The tender's **submission window must be open** on the search date.
- Meaning: Publication/open date ≤ today AND deadline > today.
- If only a deadline is shown (no open date), assume it is open if deadline > today.

### Rule 2: Minimum Deadline (30-day rule)
- Deadline must be **at least 30 days from today**.
- Example: if searching on May 9, 2026 → deadline must be June 8, 2026 or later.

### Rule 3: Active Through Year-End
- The **contract performance period** (if stated) should extend through the end of the current calendar year.
- If only a submission deadline is shown (no contract period), this rule is waived — include the tender and note "Contract period not stated."
- If the contract explicitly ends before the end of the current year, skip it.

### Date Uncertainty Handling
If date information is ambiguous or unclear:
- Include the tender in the spreadsheet
- Flag it with ⚠ in column A
- Set Deadline column to "Unclear — verify manually"
- Do not discard on the basis of date uncertainty alone

---

## Supplier Eligibility Filter

### Include if:
- The tender explicitly states "open to international bidders"
- The tender explicitly states "US-based companies eligible" or similar
- No geographic restriction is mentioned (open procurement)
- The tender is from an international body (UN, World Bank, EU institutions — these are generally open to US suppliers)
- The tender is posted on SAM.gov or any US federal portal

### Exclude if:
- The tender explicitly states it is restricted to domestic (non-US) companies only
- The tender requires a local business registration in a country that excludes US entities
- The tender specifies EU/EEA-only suppliers without a waiver provision

### Flag if:
- Eligibility is unclear → include with note "Eligibility unclear — verify before bidding"

---

## Fit Score Thresholds

| Score | Meaning | Spreadsheet Action |
|-------|---------|-------------------|
| 4.0–5.0 | High priority match | Include + ⭐ flag |
| 2.5–3.9 | Good match | Include + ✓ flag |
| 1.0–2.4 | Weak match | Include in "Skipped" sheet with reason |
| N/A | Failed keyword/date/eligibility | Include in "Skipped" sheet with reason |

---

## Notes on AI Keyword Matching in Practice

Different portals use different terminology. Watch for:
- European portals: often use "artificial intelligence", "machine learning" in English even for non-English tenders
- CPV codes that indicate AI/tech work (EU TED): 72000000 (IT services), 72200000 (software), 72300000 (data services)
- US SAM.gov NAICS codes: 541511, 541512, 541519, 541715
- Abbreviations that vary: "A.I.", "M.L.", "NLP/NLU", "VA" (Virtual Assistant)

When in doubt about keyword relevance, include the tender and note the uncertainty.
