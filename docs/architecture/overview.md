# Architecture overview

> [Русская версия](../ru/architecture/overview.md)

> Part of the vaultify public documentation. English is the source of truth; a
> Russian mirror lives under [docs/ru/architecture/overview.md](../ru/architecture/overview.md).

## The one-paragraph version

vaultify is a local, general-purpose tool (not coding-only) that gives every AI
agent of the household three things: **shared memory** (sessions, decisions,
handoffs across harnesses), **a shared kanban board** over an SQLite registry,
and **meta-management plus full-text-style search across several Obsidian
vaults**. It is TypeScript with a pure core and zero runtime dependencies
(Node stdlib, including `node:sqlite`); everything that touches the world —
files, git, SQLite, processes — lives in thin shells. Distribution is a single
self-contained binary (Node SEA) plus a Tauri v2 desktop companion that runs
the same core in its webview.

## Goal

One local application that serves all of the user's AI agents (zcode, Claude
Code; others later) with:

1. **Shared memory** — sessions, decisions and handoffs in one place, across
   harnesses.
2. **Shared tasks** — one global kanban board over an SQLite registry, moved by
   humans and agents alike.
3. **Storage meta-management and search** — several Obsidian vaults (and any
   markdown folders; the focus is Obsidian): one index, prefix full-text-style
   search, and knowledge distilled by agents straight into the vaults.

### Non-goals (deliberately not built)

- RAG, embeddings, vector stores, cloud, servers (there is no daemon — CLI/MCP
  over stdio only).
- Passwords, encryption, agent-access gates (everything lies in the open — the
  author's call).
- Spaces/domains with separate permissions (a flat world).
- Typed entities (Project/Person/…) — a flat markdown graph instead.
- The canon of knowledge in an app-owned database (the SQLite canon was rejected).

## Principles

- **The canon is the user's data.** Knowledge lives in the vaults; raw agent
  memory is markdown in the meta-repo. The application is an index + search +
  board + transport; everything it owns is the meta-repo.
- **No LLM in the core.** Index, search, board and sync are deterministic code;
  the intelligence is skills at the harness. Zero API keys.
- **Git-native.** The meta-repo is a git repository; vaults follow per-vault
  strategies `own` / `shared` / `none`, and the meta-commands `status`/`sync`
  work over any combination.
- **Zero runtime deps, SEA, Tauri** — the build method of
  [devst](https://github.com/someguy3021/devst-public).
- **Writing into vaults only via the registry's mapping** (layout profiles) —
  never at the whim of a skill.

## Data layers

```
USER VAULTS (the canon)                    META-REPO ~/vaultify/ (what the app owns)
├── ~/Notes/Work    [git: own]             ├── config.*   ← settings: vault registry,
├── ~/Notes/Life    [git: none]            │                 layout maps, git modes
└── ~/Zettel        [git: shared]          ├── tasks.db   ← SQLite task registry (kanban)
     ↑ distillate: the agent writes        ├── memory/    ← raw agent memory (markdown):
       notes per the vault's mapping       │   sessions/  ← session summaries (sources)
       (no index/topics wiki —             │   handoffs/  ← handoff records
       the canon is the vaults)            └── index.db   ← derived vault index
                                                             (rebuildable, not in git)
```

The invariant: **two layers of memory** (the Karpathy LLM Wiki pattern). Raw
material (sessions, handoffs) is immutable and lives in the meta-repo; the
distillate (conclusions, write-ups) is living notes in the vaults, each carrying
provenance — a frontmatter link to its source session (`source: vaultify/S-NNNN`).

## Components

| Component | What | Transport |
|---|---|---|
| **CLI `vaultify`** | `init`, `vault add/list/show/rm/map`, `scan`, `search`, `status`, `sync`, `task …`, `memory …`, `mcp`, `integrations install`, `doctor` | SEA binary |
| **MCP server** | 8 tools: `vault_search`, `memory_save`, `handoff`, `resume`, `task_list`, `task_add`, `task_move`, `vault_status` | stdio; the same binary in `mcp` mode |
| **Skills + hooks** | Distillation, the morning flow, search etiquette; a SessionStart hook feeds the state (board, fresh memory, dirty vaults) into context | adapters: zcode, Claude Code (`dsh`) |
| **Tauri UI** | Board-first: kanban with drag-and-drop, the morning flow; vault management; search | a separate desktop app |

Pure core modules (`src/*.ts`, no `node:*`): argument utilities, markdown
parsing (frontmatter / wikilinks), layout profiles with auto-detection, the
vault registry, the scan/ranking/snippet plan, the kanban core, the memory
formats. Shells (`*-run.ts`): the meta-repo (`~/vaultify/`, `VAULTIFY_HOME`
overrides), the SQLite inverted index, git strategies (`own`/`shared`/`none`
detection, `status`/`sync` with auto-commit and `pull --rebase`), the kanban
database, memory files, the MCP stdio server, integrations install/doctor.

## Flows

- **The morning flow:** "what do I do today?" → a skill reads the board
  (P0/P1, staleness) + fresh memory + vault statuses → proposes 1–3 tasks with
  context.
- **An agent session:** at the end — `memory_save` (session summary →
  `memory/sessions/`) and/or `handoff` (done / left / gotchas →
  `memory/handoffs/`). A different harness picks it up with `resume`.
- **Distillation:** a skill periodically (or on request) reads the raw sessions
  → writes conclusions as notes into the vaults per the layout mapping, with
  provenance.
- **Search:** CLI/MCP → prefix full-text-style query over the derived vault
  index + tasks + memory.
- **Sync:** `vaultify status` (dirty / behind) → `vaultify sync` (commit+pull+push
  per the git strategy of every vault and the meta-repo).

## The v1 readiness criterion

**The morning flow end-to-end, in one breath:** in the morning the agent sees
the board and the memory → proposes a plan → works → at the end of the session a
handoff → the distillate appears in a vault → search finds both the old and the
new. The UI is part of the criterion.

## References

| Reference | What was borrowed |
|---|---|
| **devst** ([public repo](https://github.com/someguy3021/devst-public)) | The build method: TS / zero-deps / SEA / Tauri, `integrations install` + `doctor`, the ADR process, the task registry, documentation discipline |
| [claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian) | "You capture — the AI organizes": inbox→notes, `[[wikilinks]]`, process/organize skills; the no-RAG stance |
| [ai-memory](https://github.com/akitaonrails/ai-memory) | Shared memory across harnesses, the handoff/resume protocol, the install/doctor experience |
| [Graphify](https://github.com/Graphify-Labs/graphify) | The idea of deterministic structure extraction (here: frontmatter/wikilinks instead of ASTs) |
| [LLM Wiki (Karpathy)](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) | Two layers: immutable sources + a living distillate with provenance; search by traversal, not vectors; the wiki compounds |

The key difference from each: vaultify does not build a "second brain of the
application" (as claude-obsidian or LLM Wiki would) — the canon stays in the
user's vaults; and it does not build a database-backed memory (as ai-memory
does) — the raw memory is markdown in a git meta-repo.
