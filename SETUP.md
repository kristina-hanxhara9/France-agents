# Setup — France Retailer Enrichment for GitHub Copilot

This repo has two equivalent ways to run the enrichment, both batching
10 rows per invocation and resuming from `france_retailers-enriched.json`:

| Entry point | Where it lives | How you invoke it |
|---|---|---|
| **Custom agent** (recommended) | `.github/agents/web-enrich-retailers.agent.md` | Chat panel → agents dropdown → `web-enrich-retailers`, or type `/agents` → pick it |
| **Prompt file** | `.github/prompts/enrich-retailers.prompt.md` | Type `/enrich-retailers` in Copilot Chat |

## Prerequisites

### 1. VS Code + GitHub Copilot
- Install **GitHub Copilot** + **GitHub Copilot Chat** extensions.
- Sign in.

### 2. A web-search tool (this is the part that trips everyone up)

GitHub Copilot does **not** ship with a built-in web-search tool
usable from a prompt file. You have two options — pick one:

#### Option A: Install "Web Search for Copilot" (free-tier friendly)
1. In VS Code, install the extension
   **Web Search for Copilot** (publisher: Microsoft).
   Repo: https://github.com/microsoft/vscode-websearchforcopilot
2. Get a free Tavily API key at https://tavily.com and paste it when
   the extension prompts you (stored in VS Code secret storage).
3. Confirm the tool shows up: in Copilot Chat type `#` — you should
   see `#websearch` in the suggestion list.

This is what the agent/prompt files in this repo use (`websearch` in
the `tools:` frontmatter).

#### Option B: Use `@github #web` (Bing, enterprise-gated)
Only works if your GitHub org enabled the "Copilot Access to Bing"
policy. If it is, you can manually trigger it per message with
`@github #web <query>`, but this does not integrate cleanly with
prompt files / custom agents — you'd have to invoke it by hand.

If neither option is available, the agent will silently run without
web access and produce empty fields — so confirm Option A before you
start.

### 3. Your xlsx
Drop `france_retailers-with-keywords.xlsx` at the repo root.

## Usage

### Using the custom agent (recommended)
1. Open Copilot Chat.
2. In the chat input, click the agent dropdown (or type `/agents`) and
   pick **web-enrich-retailers**.
3. Send any message (e.g. `go`). It will do rows 2–11 and stop.
4. Send the message again for the next 10, until it prints
   `All rows enriched — nothing to do.`

### Using the prompt file
Same thing, just type `/enrich-retailers` each time instead of
selecting an agent.

## Outputs
- `france_retailers-enriched.md` — human-readable summary, appended per row.
- `france_retailers-enriched.json` — structured data, keyed by `row_index`. This file doubles as the resume cursor — delete it to start over.

## Things to verify on the first batch
1. Run one batch.
2. Open `france_retailers-enriched.json` — confirm it has 10 entries
   with `row_index` as the first field on each.
3. Pick 2 rows at random and google them yourself. If any website /
   phone in the JSON doesn't appear in real search results, the model
   is falling back on training data — tighten the "only save data
   visible in snippets" rule in the agent file.

## Reference
- Copilot prompt files: https://code.visualstudio.com/docs/copilot/customization/prompt-files
- Copilot custom agents: https://code.visualstudio.com/docs/copilot/customization/custom-agents
- Built-in tool names: https://code.visualstudio.com/docs/copilot/reference/copilot-vscode-features
- Web Search extension: https://github.com/microsoft/vscode-websearchforcopilot
