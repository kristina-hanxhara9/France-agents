---
name: web-enrich-retailers
description: Enrich the NEXT 10 un-processed French retailers from france_retailers-with-keywords.xlsx with live web data. Resumable — run again for the next batch.
user-invocable: true
---

# Retailer Enrichment Agent

Each invocation enriches **exactly the next 10 un-processed rows** from `france_retailers-with-keywords.xlsx`, then **stops**. Run the agent again for the next batch until all rows are done.

## Steps

1. Read `france_retailers-with-keywords.xlsx` at the repo root. Treat the xlsx row number (first data row = 2) as the stable `row_index`.
2. If `france_retailers-enriched.json` exists, load it. Collect every `row_index` already present — those rows are **done**, skip them.
3. Pick the next **10 rows** whose `row_index` is NOT in the done set. If fewer than 10 remain, take whatever's left. If zero remain, print `All rows enriched — nothing to do.` and stop.
4. For each row, do A–D.
5. Print: `Batch complete — {new} new, {done}/{total} overall. Run the agent again for the next batch.`
6. **Stop.** Never start another batch in the same turn.

### A. Web search (MANDATORY)
Use `web_search` with the row's `search_query` column (or build `{company_name} {trade_name} france` if missing). From the snippets **only**, extract: `website`, `phone`, `email`, `web_description`, `web_products`, `web_business_type`.

### B. Page fetch (optional)
If a website was found, use `web_fetch` on it and prefer page-body data for: `phone`, `email`, `facebook`, `instagram`, `linkedin`, `twitter`, `web_products`, `web_description`. A `403` is normal — fall back to snippets.

### C. Channel guess
Score against search + fetch text only; top scorer becomes `web_channel_guess`:
- **Photo**: photo, camera, objectif, optique, reflex
- **CE**: informatique, ordinateur, audio, video, tv, gaming
- **MDA**: electromenager, lave-linge, refrigerateur, four
- **SDA**: petit electromenager, cafetiere, aspirateur
- **Mobile**: telephone, mobile, smartphone, forfait, operateur
- **Phone Accessories**: coque, chargeur, protection
- **Refurb**: reconditionne, reparation, occasion, seconde main

### D. Save (per row)
- Append a Markdown block to `france_retailers-enriched.md` (create if missing).
- Read, parse, push the record, write back to `france_retailers-enriched.json`. One big array, `row_index` first on every record.

## Rules
1. `web_search` for **every** row. Never use training knowledge.
2. Only save data visible in a search snippet or fetched page. Missing → `""`.
3. Exclude: `societe.com, verif.com, pappers.fr, wikipedia.org, indeed.fr, glassdoor.fr`.
4. Save per row, never per batch — interruption loses ≤1 row.
5. Hard-stop after 10 rows.

## Output format

Markdown block per row:

```
- **row_index:** 2
  **Company:** Fnac Darty
  **Trade name:** Fnac
  **Website:** https://www.fnac.com
  **Phone:** 0892 350 100
  **Email:** serviceclient@fnac.com
  **Channel guess:** CE
  **Description:** French retail chain selling consumer electronics, books, and media.
  **Products:** TV, audio, gaming, informatique, photo
  **Business type:** Chain (300+ stores)
  **Social:** facebook.com/fnac.france, instagram.com/fnac
```

JSON record per row:

```json
{
  "row_index": 2,
  "company_name": "Fnac Darty",
  "trade_name": "Fnac",
  "website": "https://www.fnac.com",
  "phone": "0892 350 100",
  "email": "serviceclient@fnac.com",
  "web_description": "French retail chain selling consumer electronics, books, and media.",
  "web_products": "TV, audio, gaming, informatique, photo",
  "web_business_type": "Chain (300+ stores)",
  "web_channel_guess": "CE",
  "facebook": "https://facebook.com/fnac.france",
  "instagram": "https://instagram.com/fnac",
  "linkedin": "",
  "twitter": ""
}
```
