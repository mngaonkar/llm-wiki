# A Wiki Your Coding Agent Writes (Not RAG Over a Folder)

*Drop in a source. The agent files it into cross-linked markdown pages, answers from those pages, and keeps an append-only log. Next time you ask, it does not rediscover the same facts from scratch.*

Most people’s experience with LLMs and documents is RAG: upload a pile of files, retrieve chunks at query time, generate an answer. That works. It also throws the synthesis away. Ask a question that needs five documents, and the model hunts and stitches fragments again. Nothing accumulates.

**llm-wiki** is an [Agent Skill](https://agentskills.io) that does the other thing. The agent incrementally **builds and maintains a persistent wiki** — ordinary markdown, usually in an [Obsidian](https://obsidian.md) vault. You curate sources and ask questions. The agent does the bookkeeping: entity pages, cross-references, contradiction notes, the index, the log.

**Repo:** [https://github.com/mngaonkar/llm-wiki](https://github.com/mngaonkar/llm-wiki)

The idea is [Andrej Karpathy](https://karpathy.ai)’s — [LLM Wiki: a personal, compounding knowledge base](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). His gist is an idea file you paste into an agent and instantiate. This repo is that pattern packaged as `SKILL.md` for **Claude Code**, **Grok**, and **Gemini CLI**. It is not affiliated with Karpathy.

It is also **not** [nvk/llm-wiki](https://github.com/nvk/llm-wiki) (`wiki@llm-wiki`). Different project. Plugin name here is `llm-wiki`.

The wiki itself is **not** this repo. It lives in a folder you choose (`wiki_root`).

## The idea in one picture

```
You (sources, questions)
        │
        ▼
   coding agent  ──reads──►  SKILL.md + config.json
        │
        │  writes only inside wiki_root
        ▼
   markdown wiki                 you browse
   ┌─────────────────┐           ┌──────────┐
   │ index.md        │◄─────────►│ Obsidian │
   │ log.md          │           │ graph,   │
   │ schema.md       │           │ backlinks│
   │ Person.md       │           └──────────┘
   │ Project.md      │
   └─────────────────┘
        ▲
        │
   ingest / query / lint
```

Three layers, same as the gist:

**Raw sources** are immutable. LinkedIn PDF, GitHub API, a Medium RSS feed, a paper, a URL. The agent reads them. It never edits them.

**The wiki** is the compiled layer. One page per person, project, concept. `[[WikiLinks]]` between them. YAML frontmatter so Obsidian shows Properties. A source that contradicts an existing fact is **flagged**, not silently overwritten.

**The skill** is the schema and the workflow. `SKILL.md` tells the agent when to INIT, INGEST, QUERY, LINT, and STATUS, and that every write stays inside `wiki_root` and is logged the same turn.

The split is the whole design. RAG is an interpreter — re-derive on every question. A wiki is a compiler — integrate once, keep current, query the compiled pages.

## What it looks like

I pointed it at my Obsidian vault, ingested a LinkedIn export, a GitHub profile, and a Medium feed, then said **tell me about myself**.

The agent did not scrape the open web for a biography. It read `index.md`, opened the person page and the catalog pages, and answered from those files — role, career path, certs, public repos, writing, and the gaps the wiki does not cover. A GitHub catalog of 74 repos became **one** catalog page plus a handful of flagship project pages, not 74 notes. A Medium feed became one catalog plus the posts that already had identity in the wiki.

That is the point. Chat history evaporates. The wiki is still there in the next session.

A typical first hour:

1. **Start a wiki for my notes. Store it in /absolute/path/to/vault.**
2. **Ingest https://github.com/you** (or a PDF, a URL, pasted text, LinkedIn, a blog).
3. **What does the wiki say about X?**
4. **Lint the wiki.**

One ingest can touch a dozen pages: new entity pages, updates to related pages, `index.md`, and a log entry at the top of `log.md`.

## Installing it

Install from GitHub. Do **not** also keep a copy in `~/.claude/skills/llm-wiki` (or `~/.grok/skills/…`). The plugin and a user-skill folder fight over the name `llm-wiki`. If you already copied the folder there, remove it after the plugin install.

**Claude Code**

```bash
claude plugin marketplace add mngaonkar/llm-wiki
claude plugin install llm-wiki@llm-wiki
```

This session only, no install: `claude --plugin-dir /path/to/llm-wiki`.

**claude.ai / Cowork:** zip this folder so `SKILL.md` is at the zip root, enable code execution, **Customize → Skills → +**.

**Grok**

```bash
grok plugin install mngaonkar/llm-wiki --trust
grok plugin enable llm-wiki
```

Grok installs the git repo as the plugin (`grok plugin install owner/repo`). Adding the repo as a marketplace and then `grok plugin install llm-wiki` is **not** enough.

**Gemini CLI**

```bash
gemini skills install https://github.com/mngaonkar/llm-wiki.git --scope user
```

Then `/skills reload` and `/skills list`.

After Grok or Claude, start a **new session** (or `/plugins` → reload) so a leftover skills-dir copy is gone.

Forked? Replace `mngaonkar/llm-wiki` with your `owner/repo`.

**Manual fallback** (only if you cannot use a plugin): clone into `~/.claude/skills/llm-wiki`, `~/.grok/skills/llm-wiki`, or `~/.gemini/skills/llm-wiki`. Pick **either** plugin **or** user skill, not both.

Update later with `grok plugin update llm-wiki` or `claude plugin marketplace update llm-wiki`.

## Point it at a wiki

`wiki_root` is an **absolute** path to the markdown folder — the vault root, or a `wiki/` subfolder if you do not want `index.md` mixed with daily notes. On Windows use forward slashes (`C:/Users/you/wiki`).

The agent will **ask** if `config.json` is missing. To set it yourself, copy `config.json.example` to `config.json` **next to the installed `SKILL.md`**:

```json
{
  "wiki_root": "/absolute/path/to/your/wiki"
}
```

Where that file lives (`grok plugin details llm-wiki` / `claude plugin details llm-wiki` print the directory):

| How you installed | Put `config.json` here |
|---|---|
| Grok plugin | `~/.grok/installed-plugins/llm-wiki-<id>/config.json` (next to `SKILL.md`) |
| Claude plugin | `~/.claude/plugins/cache/llm-wiki/llm-wiki/<version>/config.json` |
| Gemini | The directory `gemini skills install` created (`/skills list` shows it) |
| User skill (manual copy only) | `~/.claude/skills/llm-wiki/` (or `~/.grok/skills/…`, `~/.gemini/skills/…`) |

Do not commit someone else’s `config.json`. This repo gitignores it.

Empty folder → say **Start a wiki for …** as above. **Existing vault → do not INIT.** Copy `config.json`, set `wiki_root`, say **lint the wiki**. The skill matches filenames already in the vault (Title Case with spaces vs `kebab-case`).

The hard rule the skill enforces: **every file the agent creates or edits lives inside `wiki_root`.** Sources may live anywhere. Destinations may not.

## Talking to it

| You say | What happens |
|---|---|
| Start a wiki / create a wiki | INIT |
| Ingest this / ingest my GitHub / blog / LinkedIn | INGEST (a catalog source gets one index page, not a page per repo) |
| What does the wiki say about X / tell me about myself | QUERY (wiki-first; gaps called out) |
| Lint the wiki / wiki status | LINT / STATUS |
| Update the plan to cover X | UPDATE an existing plan page; do not re-ingest |

Every write is logged in `wiki_root/log.md` the same turn, with a real timestamp. Read-only queries are not logged.

Catalog ingest is the rule that keeps a GitHub or Medium account from exploding the vault: one catalog page for the account; extra pages only for things that already have wiki identity, or that are clearly flagship.

## Obsidian is the browser

The agent writes ordinary `.md` files. Obsidian is how you look at them: graph, backlinks, search, Properties. Keep this **skill** in the agent/plugin directory. Keep the **wiki** in the vault. Do not copy this repo into the vault as notes.

1. Vault folder on disk → that path is `wiki_root`.
2. Set `config.json` as above.
3. Open the folder as a vault. Agent writes show up immediately (filesystem watch). No Obsidian plugin required.

| File | Role |
|---|---|
| `index.md` | Map of content. Pin it. |
| `log.md` | Append-only ingest history. Noisy on the graph; leave it, or exclude it from graph view. |
| `schema.md` | Naming and entity types. Optional if the vault predates INIT. |
| Topic pages | One note per person / project / concept. YAML (`type`, `aliases`) shows as Properties. |

Pages use `[[WikiLinks]]`. Obsidian resolves by **filename stem**, case-insensitive. **Space ≠ hyphen:** `[[GLM 5.2]]` does not find `glm-5.2.md` unless that file has `aliases: ["GLM 5.2"]`. The skill sets `aliases` to the title used in `index.md`.

Sources **outside** the vault are `file:///` links, not wikilinks. URLs stay markdown links.

Lint indexes every `.md` under `wiki_root` except `index.md` / `log.md` / `schema.md`. If the whole vault is `wiki_root`, daily notes land under Miscellaneous unless you point `wiki_root` at a wiki subfolder.

Sync (Obsidian Sync, iCloud, Git, Syncthing) is fine — the wiki is just files. Avoid two agents writing the same vault at once. The agent must not edit `.obsidian/`.

Karpathy’s line still holds: Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase.

## What this is not

It is not a vector database, an embedding pipeline, or NotebookLM. There is no search engine in this repo. At moderate scale the agent reads `index.md` and greps the pages.

It is not a hosted wiki product. There is no server, no account, no sync of your notes through this skill. Files stay on your disk.

It is not a replacement for your sources. The LinkedIn PDF and the GitHub API remain the source of truth; the wiki is the compiled view, and it can be wrong. That is why contradictions are flagged instead of overwritten, and why query answers list **gaps**.

It is not Karpathy’s gist, and it is not [nvk/llm-wiki](https://github.com/nvk/llm-wiki). This is one skill packaging of the gist for coding-agent CLIs.

What it *is*: ingest → pages → query → lint, as a `SKILL.md` the agent actually follows, with path safety and an append-only log so you can see what it did.

## In this repo

```
SKILL.md                      # when to run, how to write the wiki
config.json.example           # copy to config.json next to SKILL.md
references/                   # index.md and log.md formats
.claude-plugin/               # Claude Code plugin + marketplace
.grok-plugin/                 # Grok marketplace
LICENSE                       # MIT
```

You need Claude Code, Grok, or Gemini CLI (anything that loads `SKILL.md` folders) and local file read/write at `wiki_root`. Obsidian is optional for ingest and query; it is the best way to browse.

## Links

- **GitHub:** [mngaonkar/llm-wiki](https://github.com/mngaonkar/llm-wiki)
- **Karpathy’s gist:** [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- **Agent Skills:** [agentskills.io](https://agentskills.io)
- **Obsidian:** [obsidian.md](https://obsidian.md)

MIT license. See [LICENSE](LICENSE).

## Closing

Humans abandon wikis because the maintenance grows faster than the value — updating cross-references, keeping summaries current, noticing when a new source contradicts an old claim. The agent does not get bored, and it can touch fifteen files in one pass.

Your job is sources and questions. The agent’s job is the rest.

Install the skill, point `wiki_root` at a folder (or an existing vault), ingest one source, and ask what the wiki says about it. Then open Obsidian and follow the links.
