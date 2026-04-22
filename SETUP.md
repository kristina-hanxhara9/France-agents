# Setup — France Retailer Enrichment (Copilot CLI)

The Copilot CLI ships `web_search` and `web_fetch` as built-in tools,
so this agent needs zero extra setup beyond signing in.

## Install

```bash
npm install -g @github/copilot
copilot   # first launch prompts GitHub sign-in
```

## Use

1. Drop `france_retailers-with-keywords.xlsx` at the repo root.
2. From the repo root:

```bash
copilot
```

3. Inside Copilot, type `/agent`, pick **web-enrich-retailers**, send any
   message (e.g. `go`). It processes the next 10 rows and stops.
4. Repeat step 3 until it prints `All rows enriched — nothing to do.`

### One-shot loop (unattended)

```bash
while true; do
  out=$(copilot --agent web-enrich-retailers --prompt "go")
  echo "$out"
  echo "$out" | grep -q "All rows enriched" && break
done
```

## Outputs

- `france_retailers-enriched.md` — human-readable, appended per row.
- `france_retailers-enriched.json` — structured data; each record's
  first field is `row_index`. This file doubles as the resume cursor —
  delete it to start over.

## Verify after the first batch

Open `france_retailers-enriched.json`, pick two rows at random, google
them yourself. If any website or phone in the JSON doesn't show up in
real search results, the model is fabricating — tighten the "only save
data visible in snippets" rule in
`.github/agents/web-enrich-retailers.agent.md`.

## References
- [Copilot CLI overview](https://docs.github.com/en/copilot/concepts/agents/about-copilot-cli)
- [Custom agents configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [Creating custom agents for CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli)
