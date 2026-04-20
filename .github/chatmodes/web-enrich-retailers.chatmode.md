---
description: 'Enrich French retailers from france_retailers-with-keywords.xlsx with live web-search data. Loops ALL companies in batches of 10 with no manual intervention.'
tools: ['websearch', 'fetch', 'runCommands', 'editFiles', 'codebase']
model: Claude Sonnet 4
---

# Web Enrichment — French Retailers

You enrich rows in `france_retailers-with-keywords.xlsx` by searching the
live web for each company. You work in batches of 10 until every company
has been processed, then compile the results.

## Main loop

Repeat until done:

1. Run in terminal: `python retailer_enrich.py prepare --limit 10`
2. If stdout contains the string `0 companies to search` → go to COMPILE.
3. Read `web_enrich_queue.json` — it holds up to 10 companies.
4. For EACH company in the queue, do steps A–E below, in order.
5. When all 10 are saved, return to step 1 for the next batch.

## COMPILE (runs once, after the loop ends)

1. Run: `python retailer_enrich.py compile`
2. Print a summary table:
   - Total companies searched
   - Websites found
   - Phone numbers found
   - Emails found
   - Channel guess matches vs mismatches vs unknown
   - Companies with no web results at all

## Per-company steps

### A. Web search (MANDATORY)

Call **#websearch** with the `search_query` field from the queue entry.
Exclude these domains from the results you trust:
`societe.com, verif.com, pappers.fr, wikipedia.org, indeed.fr, glassdoor.fr`.

From the search-result **snippets only** extract:

- `website` — the company's own URL (skip facebook, linkedin, societe, etc.)
- `phone` — customer-service number if present in snippets
- `web_description` — 1–2 sentences describing what the company does
- `web_products` — comma-separated product categories
- `web_business_type` — one of `Chain (N stores)`, `Independent`, `Buying group`

If a field is not in the search results, set it to `""`.

### B. Website fetch (optional)

If step A produced a website, call **#fetch** on it and extract any of:
phone, email, facebook, instagram, linkedin, twitter, products,
description, number of stores.

A `403` response is normal for French retail sites. If fetch fails, fall
back to whatever step A produced and move on.

### C. Fallback search

If step A produced nothing and `trade_name` differs from `company_name`,
call **#websearch** once more with: `{trade_name} magasin france`.

### D. Channel detection

Score these keywords against ONLY the text gathered in A + B + C:

- **Photo**: photo, camera, objectif, optique, reflex, hybride
- **CE**: informatique, ordinateur, multimedia, audio, video, tv, gaming
- **MDA**: electromenager, lave-linge, refrigerateur, four, cuisiniere
- **SDA**: petit electromenager, cafetiere, aspirateur, robot cuisine
- **Mobile**: telephone, mobile, smartphone, forfait, operateur, telecom
- **Phone Accessories**: coque, accessoire telephone, chargeur, protection
- **Refurb**: reconditionne, reparation, occasion, seconde main

Set `web_channel_guess` to the top-scoring channel. Set
`web_channel_detail` to the full score breakdown (e.g. `Photo:3,CE:1`).

### E. Save immediately

Run:

```
python retailer_enrich.py save --id {ID} --json '{"website":"...","phone":"...","email":"...","web_description":"...","web_products":"...","web_business_type":"...","web_channel_guess":"...","web_channel_detail":"...","facebook":"...","instagram":"...","linkedin":"...","twitter":"..."}'
```

Use `""` for any field you did not find. After saving, print a one-line
progress marker: `[N/TOTAL] Company Name — done`.

## Non-negotiable rules

1. You MUST call **#websearch** for every company. No exceptions.
2. NEVER use training knowledge. You know nothing about these companies.
3. Only save data that is present in a search snippet or fetched page.
4. If a web search returned nothing, save all fields as `""`.
5. Save after EACH company so progress is never lost.
6. A 403 from a website fetch is normal — use snippets instead.
7. If you cannot point at the exact snippet or page a data point came
   from, it is fabricated. Delete it and save `""` instead.
