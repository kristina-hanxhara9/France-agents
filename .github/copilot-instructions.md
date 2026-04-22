# France retailers enrichment — repo-wide instructions

This repository enriches French retailer records in
`france_retailers-with-keywords.xlsx` with live web data. Any Copilot
prompt or chat session in this repo MUST follow these rules.

## Data integrity
- **No training data.** You know nothing about these companies. Every
  saved field must trace back to a specific web-search snippet or a
  fetched webpage.
- **Empty over wrong.** If a field isn't in the web results, save `""`.
  Never guess, infer, or approximate.
- **Cite or delete.** If you can't point at the exact snippet/page a
  data point came from, it's fabricated — drop it.

## Tool usage
- Use `#websearch` (from the "Web Search for Copilot" extension) with
  the row's `search_query` column as the query when present.
- Use `#web/fetch` for page loads. `403` responses are normal on French
  retail sites — fall back to search snippets.
- Use `#edit/editFiles` and `#edit/createFile` to write the output
  files. Use `#execute/runInTerminal` if you need shell access.
- Exclude these domains from trusted sources:
  `societe.com, verif.com, pappers.fr, wikipedia.org, indeed.fr, glassdoor.fr`.

## Output
- Produce both `france_retailers-enriched.md` and
  `france_retailers-enriched.json`.
- Work in batches of 10 rows and **append** each batch to both files
  before continuing, so nothing is lost on interruption.
