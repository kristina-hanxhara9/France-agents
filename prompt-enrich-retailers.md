## Task:
Each time this prompt is run, enrich **exactly the next 10 un-processed rows** from `france_retailers-with-keywords.xlsx`, then **stop** and tell the user to run it again for the next batch.

### Step-by-step
1. Read `france_retailers-with-keywords.xlsx` (project root). Treat the row number (starting at 2 for the first data row) as the stable `row_index`.
2. If `france_retailers-enriched.json` exists, load it and collect every `row_index` already present. These are **done** — skip them.
3. Pick the next **10 rows** whose `row_index` is NOT in the done set. If fewer than 10 remain, process whatever's left. If zero remain, print `All rows enriched — nothing to do.` and stop.
4. For each row in the batch, do A–D below.
5. After the batch is saved, print:
   `Batch complete — {processed_this_batch} new, {total_done}/{total_rows} overall. Run the prompt again for the next batch.`
6. **Stop.** Do NOT start another batch in the same turn.

### A. Web search (MANDATORY)
Call `#websearch` with the row's `search_query` column, or build `{company_name} {trade_name} france` if missing. From the search-result **snippets only**, extract: website, phone, email, web_description, web_products, web_business_type.

### B. Website fetch (optional)
If a website was found, call `#web/fetch` on it and prefer page-body data for: phone, email, facebook, instagram, linkedin, twitter, web_products, web_description. A `403` is normal — fall back to snippets.

### C. Channel guess
Score against search+fetch text only; top scorer becomes `web_channel_guess`:
- **Photo**: photo, camera, objectif, optique, reflex
- **CE**: informatique, ordinateur, audio, video, tv, gaming
- **MDA**: electromenager, lave-linge, refrigerateur, four
- **SDA**: petit electromenager, cafetiere, aspirateur
- **Mobile**: telephone, mobile, smartphone, forfait, operateur
- **Phone Accessories**: coque, chargeur, protection
- **Refurb**: reconditionne, reparation, occasion, seconde main

### D. Append to both output files (after EACH row)
- Append to `france_retailers-enriched.md` in the Markdown format below.
- Append to `france_retailers-enriched.json` — one big array; read, push, write back. Always include `row_index` as the first field.

### Rules (non-negotiable)
- `#websearch` for **every** row. Never use training knowledge.
- Only save data visible in a snippet or fetched page. Missing → `""`.
- Exclude: `societe.com, verif.com, pappers.fr, wikipedia.org, indeed.fr, glassdoor.fr`.
- Save per row, not per batch — interruption loses ≤1 row.
- Hard-stop after 10 rows per invocation.

### Example Output: Markdown

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

- **row_index:** 3
  **Company:** Boulanger SAS
  **Trade name:** Boulanger
  **Website:** https://www.boulanger.com
  **Phone:** 03 59 35 20 00
  **Email:** ""
  **Channel guess:** MDA
  **Description:** French specialist retailer of home appliances and electronics.
  **Products:** electromenager, tv, informatique, smartphone
  **Business type:** Chain (170+ stores)
  **Social:** facebook.com/boulanger, instagram.com/boulanger_officiel

### Example Output: JSON
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
  },
  {
    "row_index": 3,
    "company_name": "Boulanger SAS",
    "trade_name": "Boulanger",
    "website": "https://www.boulanger.com",
    "phone": "03 59 35 20 00",
    "email": "",
    "web_description": "French specialist retailer of home appliances and electronics.",
    "web_products": "electromenager, tv, informatique, smartphone",
    "web_business_type": "Chain (170+ stores)",
    "web_channel_guess": "MDA",
    "facebook": "https://facebook.com/boulanger",
    "instagram": "https://instagram.com/boulanger_officiel",
    "linkedin": "",
    "twitter": ""
  }
]
