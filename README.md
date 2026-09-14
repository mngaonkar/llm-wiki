# llm-wiki

Persistent markdown wiki for coding agents. You curate sources; the agent ingests them into cross-linked pages, answers from the wiki, and keeps an append-only log.

**Credit:** The idea is [Andrej Karpathy](https://karpathy.ai)’s — [LLM Wiki: a personal, compounding knowledge base](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). This skill is an Agent Skills (`SKILL.md`) packaging of that pattern for Grok, Claude Code, and Gemini CLI.

```
llm-wiki/
  SKILL.md                 # required — when to run, how to write
  config.json              # you create this — wiki_root only
  config.json.example
  README.md
  references/
    index.md               # index.md format
    log.md                 # log.md format
```

The wiki itself is **not** this folder. It lives at `wiki_root` in `config.json` (any directory of markdown; Obsidian vaults work).

---

## Install

Copy this **directory** (not only `SKILL.md`). The agent needs `references/` and will write `config.json` next to `SKILL.md`.

Assume the skill folder you are copying is `./llm-wiki` (clone, download, or this path).

### Grok

User-wide (all projects):

```bash
mkdir -p ~/.grok/skills/llm-wiki
cp SKILL.md config.json.example README.md ~/.grok/skills/llm-wiki/
cp -R references ~/.grok/skills/llm-wiki/
cp ~/.grok/skills/llm-wiki/config.json.example ~/.grok/skills/llm-wiki/config.json
```

This repo only:

```bash
mkdir -p .grok/skills/llm-wiki
cp SKILL.md config.json.example README.md .grok/skills/llm-wiki/
cp -R references .grok/skills/llm-wiki/
cp .grok/skills/llm-wiki/config.json.example .grok/skills/llm-wiki/config.json
```

Grok also scans `~/.claude/skills/` by default. If you already installed for Claude Code, you do not need a second copy unless you want Grok-only overrides.

Then set `wiki_root` (see [Configure](#configure)). In a Grok session: `/skills` to confirm `llm-wiki`, or say “start a wiki”. Skills on disk reload without a restart.

Windows: `%USERPROFILE%\.grok\skills\llm-wiki\`.

### Claude Code

User-wide:

```bash
mkdir -p ~/.claude/skills/llm-wiki
cp SKILL.md config.json.example README.md ~/.claude/skills/llm-wiki/
cp -R references ~/.claude/skills/llm-wiki/
cp ~/.claude/skills/llm-wiki/config.json.example ~/.claude/skills/llm-wiki/config.json
```

This repo only (commit with the project):

```bash
mkdir -p .claude/skills/llm-wiki
cp SKILL.md config.json.example README.md .claude/skills/llm-wiki/
cp -R references .claude/skills/llm-wiki/
cp .claude/skills/llm-wiki/config.json.example .claude/skills/llm-wiki/config.json
```

Set `wiki_root`, then start a session. Folder name `llm-wiki` is the skill name. Say “ingest this” or “query the wiki”; no slash command is required.

**claude.ai / Cowork:** zip the `llm-wiki` folder (`SKILL.md` at the zip root of that folder), enable code execution, then **Customize → Skills → +**.

Windows: `%USERPROFILE%\.claude\skills\llm-wiki\`.

### Gemini CLI

User-wide:

```bash
mkdir -p ~/.gemini/skills/llm-wiki
cp SKILL.md config.json.example README.md ~/.gemini/skills/llm-wiki/
cp -R references ~/.gemini/skills/llm-wiki/
cp ~/.gemini/skills/llm-wiki/config.json.example ~/.gemini/skills/llm-wiki/config.json
```

This workspace only:

```bash
mkdir -p .gemini/skills/llm-wiki
cp SKILL.md config.json.example README.md .gemini/skills/llm-wiki/
cp -R references .gemini/skills/llm-wiki/
cp .gemini/skills/llm-wiki/config.json.example .gemini/skills/llm-wiki/config.json
```

From a checkout of this skill:

```bash
gemini skills install /path/to/llm-wiki --scope user
# or --scope workspace
```

If the files are already on disk:

```
/skills link /path/to/llm-wiki --scope user
```

Set `wiki_root`, then `/skills reload` and `/skills list` to confirm. Gemini also reads `~/.agents/skills/` and `.agents/skills/`.

Windows: `%USERPROFILE%\.gemini\skills\llm-wiki\`.

---

## Configure

Edit `config.json` in the **installed** skill directory (same folder as `SKILL.md`):

```json
{
  "wiki_root": "/absolute/path/to/your/wiki"
}
```

- Must be an **absolute** path. On Windows use forward slashes (`C:/Users/you/wiki`).
- The agent will not guess this. Missing `wiki_root` → it asks, or you run INIT (“start a wiki”).
- Do not copy another person’s `config.json`; it points at their vault.

First run in chat: **“Start a wiki for &lt;topic&gt;. Store it in &lt;path&gt;.”** That writes `schema.md`, `index.md`, and `log.md` under `wiki_root`.

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

## Requirements

- An agent that loads `SKILL.md` folders (Grok, Claude Code, Gemini CLI).
- Ability to read/write local files at `wiki_root`.
- Optional: Obsidian, if you want to browse the markdown vault in a UI.

---

## Credit

**Andrej Karpathy** originated this approach: the LLM maintains a persistent wiki of markdown pages (ingest → pages → query → lint), rather than one-shot RAG over a pile of files.

- Gist: [https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)

This repository is an implementation of that idea as a coding-agent skill. It is not affiliated with Karpathy.
