# log.md Format

`log.md` is the wiki's **append-only** operation history. It lives at `wiki_root/log.md`.

## Rules

- **Append-only.** New entries go at the **TOP** (most recent first), immediately after the `# Wiki Log` heading. Never edit or delete existing entries.
- **Every write to the wiki MUST be logged. No exceptions.** A wiki change without a log entry is an execution bug.
- **Log in the same tool-call batch as the change** — never after the fact, never skipped.
- **Timestamp is mandatory — date AND time (HH:MM, 24-hour).** Never write a date-only header. Get the real current time by running `date "+%Y-%m-%d %H:%M"` before writing; capture it once at the start of the operation and reuse it for that entry. Do not guess or reuse a stale timestamp.

## Entry format

```markdown
## YYYY-MM-DD HH:MM — category — "brief description"
- **Pages created**: (list or omit)
- **Pages updated**: (list or omit)
- **Pages deleted**: (list or omit)
- **Notes**: one-line summary of what changed and why
```

Omit the `Pages created/updated/deleted` lines that don't apply.

## Categories (lowercase, exactly as shown)

| Category | When to use |
|----------|-------------|
| `init` | Wiki initialized for the first time |
| `ingest` | Source ingested; pages created or updated from external content |
| `update` | Page edited, index updated, schema changed, lint fixes applied |
| `delete` | Pages or files removed (requires user confirmation) |

**Brief description** — quoted, max ~8 words, names the source or action (e.g., `"Cohere one-pagers"`, `"task list from meeting notes"`, `"lint fix: dead links"`).

## What is NOT logged

Read-only operations: QUERY (when no page is saved), STATUS, and LINT (when no fixes are applied).

## Example entry

```markdown
## 2026-09-08 14:35 — ingest — "Cohere AMD Instinct one-pagers"
- **Pages created**: customers/cohere/cohere.md, customers/cohere/amd-instinct-agent.md
- **Pages updated**: index.md
- **Notes**: 3-page PDF; three AMD+Cohere joint solutions (Agent/Search/Specialist)
```

