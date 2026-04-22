---
name: web-enrich-retailers
description: Enrich the NEXT 10 un-processed French retailers from france_retailers-with-keywords.xlsx with live web data. Resumable — run again for the next batch.
user-invocable: true
tools: ['websearch', 'web/fetch', 'edit/editFiles', 'edit/createFile', 'execute/runInTerminal', 'execute/getTerminalOutput']
---

# Retailer Enrichment Agent

## Task
Each invocation enriches **exactly the next 10 un-processed rows** from `france_retailers-with-keywords.xlsx`, then **stops**. The user runs the agent again for the next batch until all rows are done.

## Step-by-step
1. Read `france_retailers-with-keywords.xlsx` (project root). Treat the xlsx row number (first data row = 2) as the stable `row_index`.
2. If `france_retailers-enriched.json` exists, load it and collect every `row_index` already present. These rows are **done** — skip them.
3. Pick the next **10 rows** whose `row_index` is NOT in the done set. If fewer than 10 remain, process whatever's left. If zero remain, print `All rows enriched — nothing to do.` and stop.
4. For each row in the batch, do A–D below.
5. Print one line summary: `Batch complete — {processed} new, {total_done}/{total_rows} overall. Run /web-enrich-retailers again for the next batch.`
6. **Stop.** Do NOT start another batch in the same turn.

## A. Web search (MANDATORY)
Call the **`#websearch`** tool with the row's `search_query` column (or build `{company_name} {trade_name} france` if that column is missing). From the search-result **snippets only**, extract: `website`, `phone`, `email`, `web_description`, `web_products`, `web_business_type`.

## B. Page fetch (optional)
If a website was found, call **`#web/fetch`** on it and prefer any of these from the page body: `phone`, `email`, `facebook`, `instagram`, `linkedin`, `twitter`, `web_products`, `web_description`. A `403` is normal — fall back to snippets.

## C. Channel guess
Score these keywords against ONLY the search+fetch text, set `web_channel_guess` to the top scorer:
- **Photo**: photo, camera, objectif, optique, reflex
- **CE**: informatique, ordinateur, audio, video, tv, gaming
- **MDA**: electromenager, lave-linge, refrigerateur, four
- **SDA**: petit electromenager, cafetiere, aspirateur
- **Mobile**: telephone, mobile, smartphone, forfait, operateur
- **Phone Accessories**: coque, chargeur, protection
- **Refurb**: reconditionne, reparation, occasion, seconde main

## D. Append to both output files (after EACH row)
- Use **`#edit/editFiles`** (or `#edit/createFile` if the file doesn't exist) to append the row's Markdown block to `france_retailers-enriched.md`.
- Read, parse, push the new record, and write back to `france_retailers-enriched.json`. It's one big array; always include `row_index` as the first field.

## Non-negotiable rules
1. Call `#websearch` for **every** row. Never use training knowledge.
2. Only save data visible in a search snippet or a fetched page. Missing → `""`.
3. Exclude these domains: `societe.com, verif.com, pappers.fr, wikipedia.org, indeed.fr, glassdoor.fr`.
4. Save after EACH row — never batch writes — so an interruption loses at most one row.
5. Hard-stop after 10 rows per invocation.

## Example Output: Markdown (appended per row)

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

## Example Output: JSON
[
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
]
