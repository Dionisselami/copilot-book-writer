# copilot-book-writer

**Write a book in VS Code with Copilot agent mode.** One `.vscode/mcp.json`, one instructions file,
and the [Proseify](https://proseify.xyz) MCP server turns Copilot from an autocomplete that can also write prose
into a book-writing engine with a genre discipline and a scoring gate.

[![Listed on mcpservers.org](https://mcpservers.org/badge.svg)](https://mcpservers.org/servers/dionisselami/proseify-mcp) [![MCP Registry](https://img.shields.io/badge/MCP_registry-io.github.Dionisselami%2Fproseify--mcp-4a3f35)](https://registry.modelcontextprotocol.io/v0/servers?search=proseify)

---

## Why this is not just "add an MCP server"

VS Code's MCP configuration has one genuinely confusing detail and one genuinely useful one.

The confusing detail: the top-level key is **`servers`**, not `mcpServers` — unlike Cursor, Claude
Desktop, Cline, Continue and everything else. Get it wrong and VS Code shows no error at all; the
server simply never appears, and the tools quietly are not there.

The useful detail: `.vscode/mcp.json` is a committed file, so VS Code supports `$${input:proseify-key}`
input variables that prompt once and store the value in OS secret storage. That means you can share
this repo, and your key still never lands in git.

```text
premise  →  genre recipe  →  corpus passages  →  outline  →  chapter drafts  →  evaluate_book  →  revision
```

Never let the model start at "chapter drafts". The ordering is the whole product: a genre
recipe read *before* prose beats any amount of "write like Stephen King" prompting, because it
hands the model concrete numbers — pacing beats, dialogue ratio, sentence-length profile — that
its defaults would otherwise flatten to its own house style.

---

## Quick start

### 1. Get a Proseify key

https://proseify.xyz — sign in, pick a plan (from $9/mo, or the one-time Founding Lifetime tier),
key issued on payment.

### 2. Open this repo (or copy `.vscode/mcp.json` into yours)

```json
{
  "inputs": [
    { "id": "proseify-key", "type": "promptString",
       "description": "Proseify API key", "password": true }
  ],
  "servers": {
    "proseify": {
      "type": "http",
      "url": "https://mcp.proseify.xyz/mcp",
      "headers": { "Authorization": "Bearer ${input:proseify-key}" }
    }
  }
}
```

VS Code prompts for the key once and stores it securely. Then start the server: open
`.vscode/mcp.json` and click **Start** above the entry, or run **MCP: List Servers** from the
Command Palette.

### 3. Switch Copilot Chat to agent mode

This is a hard requirement — VS Code discovers the server without Copilot, but MCP tools are only
callable from Copilot Chat's **agent** mode, not Ask or Edit. Open the tools picker in the chat
panel and confirm the seven Proseify tools are listed.

### 3. Install the writing skill

```bash
npx skills add https://proseify.xyz --skill anti-prose-slop -y
```

The MCP server gives the agent the *tools*. The `anti-prose-slop` skill gives it the *method* —
genre recipe first, corpus grounding, chapter discipline, and the per-chapter edit pass. Install
both or you get tool access and the same generic prose you had before.

### 4. Give it a premise

> Write a 20-chapter comedy of manners about a family that inherits a failing seaside hotel and a
> guest book full of grudges. Around 45,000 words.

---

## What's in here

| File | Purpose |
|------|---------|
| `.vscode/mcp.json` | The `proseify` server, key via `$${input:proseify-key}` — never written to the file |
| `.github/copilot-instructions.md` | Repo-wide instructions Copilot loads automatically |
| `LICENSE` | MIT |

### The five-step loop

1. **Pick the genre before any prose exists.** `list_genres`, then `get_genre_recipe` for the
   closest fit. Blends are allowed — name both and let the recipe argue for one.
2. **Ground the register.** `search_corpus` (or `get_style_references`) for 3–5 model passages
   in the target genre. Read them. This is the calibration step, not decoration.
3. **`plan_book` once.** Hold the outline. Do not re-plan mid-draft; if the outline is wrong,
   fix it deliberately and say so.
4. **Draft chapter by chapter**, pausing after each for the per-chapter edit pass below.
5. **`evaluate_book` before you call it done.** Fix the chapters it flags and re-run the gate.

### The per-chapter edit pass

The failure mode of AI prose is not grammar — it is sameness. Every chapter gets struck against
this list before it counts as drafted:

- `seemed to`, `began to`, `started to`, `could feel` — delete or recast
- filter words: *felt, noticed, watched, saw, heard* — put the reader in the perception instead
- stacked adverbs on dialogue tags; `said` is not a problem, `said loudly, angrily` is
- sentence openings repeated across the chapter (and the weather-opening default)
- three-item lists used as rhythm filler
- dialogue that exists to explain the plot to the reader
- simile endings: the last line becoming a metaphor for the chapter

### If you only read one paragraph

The agent's own model does all the writing — Proseify calls no LLM and stores no manuscript.
There is no token bill from us: it is a corpus, a set of genre recipes, and a pipeline.


---

## VS Code and Copilot notes and gotchas

- **`servers`, not `mcpServers`, and `"type": "http"` is mandatory.** Wrong key or missing type is a
  silent failure: no error, no server, no tools. Check these two fields first, every time.
- **Agent mode only.** Ask and Edit modes do not surface MCP tools at all. The tools button in the
  chat panel is where you verify the connection actually worked.
- **Workspace trust is required.** An untrusted workspace skips MCP servers entirely.
- **Copilot checks tool availability at session start.** If you start the server after the chat
  began, start a new chat — the tools are not retrofitted into the running session.
- **Prefer the `$${input:proseify-key}` input over a literal key.** This file is designed to be committed, and a
  hardcoded key in `.vscode/mcp.json` is a published secret waiting for its first push.
- **Review each chapter in the editor while the agent drafts the next one.** Copilot's strength here
  is that you are already looking at the prose in a diff-friendly pane; use that instead of accepting
  a whole book at the end.
- **Large tool schemas cost budget.** If you are writing code in the same workspace, keep the book
  work in its own folder so the seven Proseify tools are not competing with your code context.

| Tool | What it does |
|------|--------------|
| `list_genres` | The 11 genres the corpus covers (theatre, horror, romance, adventure, literary, mystery, gothic, sci-fi, fantasy, comedy, children's) |
| `get_genre_recipe` | Pacing beats, dialogue ratio, sentence-length profile and stylistic anchors derived from that tradition |
| `search_corpus` | FTS5 full-text search across 2,500+ chapters of public-domain classics (quoted phrases, AND/OR/NOT, wildcards) |
| `get_style_references` | Model passages from books in the target genre — the register you are aiming at |
| `plan_book` | A full chapter-by-chapter outline from a one-line premise, each beat carrying a corpus style reference |
| `write_book` | The one-shot flow: plan → draft → evaluate, in a single session |
| `evaluate_book` | Scores a draft against genre benchmarks (structure, pacing, chapter coverage, word budget) so weak chapters get revised |

## The corpus

80+ books and 2,500+ chapters of public-domain literature — Project Gutenberg and similar
sources — every chapter verified against its own file header at download time, so nothing in
the library has a copyright question hanging over it. Works from Austen, Stevenson, Hugo,
Conrad, Verne, the Brontës, Shelley and the gothic masters, grouped by genre and indexed for
full-text search.

Commercial use of what you write on top of it is safe. Full-text searchable chapter by chapter,
not a scrape of titles.

## Pricing

| Plan | Price | Rate limit |
|------|-------|-----------|
| Starter | $9/mo | 120 req/min |
| Pro | $19/mo | 400 req/min |
| Studio | $49/mo | 9999 req/min |
| Founding Lifetime | one-time | 400 req/min, all genres |

Sign in at https://proseify.xyz, pick a plan, and the key is issued the moment the purchase
clears. Cancel from https://proseify.xyz/account; 14-day refund window
(https://proseify.xyz/refunds).

## Other clients

Same Proseify server, same workflow, different config file. The cookbook is the method itself.

- [claude-book-writer — write a book with Claude Code](https://github.com/Dionisselami/claude-book-writer)
- [chatgpt-book-writer — write a book on your ChatGPT plan](https://github.com/Dionisselami/chatgpt-book-writer)
- [cursor-book-writer — write a book in Cursor](https://github.com/Dionisselami/cursor-book-writer)
- [gemini-cli-book-writer — write a book with Gemini CLI](https://github.com/Dionisselami/gemini-cli-book-writer)
- [windsurf-book-writer — write a book in Windsurf](https://github.com/Dionisselami/windsurf-book-writer)
- [claude-book-cookbook — recipes for writing a whole book with Claude](https://github.com/Dionisselami/claude-book-cookbook)

## License

MIT for everything in this repository. The corpus texts themselves are public domain.

## Disclaimer

Unofficial. This repository is not affiliated with, endorsed by, or sponsored by GitHub, GitHub Copilot, Microsoft or Visual Studio Code. Client names appear only to describe which configuration file and transport the instructions are for. Proseify is an independent product — https://proseify.xyz.

Keep your Proseify key in local agent config or an environment variable. Never commit it, print it, log it, or paste it into a repository file.

## Links

- **Sign up / key issuance:** https://proseify.xyz
- **Filled-in config for your key:** https://proseify.xyz/agent
- **FAQ (ownership, KDP, what an MCP server is):** https://proseify.xyz/faq
- **Server repo & MCP registry entry:** https://github.com/Dionisselami/proseify-mcp
- **Support:** support@proseify.xyz
