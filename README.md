# A Personal Wiki Your Agent Builds From Your Sources

Drop in a source. The agent files it into cross-linked markdown pages and answers from those pages. Next question does not re-derive the same facts from scratch.

RAG retrieves chunks at query time and throws the synthesis away. **llm-wiki** compiles sources into a wiki once, then keeps that wiki current.

It is an [Agent Skill](https://agentskills.io) (`SKILL.md`) for **Claude Code**, **Grok**, and **Gemini CLI**. The wiki lives in a folder you choose (`wiki_root`) — usually an [Obsidian](https://obsidian.md) vault. This repo is the skill, not the wiki.

Idea: [Andrej Karpathy — LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). This packaging is not affiliated with him.

**Repo:** [https://github.com/mngaonkar/llm-wiki](https://github.com/mngaonkar/llm-wiki)

## How it works

```
source  →  ingest  →  wiki pages  →  query  →  answer
                         ↑                       │
                       lint ←────────────────────┘
```

- **Sources** stay as they are. The agent reads a PDF, URL, GitHub API, or RSS feed. It never edits them.
- **Wiki** is one markdown page per person, project, or concept, linked with `[[WikiLinks]]`. A new source that contradicts an existing fact is flagged, not overwritten.
- **Skill** is `SKILL.md`. It tells the agent to INIT, INGEST, QUERY, or LINT — and to write only inside `wiki_root`, logging every write in `log.md`.

I pointed it at my Obsidian vault, ingested LinkedIn, GitHub, and Medium, then said **tell me about myself**. The agent answered from wiki pages (role, career, certs, repos, posts) and listed gaps. 74 GitHub repos became one catalog page plus a few flagship project pages, not 74 notes.

## Install

Do not also copy this folder into `~/.claude/skills/llm-wiki` or `~/.grok/skills/…`. Plugin and user-skill copies of the same name collide. Remove the skills-dir copy after plugin install. Start a new session.

**Claude Code**

```bash
claude plugin marketplace add mngaonkar/llm-wiki
claude plugin install llm-wiki@llm-wiki
```

This session only: `claude --plugin-dir /path/to/llm-wiki`.

**claude.ai / Cowork:** zip the folder with `SKILL.md` at the zip root → enable code execution → Customize → Skills → +.

**Grok**

```bash
grok plugin install mngaonkar/llm-wiki --trust
grok plugin enable llm-wiki
```

Grok needs `grok plugin install owner/repo`. Marketplace add then `grok plugin install llm-wiki` is not enough.

**Gemini CLI**

```bash
gemini skills install https://github.com/mngaonkar/llm-wiki.git --scope user
```

Then `/skills reload`.

Fork: replace `mngaonkar/llm-wiki` with your `owner/repo`.

Update: `grok plugin update llm-wiki` or `claude plugin marketplace update llm-wiki`.

## Configure

`wiki_root` is an absolute path to the markdown folder. Windows: `C:/Users/you/wiki`.

Copy `config.json.example` to `config.json` next to the installed `SKILL.md`:

```json
{ "wiki_root": "/absolute/path/to/your/wiki" }
```

| Install | `config.json` lives here |
|---|---|
| Grok plugin | `~/.grok/installed-plugins/llm-wiki-<id>/config.json` |
| Claude plugin | `~/.claude/plugins/cache/llm-wiki/llm-wiki/<version>/config.json` |
| Gemini | directory from `/skills list` |
| Manual clone | `~/.claude/skills/llm-wiki/` (or `~/.grok/skills/…`, `~/.gemini/skills/…`) |

`grok plugin details llm-wiki` / `claude plugin details llm-wiki` print the directory. The repo gitignores `config.json`.

Empty folder: **Start a wiki for \<topic\>. Store it in \<absolute-path\>.**

Existing vault: do not INIT. Set `wiki_root`, then **lint the wiki**. The skill keeps the vault’s filenames.

Every write stays inside `wiki_root`. Sources can live anywhere.

## Use

| You say | Agent does |
|---|---|
| Start a wiki / create a wiki | INIT |
| Ingest this / ingest my GitHub, blog, LinkedIn | INGEST — one catalog page per account, not a page per repo |
| What does the wiki say about X / tell me about myself | QUERY — wiki only; lists gaps |
| Lint the wiki / wiki status | LINT / STATUS |
| Update the plan to cover X | UPDATE the plan page; do not re-ingest |

Then: **Ingest \<URL or file\>** and **What does the wiki say about X?**

Writes go in `log.md` the same turn, with a timestamp. Read-only queries are not logged.

## Obsidian

The agent writes `.md` files. Obsidian shows graph, backlinks, and Properties. Keep the skill in the plugin directory; keep the wiki in the vault. Do not copy this repo into the vault.

1. Vault path = `wiki_root`.
2. Set `config.json`.
3. Open the folder as a vault. No Obsidian plugin needed.

Use a subfolder (`…/Obsidian/wiki`) if you do not want `index.md` mixed with daily notes.

| File | Role |
|---|---|
| `index.md` | catalog of pages |
| `log.md` | append-only ingest history |
| `schema.md` | naming rules; skip if the vault already has its own |
| Topic pages | one note per entity; YAML `type` / `aliases` show as Properties |

`[[WikiLinks]]` resolve by filename stem. Space ≠ hyphen: `[[GLM 5.2]]` misses `glm-5.2.md` unless that file has `aliases: ["GLM 5.2"]`. The skill sets that alias.

Sources outside the vault are `file:///` links. URLs stay markdown links.

Lint indexes every `.md` under `wiki_root` except `index.md`, `log.md`, `schema.md`. Point `wiki_root` at a wiki subfolder if you do not want daily notes indexed.

Do not run two agents on the same vault. The agent must not edit `.obsidian/`.

## Limits

No embeddings, no search engine, no server. The agent reads `index.md` and greps pages. Files stay on disk. The wiki can be wrong; contradictions are flagged and query answers list gaps.

## Files

```
SKILL.md                 workflows
config.json.example      copy next to SKILL.md
references/              index.md and log.md formats
.claude-plugin/          Claude marketplace
.grok-plugin/            Grok marketplace
LICENSE                  MIT
```

Needs a coding agent that loads `SKILL.md` and write access to `wiki_root`. Obsidian is optional to browse.

## Links

- [mngaonkar/llm-wiki](https://github.com/mngaonkar/llm-wiki)
- [Karpathy gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [Agent Skills](https://agentskills.io)
- [Obsidian](https://obsidian.md)
