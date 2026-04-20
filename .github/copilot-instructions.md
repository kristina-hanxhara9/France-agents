# France retailers enrichment — repo-wide instructions

This repository enriches French retailer records in
`france_retailers-with-keywords.xlsx` with live web data. Any Copilot
agent, chat mode, or prompt file in this repo MUST follow these rules.

## Data integrity

- **No training data.** You know nothing about these companies. Every
  field you save must trace back to a specific web-search snippet or a
  fetched webpage.
- **Empty over wrong.** If a field isn't in the web results, save `""`.
  Never guess, never infer, never approximate.
- **Cite or delete.** If you can't point at the exact snippet/page a
  data point came from, it's fabricated — drop it.

## Tool usage

- Use `#websearch` for search. The `search_query` column is pre-built;
  use it as-is unless instructed otherwise.
- Use `#fetch` for page loads. `403` responses are normal on French
  retail sites — fall back to search snippets.
- Exclude these domains from sources:
  `societe.com, verif.com, pappers.fr, wikipedia.org, indeed.fr, glassdoor.fr`.

## Persistence

- Save after EVERY company via
  `python retailer_enrich.py save --id {ID} --json '{...}'`.
- Never batch saves. A crash mid-batch must not lose progress.

## Batching

- One batch = 10 companies max.
- Use `python retailer_enrich.py prepare --limit 10` to fetch the next
  batch. Stop when stdout says `0 companies to search`.
- After the loop ends, run `python retailer_enrich.py compile` once.
