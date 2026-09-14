# llm-wiki

Persistent markdown wiki for coding agents. You point at sources; the agent ingests them into cross-linked pages, answers from the wiki, and keeps an append-only log.

**Repo:** [github.com/mngaonkar/llm-wiki](https://github.com/mngaonkar/llm-wiki)

**Credit:** The idea is [Andrej Karpathy](https://karpathy.ai)’s — [LLM Wiki: a personal, compounding knowledge base](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). This is an [Agent Skills](https://agentskills.io) (`SKILL.md`) packaging of that pattern for **Claude Code**, **Grok**, and **Gemini CLI**.

This is **not** [nvk/llm-wiki](https://github.com/nvk/llm-wiki) (`wiki@llm-wiki`). Different project. Plugin name here is `llm-wiki`.

The wiki itself is **not** this repo. It lives in a folder you choose (`wiki_root`) — typically an [Obsidian](https://obsidian.md) vault.

---

## Install

| Agent | Commands |
|---|---|
| **Claude Code** | `claude plugin marketplace add mngaonkar/llm-wiki` then `claude plugin install llm-wiki@llm-wiki` |
| **Grok** | `grok plugin marketplace add mngaonkar/llm-wiki` then `grok plugin install llm-wiki --trust` |
| **Gemini CLI** | `gemini skills install https://github.com/mngaonkar/llm-wiki.git --scope user` |

Forked the repo? Replace `mngaonkar/llm-wiki` with your `owner/repo`.

**Claude, this session only** (no install): `claude --plugin-dir /path/to/llm-wiki`

**claude.ai / Cowork:** zip this folder so `SKILL.md` is at the zip root, enable code execution, **Customize → Skills → +**.

**Manual fallback:** clone into `~/.claude/skills/llm-wiki`, `~/.grok/skills/llm-wiki`, or `~/.gemini/skills/llm-wiki`. Do not also plugin-install the same path — the agent would load the skill twice.

After Gemini install: `/skills reload` then `/skills list`.

---

## Quick start

1. Install (table above).
2. In a new session say: **Start a wiki for &lt;topic&gt;. Store it in &lt;absolute-path&gt;.**
3. Then: **Ingest &lt;URL or file&gt;** or **What does the wiki say about X?**

If you already have an Obsidian vault, skip INIT. Set `wiki_root` to that vault (see [Configure](#configure)) and say **lint the wiki**.

---

## Configure

`wiki_root` is an **absolute** path to the markdown folder (vault root, or a `wiki/` subfolder). On Windows use forward slashes (`C:/Users/you/wiki`).

The agent will **ask** if `config.json` is missing. To set it yourself, copy `config.json.example` to `config.json` **next to the installed `SKILL.md`**:

```json
{
  "wiki_root": "/absolute/path/to/your/wiki"
}
```

Where that file lives:

| How you installed | Put `config.json` here |
|---|---|
| User skill (manual copy) | `~/.claude/skills/llm-wiki/` (or `~/.grok/skills/…`, `~/.gemini/skills/…`) |
| Claude / Grok plugin | Next to `SKILL.md` inside the installed plugin directory (`claude plugin details llm-wiki` / `grok plugin details llm-wiki` prints the path) |
| Gemini | The directory `gemini skills install` created (`/skills list` shows it) |

Do not commit someone else’s `config.json`. This repo gitignores it.

Empty folder → INIT as in Quick start. Existing vault → do **not** INIT; lint or ingest. The skill matches filenames already in the vault (Title Case with spaces vs `kebab-case`).

---

## Use

| You say | What happens |
|---|---|
| Start a wiki / create a wiki | INIT |
| Ingest this / ingest my GitHub / blog / LinkedIn | INGEST (catalog sources get one index page, not a page per repo) |
| What does the wiki say about X / tell me about myself | QUERY |
| Lint the wiki / wiki status | LINT / STATUS |
| Update the plan to cover X | UPDATE an existing plan page |

The agent only writes wiki pages **inside** `wiki_root`. Every write is logged in `wiki_root/log.md`.

---

## Obsidian

The agent writes ordinary `.md` files. Obsidian is the browser: graph, backlinks, search, and Properties. Keep this **skill** in the agent/plugin directory; keep the **wiki** in the vault. Do not copy this repo into the vault as notes.

1. Vault folder on disk → that path is `wiki_root`.
2. Set `config.json` as above.
3. Open the folder as a vault. Agent writes show up immediately (filesystem watch). No Obsidian plugin required.

Use a **subfolder** (`…/Obsidian/wiki`) if you do not want `index.md` / `log.md` mixed with daily notes.

| File | Role in Obsidian |
|---|---|
| `index.md` | Map of content. Pin it or add it to a home note. |
| `log.md` | Append-only ingest history. Noisy on the graph; leave it, or exclude it from graph view. |
| `schema.md` | Naming and entity types. Optional if the vault predates INIT. |
| Topic pages | One note per person / project / concept. YAML (`type`, `aliases`) shows as Properties. |

Pages use `[[WikiLinks]]`. Obsidian resolves by **filename stem**, case-insensitive. **Space ≠ hyphen:** `[[GLM 5.2]]` does not find `glm-5.2.md` unless that file has `aliases: ["GLM 5.2"]`. The skill sets `aliases` to the title used in `index.md`.

Sources **outside** the vault are `file:///` links, not wikilinks. URLs stay markdown links.

Lint indexes every `.md` under `wiki_root` except `index.md` / `log.md` / `schema.md`. If the whole vault is `wiki_root`, daily notes land under Miscellaneous unless you point `wiki_root` at a wiki subfolder.

Sync (Obsidian Sync, iCloud, Git, Syncthing) is fine — the wiki is just files. Avoid two agents writing the same vault at once. The agent must not edit `.obsidian/`.

---

## In this repo

```
SKILL.md                      # when to run, how to write the wiki
config.json.example           # copy to config.json next to SKILL.md
references/                   # index.md and log.md formats
.claude-plugin/               # Claude Code plugin + marketplace
.grok-plugin/                 # Grok marketplace
LICENSE                       # MIT
```

---

## Requirements

- Claude Code, Grok, or Gemini CLI (anything that loads `SKILL.md` folders).
- Local file read/write at `wiki_root`.
- Optional: [Obsidian](https://obsidian.md) to browse the vault. Not required for ingest or query.

---

## Credit

**Andrej Karpathy** originated this approach: the LLM maintains a persistent wiki of markdown pages (ingest → pages → query → lint), rather than one-shot RAG over a pile of files.

- Gist: [https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)

This repository implements that idea as a coding-agent skill. It is not affiliated with Karpathy.

MIT license. See [LICENSE](LICENSE).
