# index.md Format

`index.md` is the wiki's content catalog. It lives at `wiki_root/index.md` and mirrors the folder tree.

## Rules

- **index.md mirrors the folder tree.** Every folder under `wiki_root` is treated the same way — no folder (including `tasks/`) gets special semantics, status grouping, or bespoke ordering.
- Give each top-level folder its own `## Section`, deriving the heading from the folder name (`customers/` → `## Customers`, `tasks/` → `## Tasks`). Nested subfolders can become `### Subsections` if it aids readability.
- List every page under its folder's section. Pages that live directly at `wiki_root` (not in a folder) go under the schema category that fits, or `## Miscellaneous`.
- **Always update index.md when a new page is created.**
- Keep the `_Last updated: YYYY-MM-DD_` line current.

## Entry format

One line per page:

```
- [Title](path/to/file.md) — one-line description
```

- **Title** — the human-readable display name. It should match the page's `aliases:` frontmatter so `[[WikiLinks]]` using this title resolve to the file.
- **path** — relative to `wiki_root`, forward slashes.
- **description** — a short phrase, ideally ≤ ~12 words.

## Skeleton

```markdown
# Wiki Index
_Last updated: YYYY-MM-DD_

## Customers
- [Cohere](customers/cohere/cohere.md) — Enterprise AI company; AMD Instinct joint solutions
- [PubMatic](customers/pubmatic/pubmatic.md) — Ad-tech; LightGBM on MI325X/ROCm

## Technology
- [GLM 5.2](customers/spectro-cloud/glm-5.2.md) — 753B FP8 MoE served via vLLM/ROCm

## Projects
- [LLM Evaluation Framework](projects/llm-eval-framework.md) — Two-stage LLM-as-judge eval

## References
- [Red Hat - Dell - AMD](references/redhat-dell-amd.md) — Lab inventory: BMC/host addresses + access

## Concepts
_No pages yet._

## People
_No pages yet._
```

## Link resolution note

Obsidian resolves `[[WikiLink]]` by **filename basename**, case-insensitive, and treats **space ≠ hyphen**. Files are lowercase-hyphenated (`glm-5.2.md`), but index/body links use Title-Case display names (`[[GLM 5.2]]`). These only resolve if the target page carries an `aliases: ["GLM 5.2"]` frontmatter entry. Always give a new page an alias matching the Title used in the index — otherwise the link is dead. See the topic-page template in SKILL.md.

