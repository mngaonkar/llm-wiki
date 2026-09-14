---
name: llm-wiki
description: "Build and maintain a persistent, structured knowledge base (wiki) in your Obsidian vault or any local directory. The LLM incrementally ingests sources, updates cross-referenced markdown pages, and synthesizes answers — a compounding artifact that grows richer with every source added. Use when user says 'add to wiki', 'ingest this', 'ingest my github/blog/linkedin', 'query the wiki', 'update the wiki', 'what does the wiki say about X', 'tell me about myself', or wants to build a personal knowledge base. Keywords: wiki, knowledge base, ingest, note, obsidian, kb, digest, github, blog."
metadata:
  author: llm-wiki
  version: "1.4.0"
  credit: "Idea: Andrej Karpathy, https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f"
  category: knowledge-management
  tags: ["wiki", "knowledge-base", "obsidian", "notes", "rag"]
compatibility:
  universal: true
---

# LLM Wiki Skill

Build and maintain a **persistent, compounding knowledge base** from raw sources. Unlike one-shot RAG, this skill maintains a structured wiki of markdown pages that accumulates and cross-references knowledge over time.

**Credit — Andrej Karpathy.** The ingest → wiki pages → query → lint loop is his [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) idea. This skill packages that pattern for coding agents; it is not affiliated with Karpathy.

## Core Concept

```
Raw sources  →  [Ingest]  →  Wiki pages  →  [Query]  →  Answers
                                ↑                           |
                            [Lint] ←────────────────────────┘
                         (periodic health check)
```

The wiki is a **persistent artifact** the LLM writes and maintains. The human curates sources and asks questions; the LLM handles every cross-reference, deduplication, and synthesis step.

## Wiki Location & Path Safety (Non-Negotiable)

The wiki root is stored in **`config.json`**, located in this skill's own directory (alongside `SKILL.md`).

```json
{ "wiki_root": "/absolute/path/to/wiki" }
```

See **`config.json.example`** in this skill directory for a sample. On Windows use forward slashes in the path (e.g. `C:/Users/name/wiki`). If `config.json` doesn't exist yet, copy the example and set `wiki_root`, or run INIT.

**Before ANY operation (read or write), you MUST:**
1. Read `config.json` from the skill directory to get `wiki_root`.
2. Treat `wiki_root` as the ONE and only base for the wiki. Every page path in `index.md`, `log.md`, and `[[WikiLink]]` references is relative to `wiki_root`.
3. Resolve every file you intend to read or write to an absolute path by joining `wiki_root` + the relative path.

**The hard rule — never write outside the wiki root:**

> **Every file the skill creates or edits MUST live inside `wiki_root`.**
> Before every Write or Edit, verify the resolved absolute path begins with `wiki_root`. If it does not, STOP — do not write. Report the mismatch to the user instead.

- Never write to a bare relative path (e.g. `customers/foo.md`) — a relative path resolves against the *current working directory*, which may differ from `wiki_root`. Always prepend `wiki_root` to form the absolute path first.
- Beware look-alike sibling directories (e.g. `wiki/` vs a source folder named similarly). The only safe destination is a path that starts with the exact `wiki_root` string from config.
- Source files being ingested may live anywhere (they are read-only inputs). Only the *destination* wiki pages are constrained to `wiki_root`.

**If `config.json` is missing or has no `wiki_root`:** do not guess. Ask the user for the wiki location (or run INIT), then write it to `config.json` before proceeding.

## Wiki Structure

The wiki lives in the directory recorded as `wiki_root` in `config.json` (see Wiki Location & Path Safety above). It contains:

```
wiki/
  index.md          # content catalog, organized by category
  log.md            # append-only chronological record of all operations
  schema.md         # wiki conventions, entity types, naming rules
  <topic>.md        # one page per entity / concept / person / project
  <topic>/          # subdirectory when a topic has many sub-pages
```

### index.md format
`index.md` is the content catalog; it mirrors the folder tree. **Full spec: [references/index.md](references/index.md).** Quick shape:

```markdown
# Wiki Index
_Last updated: YYYY-MM-DD_

## Category A
- [Topic 1](topic1.md) — one-line description
```

### log.md format
`log.md` is **append-only**; new entries go at the TOP. Each entry is `## YYYY-MM-DD HH:MM — category — "brief description"` followed by `Pages created/updated/deleted` + `Notes` bullets. Categories: `init`, `ingest`, `update`, `delete`. **Full spec: [references/log.md](references/log.md).**

### schema.md
Defines conventions for this specific wiki: entity types, naming rules, cross-reference style, what each category covers. Created during `init` and updated as the wiki evolves.

**If `schema.md` is missing** (lint-bootstrapped vault, adopted notes folder): do not block and do not invent a new naming scheme. Follow `index.md` categories and **existing filenames** (spaces vs hyphens, Title Case vs lowercase). Offer to write `schema.md` from those observed conventions; only write it if the user agrees. The hyphenated INIT template applies only to an empty wiki.

### Topic page format
```markdown
---
type: <entity-type>
aliases: ["Human-Readable Title"]
---

# Entity / Topic Name

_Last updated: YYYY-MM-DD_

## Summary
One paragraph synthesis of what this entity/concept is.

## Key Facts
- Bullet list of important facts with sources noted inline [Source: title]

## Relationships
- [[Related Topic 1]] — nature of relationship
- [[Related Topic 2]] — nature of relationship

## Notes / Open Questions
- Contradictions, gaps, or things needing follow-up

## Sources
Ingested YYYY-MM-DD from the [source folder](file:///ABSOLUTE/ENCODED/PATH/):
- [source-file-name](file:///ABSOLUTE/ENCODED/PATH/source-file-name)
```

**Frontmatter is mandatory on every page.** Set `type:` to the entity type and `aliases:` to the Human-Readable Title you use in `index.md` and `[[WikiLinks]]`. Obsidian resolves links by **filename basename**, case-insensitive, treating **space ≠ hyphen** — so `[[GLM 5.2]]` will NOT find `glm-5.2.md` unless that file has `aliases: ["GLM 5.2"]`. Without the alias the link is dead. (Single-word Title == filename, e.g. `[[Cohere]]` → `cohere.md`, needs no alias.)

**Sources must be navigable links.** When a source is a file/folder on disk, cite it as a clickable Markdown link, not plain text:
- Use an **absolute `file:///` URI** (e.g. `file:///C:/Users/.../folder/file.md`). Obsidian wikilinks only resolve *inside* the vault, so sources that live outside `wiki_root` (a common case) are unreachable as wikilinks — `file:///` URIs open via the OS instead.
- **Percent-encode spaces** in the path as `%20` (and other reserved chars as needed). Windows paths use forward slashes after `file:///`.
- Link the **source folder** once, then each **individual file** on its own line, so the user can open either.
- If a source is a URL, just link the URL directly.

---

## Operations

### INIT — Initialize a new wiki

When the user asks to start, create, or initialize a wiki:

1. Ask for the wiki directory path if not specified (default: `wiki/` in CWD).
2. **Save the location** — write the chosen absolute path to `config.json` (in the skill directory) as `wiki_root`. This is what every future operation reads. Do this before creating any files.
3. Ask what domain/topic this wiki will cover.
4. Create the `wiki_root` directory.
5. Write `schema.md` defining conventions based on the domain.
6. Write `index.md` (empty, with category stubs from the schema).
7. Write `log.md` with the init entry.
8. **LOG** — write `INIT` entry to log.md listing files created.
9. Report the structure and the saved `wiki_root` to the user.

**schema.md starter template:**
```markdown
# Wiki Schema

## Domain
<what this wiki is about>

## Entity Types
- **Person**: researchers, authors, practitioners
- **Concept**: ideas, techniques, methods
- **Project**: ongoing efforts, products, papers
- **Organization**: companies, labs, groups
- **Event**: conferences, releases, incidents

## Naming Conventions
- File names: lowercase, hyphens (e.g., `attention-mechanism.md`) — **INIT default only**. If the wiki already has pages, match those names instead.
- Cross-references: use [[WikiLink]] style
- Dates: ISO 8601 (YYYY-MM-DD)

## Categories (for index.md)
<list the top-level categories relevant to this domain>

## Source Types Accepted
- Articles, blog posts, papers (paste text or URL)
- Public profiles and catalogs (GitHub, GitLab, blogs, LinkedIn) — see Catalog ingest
- Meeting notes, transcripts
- Books (chapters/excerpts)
- Data files (CSV, JSON — summarized)
```

---

### INGEST — Add a new source to the wiki

Triggered when user says: "add this to the wiki", "ingest this", "read this and update the wiki", or pastes content directly.

**Steps:**
1. Read the source (text pasted, file path given, or URL fetched with WebFetch). Prefer a machine-readable feed over HTML when the source is a profile or blog (API, RSS). If a fetch truncates, parse the saved file; do not ingest from the truncated view alone.
2. Read `schema.md` (if present) and `index.md` to understand current wiki structure and categories. If `schema.md` is missing, see schema.md fallback above.
3. Identify which existing pages are relevant (search for them with Grep/Glob).
4. Read those pages in full, including the section you will edit.
5. Extract key entities, facts, and relationships from the source. If the source is a **catalog** (many items under one account), use Catalog ingest below — do not mint a page per item.
6. For each entity that gets a page:
   - If a page exists: **merge** into existing headings. Do not add a second `## Writing`, `## Sources`, or other duplicate section. Update the Summary if it is now incomplete.
   - If no page exists: create a new page using the topic page format, matching this wiki's filename convention.
7. Update `index.md` to list any new pages.
8. **LOG** — append an `ingest` entry to log.md (see Logging Rule). List source, pages created, pages updated, and key facts added.
9. Report a summary: pages updated, pages created, key facts added. **If a Plan/Project page is now stale relative to new facts** (it names a gap this source fills, or its public-narrative assumptions are wrong), say so and offer to update it. Do not silently rewrite the plan.

**Extraction checklist:**
- [ ] Who are the key people/entities mentioned?
- [ ] What concepts or techniques are introduced or discussed?
- [ ] What claims does the source make? Are any contradicted by existing wiki content?
- [ ] What relationships between entities are described?
- [ ] What open questions or gaps does this surface?
- [ ] Is this one document, or a catalog of many items?

**Conflict handling:** If a new source contradicts an existing wiki fact, do NOT silently overwrite. Note both claims and their sources under the page's existing conflict heading (`## Contradictions` or `## Notes / Open Questions`). Create `## Contradictions` only if the page has neither.

**Superseded artifacts:** When a new canonical artifact replaces an old public one, mark the old page historical and point at the new page. Do not keep extending the superseded page as current.

**Catalog ingest** (GitHub/GitLab profiles, blogs, LinkedIn, any index of many items):

- Create **one catalog page** for the account or feed.
- Extra pages only for entities that already have wiki identity, or that are clearly flagship (pinned, live product, named in a Plan).
- Sample/fork/gist/archive noise stays as bullets on the catalog. Do not create a page per repo, post, or gist.
- Fetch: use API or RSS when HTML is gated, paginated, or incomplete. Examples — GitHub: `api.github.com/users/{login}` + `repos?per_page=100`; Medium: `medium.com/feed/@user` (rolling ~10-item window; older posts need a direct URL).
- Large JSON/XML: if the fetch tool truncates, run `python`/`jq` on the saved file.

---

### QUERY — Answer a question using the wiki

Triggered when user asks: "what does the wiki say about X", "find X in the wiki", "summarize X from the wiki", "tell me about myself" / "about me" (if a person page exists), or "what do we know about X".

**Steps:**
1. Read `index.md` to identify relevant pages.
2. Search for keyword matches with Grep across wiki pages.
3. Read all relevant pages in full.
4. Synthesize an answer from wiki content only — clearly distinguish wiki-sourced facts from any external knowledge. For identity questions, still wiki-first; put anything the wiki lacks under Gaps.
5. If the answer reveals something valuable not yet in the wiki, offer to file it as a new page ("Want me to save this synthesis as a new wiki page?").
6. **LOG** — if a page is created or edited from the query, append an `update` entry to log.md. Read-only queries are not logged.

**Answer format:**
```
Based on the wiki:

[synthesized answer citing page names]

Sources in wiki: page1.md, page2.md
Gaps: [anything the wiki doesn't cover that would improve this answer]
```

---

### LINT — Health check the wiki

Triggered when user says: "lint the wiki", "check the wiki", "audit the wiki".

**Steps:**
1. List all `.md` files in the vault (excluding meta-files: index.md, log.md, schema.md).
2. Read `index.md` in full.
3. Cross-reference every file against the index.
4. Scan all pages for `[[WikiLink]]` references and verify each resolves to an existing file.
5. Report all findings as a checklist (see categories below).
6. **Auto-fix without asking** for the mechanical issues: index gaps, orphaned pages, and dead path links in index.md.
7. **Update index.md** — add every unindexed page under a section that mirrors its folder / category (see index rules below). No folder is special-cased.
8. **LOG** — if any fixes were applied, append an `update` entry to log.md listing every file changed. If lint was read-only (no fixes), do not log.

**Check for:**
- **Index gaps**: `.md` files that exist but have no entry in `index.md` → auto-fix by adding to index
- **Orphaned pages**: pages not referenced anywhere in `index.md` or other pages → auto-fix by adding to index
- **Dead path links in index.md**: markdown links `[text](path.md)` where the file doesn't exist → flag for human review (do not auto-delete)
- **Dead WikiLinks**: `[[WikiLink]]` references with no matching file → flag as stubs needing ingest
- **Stale pages**: knowledge pages with no `## Sources` section → flag for human review
- **Contradictions**: pages with a `Contradictions` subsection, or conflict notes under `## Notes / Open Questions` → flag for human review
- **Missing schema.md**: ℹ️ informational — offer to write it from observed filenames and index categories; do not auto-write

**index.md update rules during lint:** index.md mirrors the folder tree — each top-level folder gets its own `## Section`, every page listed under it, no folder special-cased. Full rules and entry format: **[references/index.md](references/index.md)**.

**Dead-WikiLink remedy:** before flagging a `[[WikiLink]]` as a missing stub, check whether the target already exists under a hyphenated filename (e.g. `[[GLM 5.2]]` vs `glm-5.2.md`). If it does, the fix is to add `aliases: ["GLM 5.2"]` to that existing page — not to create a stub. Only stub links that have no existing target at all.

Report findings as a checklist: ✅ auto-fixed, ⚠️ needs human review, ℹ️ informational.

---

### STATUS — Show wiki overview

Triggered when user says: "wiki status", "what's in the wiki", "show me the wiki".

Read `index.md` and `log.md`, then report:
- Total page count
- Pages by category
- Last 5 log entries (recent activity)
- Any lint warnings (run a quick lint pass)

---

## Logging Rule (Non-Negotiable)

> **Every write to the wiki MUST be logged in log.md. No exceptions.**
> Log in the **same turn**, before the user-facing report. If writes span multiple tool batches, put the log entry in the last write batch of that operation.
> A wiki change without a log entry is a bug in execution.

- **What must be logged:** `init` (wiki initialized), `ingest` (source ingested, pages created/updated), `update` (page edited, index updated, lint fixed, query result saved), `delete` (file removed).
- **What is NOT logged:** read-only ops — QUERY (no page saved), STATUS, LINT (no fixes applied).
- **Timestamp is mandatory — date AND time (HH:MM, 24-hour).** Run `date "+%Y-%m-%d %H:%M"` before writing; never guess or reuse a stale timestamp. New entries go at the TOP of log.md.

**Full entry format and examples: [references/log.md](references/log.md).**

---

## Execution Rules

1. **Stay inside `wiki_root` — every write, no exceptions.** Read `config.json` first; resolve each destination to an absolute path under `wiki_root` and verify it begins with `wiki_root` before writing. Never write to a bare relative path or outside the root. See Wiki Location & Path Safety above.
2. **Log every write — same turn, no exceptions.** Format and categories: [references/log.md](references/log.md). Get real time via `date "+%Y-%m-%d %H:%M"`. If writes span batches, log in the last write batch, still before the user-facing report.
3. **Never overwrite wiki facts silently.** Contradictions get flagged, not resolved by assumption. Use the page's existing conflict heading.
4. **Always update index.md** when a new page is created.
5. **Read existing pages before writing.** Never create a page without checking if one already exists for that entity. When editing, merge into existing headings; do not duplicate sections.
6. **Keep page summaries current.** When adding facts to a page, revise the Summary paragraph if it's now incomplete.
7. **Source attribution.** Every fact should be traceable to a source in the Sources section.
8. **Ask before bulk deletes.** Never delete wiki pages without explicit user confirmation.
9. **Match this wiki's filenames.** INIT template is lowercase-hyphens. An existing vault's Title Case / spaces / folders win.

## Triggering This Skill

| User says | Operation |
|-----------|-----------|
| "start a wiki", "create a wiki for X" | INIT |
| "add this to the wiki", "ingest [URL/file/text]" | INGEST |
| "ingest my github / blog / linkedin" | INGEST (catalog ingest) |
| "what does the wiki say about X" | QUERY |
| "find X in the wiki" | QUERY |
| "tell me about myself", "what do we know about X" | QUERY (wiki-first; Gaps for anything missing) |
| "lint the wiki", "audit the wiki" | LINT |
| "wiki status", "what's in the wiki" | STATUS |
| "update the wiki with this" | INGEST |
| "update the plan to cover X" (wiki already has the facts) | `update` the Plan page; do not re-ingest |
| "create task list", "action items", pastes meeting todos | INGEST (pages land in the relevant folder, e.g. `tasks/`, and are indexed like any other) |

## Example Session

```
User: Start a wiki for my AI research notes. Store it in ~/notes/ai-wiki.

Claude: [INIT] Creates ~/notes/ai-wiki/ with schema.md, index.md, log.md
        Schema defines: Papers, Concepts, People, Labs as entity types
        Reports structure.

User: [pastes paper abstract about attention mechanisms]

Claude: [INGEST] Creates attention-mechanism.md, transformer.md
        Updates index.md under "Concepts"
        Logs the operation.

User: Ingest https://github.com/example

Claude: [INGEST-CATALOG] One GitHub catalog page; flagship repos only;
        merges into the person page; offers to update a stale Plan.

User: What does the wiki say about transformers?

Claude: [QUERY] Reads transformer.md, attention-mechanism.md
        Synthesizes answer, notes gaps.

User: Lint the wiki.

Claude: [LINT] Reports: 0 orphans, 1 dead link ([[BERT]] not yet a page),
        suggests creating a stub.
```

