# Setup — France Retailer Enrichment (GitHub Copilot CLI)

This repo ships a Copilot CLI custom agent that enriches retailers from
`france_retailers-with-keywords.xlsx`, **10 rows per invocation**,
resuming from `france_retailers-enriched.json` so you can run it over
and over until the whole sheet is done.

**Use the Copilot CLI** — it has web search and URL fetch built in, no
extensions, no API keys. (VS Code's Copilot does NOT have built-in web
search; that path requires the Tavily-backed extension — see Appendix.)

## Prerequisites

### 1. Install the Copilot CLI

```bash
npm install -g @github/copilot
```

Sign in once:

```bash
copilot
```

(You'll be prompted to authenticate with GitHub on first launch.)

Docs: https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli-agents/overview

### 2. Drop your xlsx at the repo root

Place `france_retailers-with-keywords.xlsx` in the project root. The
agent reads it from that path.

### 3. That's it

The `web` tool (search + fetch) is built into the CLI — no extension,
no API key. The agent declares `tools: ['web', 'edit', 'execute', 'read', 'search']`
and those all come for free.

## Running the agent

### Interactive

```bash
copilot
```

Once inside, type `/agent` and pick **web-enrich-retailers**, then
send `go` (or any message). It processes rows 2–11 and stops. Repeat:

```
/agent web-enrich-retailers
go
```

…until it prints `All rows enriched — nothing to do.`

### One-shot (scriptable)

Each invocation = one batch of 10:

```bash
copilot --agent web-enrich-retailers --prompt "go"
```

Wrap it in a shell loop if you just want to let it rip:

```bash
while true; do
  out=$(copilot --agent web-enrich-retailers --prompt "go")
  echo "$out"
  echo "$out" | grep -q "All rows enriched" && break
done
```

## Outputs

- **`france_retailers-enriched.md`** — human-readable, appended per row.
- **`france_retailers-enriched.json`** — structured data. Each record's
  first field is `row_index` (the xlsx row number). This file doubles
  as the resume cursor; delete it to start over.

## Sanity-check the first batch

1. Run one batch.
2. Open `france_retailers-enriched.json`. Confirm 10 entries, each with
   `row_index` first.
3. Pick 2 rows at random. Google them yourself. If any website / phone
   doesn't appear in real search results, the model is using training
   data — sharpen the "only save data visible in snippets" rule in
   `.github/agents/web-enrich-retailers.agent.md`.

## Layout

```
.github/
  agents/
    web-enrich-retailers.agent.md   ← CLI agent (PRIMARY)
  prompts/
    enrich-retailers.prompt.md      ← VS Code prompt variant (optional)
  copilot-instructions.md           ← repo-wide rules
prompt-enrich-retailers.md          ← human-readable reference
france_retailers-with-keywords.xlsx ← you provide this (gitignored)
france_retailers-enriched.json      ← generated, resume cursor
france_retailers-enriched.md        ← generated, human-readable
```

## Appendix: the VS Code path (more fiddly)

The VS Code Copilot extension does **not** include a built-in web
search tool. To use the same agent file in VS Code Chat you'd need:

1. Install the **Web Search for Copilot** extension by Microsoft
   (https://github.com/microsoft/vscode-websearchforcopilot).
2. Get a free Tavily API key at https://tavily.com and paste it when
   the extension asks.
3. Note that VS Code uses different tool identifiers (`web/fetch`,
   `edit/editFiles`, `execute/runInTerminal`, etc.) — this agent file
   uses the CLI naming (`web`, `edit`, `execute`).

For this project, the CLI path is strictly simpler and is the
recommended setup.

## References
- GitHub Copilot CLI overview: https://docs.github.com/en/copilot/concepts/agents/about-copilot-cli
- Custom agents configuration: https://docs.github.com/en/copilot/reference/custom-agents-configuration
- Create custom agents for CLI: https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli
- Sample agents: https://github.com/github/copilot-cli-for-beginners/tree/main/samples/agents
