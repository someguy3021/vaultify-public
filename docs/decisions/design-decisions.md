# Design decisions (v1)

> [Русская версия](../ru/decisions/design-decisions.md)

> Part of the vaultify public documentation. English is the source of truth; a
> Russian mirror lives under
> [docs/ru/decisions/design-decisions.md](../ru/decisions/design-decisions.md).
>
> All seventeen decisions were accepted at the design phase and realized in v1
> the same day; architectural context:
> [docs/architecture/overview.md](../architecture/overview.md).

## The decision table

| # | Decision | Choice | Rejected alternative |
|---|---|---|---|
| R1 | Toolchain | Like devst: TypeScript, zero runtime deps, Node SEA binary, Tauri UI | — |
| R2 | Transport to agents | CLI (SEA) + MCP over stdio + skills/hooks. **No daemon** | A localhost daemon (as in ai-memory) — unnecessary for v1 |
| R3 | Harnesses in v1 | zcode + Claude Code (installer adapters, `doctor`) | 20+ harnesses at once — too expensive |
| R4 | Intelligence | No LLM in the core; distillation/linking/planning are skills at the harness | LLM/embeddings in the core |
| R5 | Canon of knowledge | **The user's vaults.** The distillate is written by the agent into the vaults per the mapping | An SQLite canon (double truth); an app-owned wiki (rejected: the canon must be single — Obsidian) |
| R6 | Agent memory | Two layers: raw (sessions/handoffs, markdown, meta-repo) → distillate (notes in the vaults, with provenance) | A single layer; SQLite-backed memory |
| R7 | Raw memory format | Markdown in the git meta-repo (human-readable, git history for free) | SQLite rows (as in ai-memory) |
| R8 | Tasks | An SQLite registry (devst-style) with **kanban statuses**: `inbox/doing/waiting/someday/done` | Markdown cards; a minimal set of statuses |
| R9 | Application data | One meta-repo `~/vaultify/`: config, tasks.db, memory/, index.db | Several repos; a wiki inside the meta-repo |
| R10 | Git nativeness | Per-vault strategies: `own` (default) / `shared` (inside someone else's repo) / `none`; meta-commands `status`/`sync` | A hard "1 vault = 1 repo"; a custom sync protocol |
| R11 | Spaces | **A flat world**: one global search and one board; separation by tags and filters | Domain spaces; typed entities |
| R12 | What counts as a "storage" | A vault = any folder with markdown; the focus and polish are on Obsidian (frontmatter, `[[wikilinks]]`; `.obsidian/` and plugins are ignored gently) | Only real vaults with `.obsidian/` |
| R13 | Access and secrets | No gates, no passwords — everything lies in the open | A TTL gate (belongs to another project, not this one) |
| R14 | UI v1 | Board-first: kanban + the morning flow; search in the UI later | Search-first; CLI-only |
| R15 | Build order | Foundation (meta-repo, registry, index, CLI/MCP search) → memory (raw + handoffs + adapters) → UI | Board first; memory first |
| R16 | The v1 criterion | **The morning flow end-to-end** (see the architecture overview) | A core without UI |
| R17 | Writing to a vault | Layout profiles — the write location is determined by the registry | "The skill decides by context" — unverifiable |

## Layout profiles (R17 in detail)

Each vault in the registry carries a **note-type → folder mapping**; the skill
reads it (`vaultify vault show <name> --json`) and writes strictly by the
mapping. The skill chooses the *type* of a note (journal / project / concept /
…) — that is the intelligence; the *location* is deterministic — that is the
core. An unknown type goes to the `fallback` folder.

| Preset | Behavior |
|---|---|
| `strict` | A hard structure by types (in the spirit of claude-obsidian). For vaults where machine order is welcome |
| `flex` (default) | At `vault add` the existing top-level folders are auto-detected (Journal/Diary → journal, Projects → projects, Sources, People, Concepts…, with an RU/EN synonym dictionary) and the map is built from what is there. The vault is never reorganized |
| `single` | Everything from the agent goes into one `Vaultify/` folder |

Maintenance: `vaultify vault map <name>` revises the mapping by hand.

## Rejected alternatives

The decisions above were made against explicit alternatives, each worth naming:

- **An SQLite canon of knowledge.** A structured store as the source of truth
  would create double truth against the Obsidian vaults the user already
  maintains. The canon must be single — and it is the vaults.
- **An application-owned wiki** (a `~/vaultify/wiki/` of distilled notes, as in
  claude-obsidian or LLM Wiki). The same objection: it would build a second
  brain next to the user's first one. Distillate goes into the vaults instead.
- **Spaces / domains with permissions.** Typed spaces and per-domain access
  would add a permission model to a single-user local tool. A flat world: one
  global search and one board, separation by tags and filters.
- **A TTL gate / passwords.** Time-limited access and agent gates belong to a
  different project's threat model. vaultify keeps everything in the open, by
  the author's explicit call.
- **A daemon.** A localhost service (the ai-memory route) would mean a process
  to supervise, ports, a second runtime. CLI (SEA) + MCP over stdio + skills
  covers the same surface with nothing to babysit.
- **RAG / embeddings / vector stores.** Retrieval here is a prefix inverted
  index on `node:sqlite` (deliberately without FTS5) plus link traversal —
  deterministic, rebuildable, zero API keys.
