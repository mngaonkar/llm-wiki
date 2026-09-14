# llm-wiki

Persistent markdown wiki for coding agents. You curate sources; the agent ingests them into cross-linked pages, answers from the wiki, and keeps an append-only log.

**Credit:** The idea is [Andrej Karpathy](https://karpathy.ai)’s — [LLM Wiki: a personal, compounding knowledge base](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). This skill is an Agent Skills (`SKILL.md`) packaging of that pattern for Grok, Claude Code, and Gemini CLI.

Not [nvk/llm-wiki](https://github.com/nvk/llm-wiki) (`wiki@llm-wiki`). Different project; plugin name here is `llm-wiki`.

The wiki itself is **not** this repo. It lives at `wiki_root` in a local `config.json` — typically an [Obsidian](https://obsidian.md) vault.

---

## Install

After install, [set `wiki_root`](#configure). One command per agent — no copying folders.

Replace `mngaonkar/llm-wiki` with this repo if you forked it.

### Claude Code

```bash
claude plugin marketplace add mngaonkar/llm-wiki
claude plugin install llm-wiki@llm-wiki
```

This session only, no install:

```bash
claude --plugin-dir /path/to/llm-wiki
```

**claude.ai / Cowork:** zip this repo folder (`SKILL.md` at the zip root of that folder), enable code execution, **Customize → Skills → +**.

### Grok

```bash
grok plugin marketplace add mngaonkar/llm-wiki
grok plugin install llm-wiki --trust
```

Then `/skills` or say “start a wiki”.

### Gemini CLI

```bash
gemini skills install https://github.com/mngaonkar/llm-wiki.git --scope user
```

Workspace only: add `--scope workspace`. Then `/skills reload` and `/skills list`.

### Manual (fallback)

If you cannot use a plugin marketplace, copy the repo into the agent’s skills directory (`~/.claude/skills/llm-wiki`, `~/.grok/skills/llm-wiki`, or `~/.gemini/skills/llm-wiki`). Do not install **both** as a user skill and as a plugin from the same path — the agent would load it twice.

---

## Configure

Edit `config.json` in the **installed** skill directory (same folder as `SKILL.md`):

```json
{
  "wiki_root": "/absolute/path/to/your/wiki"
}
```

- Must be an **absolute** path. On Windows use forward slashes (`C:/Users/you/wiki`).
- For Obsidian, this is the **vault root** (the folder that contains `.obsidian/`), or a subfolder of that vault if you want the wiki isolated from daily notes.
- The agent will not guess this. Missing `wiki_root` → it asks, or you run INIT (“start a wiki”).
- Do not copy another person’s `config.json`; it points at their vault.

Empty directory: **“Start a wiki for &lt;topic&gt;. Store it in &lt;path&gt;.”** That writes `schema.md`, `index.md`, and `log.md` under `wiki_root`.

Existing Obsidian vault: set `wiki_root` to the vault, then **“lint the wiki”** or ingest a source. Do **not** run INIT — it would overlay a new empty catalog. The skill follows filenames already in the vault (Title Case with spaces vs `kebab-case`).

---

## Use

| You say | What happens |
|---|---|
| Start a wiki / create a wiki | INIT |
| Ingest this / ingest my GitHub / blog / LinkedIn | INGEST (catalog sources get one index page, not a page per repo) |
| What does the wiki say about X / tell me about myself | QUERY |
| Lint the wiki / wiki status | LINT / STATUS |
| Update the plan to cover X | UPDATE an existing plan page |

The agent must only write wiki pages **inside** `wiki_root`. Every write is logged in `wiki_root/log.md`.

---

## Obsidian

The agent writes ordinary `.md` files. Obsidian is the browser: graph, backlinks, search, and Properties. Keep the **skill** (`SKILL.md` + `config.json`) in the agent skills directory; keep the **wiki** in the vault. Do not copy this skill folder into the vault as if it were notes.

### Point the skill at the vault

1. In Obsidian: vault folder → that path is `wiki_root`.
2. Edit the installed skill’s `config.json`:

```json
{
  "wiki_root": "/Users/you/Documents/Obsidian"
}
```

3. Open that folder as a vault (or it already is). Agent writes show up immediately; Obsidian watches the filesystem. No plugin required.

Use a **subfolder** (`…/Obsidian/wiki`) if you do not want `index.md` / `log.md` mixed with daily notes. Then `wiki_root` is that subfolder, and only those files are in scope.

### What the vault contains

| File | Role in Obsidian |
|---|---|
| `index.md` | Map of content. Pin it or add it to a home note. |
| `log.md` | Append-only ingest history. Noisy on the graph; leave it, or exclude it from graph view. |
| `schema.md` | Naming and entity types for this vault. Optional if the vault predates INIT. |
| Topic pages | One note per person / project / concept. YAML frontmatter (`type`, `aliases`) shows up as Properties. |

Pages link with `[[WikiLinks]]`. Obsidian resolves by **filename stem**, case-insensitive. **Space ≠ hyphen:** `[[GLM 5.2]]` does not find `glm-5.2.md` unless that file has `aliases: ["GLM 5.2"]`. The skill sets `aliases` to the title used in `index.md` so graph and click-through work.

Sources **outside** the vault (a LinkedIn PDF, a download) are cited as `file:///` links, not wikilinks — Obsidian wikilinks only resolve inside the vault. URLs stay as markdown links.

### Existing vault

- Match whatever naming you already use. Do not rename notes to kebab-case to satisfy the INIT template.
- Human notes (daily notes, templates, scratch) stay. Lint indexes every `.md` under `wiki_root` except `index.md` / `log.md` / `schema.md`. If the whole vault is `wiki_root`, daily notes will appear in `index.md` under Miscellaneous unless you set `wiki_root` to a wiki subfolder.
- Say **“lint the wiki”** after the first connect to catalog what is already there.

### Sync

Any Obsidian sync (official Sync, iCloud, Git, Syncthing) is fine: the wiki is just files. Avoid two agents writing the same vault at once. Do not let the agent edit `.obsidian/` — that directory is outside the skill’s job, even if it sits next to `wiki_root`.

---

## Requirements

- An agent that loads `SKILL.md` folders (Grok, Claude Code, Gemini CLI).
- Ability to read/write local files at `wiki_root`.
- Optional: [Obsidian](https://obsidian.md) to browse the vault (graph, backlinks, Properties). Not required for ingest or query.

---

## Credit

**Andrej Karpathy** originated this approach: the LLM maintains a persistent wiki of markdown pages (ingest → pages → query → lint), rather than one-shot RAG over a pile of files.

- Gist: [https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)

This repository is an implementation of that idea as a coding-agent skill. It is not affiliated with Karpathy.
