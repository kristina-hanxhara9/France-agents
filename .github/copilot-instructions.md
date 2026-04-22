# France retailers enrichment — repo-wide rules

Any Copilot agent in this repo enriching
`france_retailers-with-keywords.xlsx` MUST follow these:

1. **Call `web_search` for every row.** No training-data lookups.
2. **Only save data visible** in a search snippet or a `web_fetch` page
   body. If a field isn't there, save `""`.
3. **Cite or delete** — if you can't point at the exact snippet/page
   a data point came from, it's fabricated. Drop it.
4. **Exclude** these domains from trusted sources:
   `societe.com, verif.com, pappers.fr, wikipedia.org, indeed.fr, glassdoor.fr`.
5. **Save per row**, never per batch — interruption loses ≤1 row.
6. `403` from `web_fetch` is normal on French retail sites. Fall back
   to search snippets.
