# CLI & MCP reference

> [Русская версия](ru/cli-reference.md)

> Part of the vaultify public documentation. English is the source of truth; a
> Russian mirror lives under [docs/ru/cli-reference.md](ru/cli-reference.md).
>
> This page mirrors the CLI's own `help` output and the MCP `tools/list` —
> nothing here is invented. Note that the CLI speaks **Russian**: its output and
> some value vocabularies (marked below) are Russian by design.

## Usage

```
vaultify init
vaultify where
vaultify remote [<url>]
vaultify deploy <dir>
vaultify vault add <path> [--name N] [--git own|shared|none] [--layout strict|flex|single]
vaultify vault list [--json] | show <name> [--json] | rm <name> | set <name> [--path <new>] [--git M] [--name <new>]
vaultify vault map <name> [--preset P] [--set type=folder]...
vaultify scan [--vault name] [--json]
vaultify vault target <name> --type journal|projects|concepts|people|sources [--json]
vaultify search <query> [--vault name] [--limit N] [--json]
vaultify status [--json]
vaultify sync [--dry]
vaultify task new "title" [--kind работа|вопрос] [--urgency N] [--link path] [--source S] [--body S]
vaultify task board [--json] | show VT-N | move VT-N <status> [--note S]
vaultify task set VT-N [--title S] [--body S] [--urgency N] [--link P] [--kind K]
vaultify task rm VT-N --yes
vaultify memory save "text" [--topic S] [--harness S]
vaultify memory handoff [--done S]... [--left S]... [--gotchas S]... [--next S] [--topic S] [--harness S]
vaultify memory resume [H-NNN] | close H-NNN | list
vaultify mcp
vaultify integrations install <zcode|dsh>
vaultify doctor
vaultify help
```

## Commands

### Setup

- **`vaultify init`** — create the data home `<install>/home/` (the `VAULTIFY_HOME`
  environment variable overrides the location): settings, the vault registry,
  `memory/`, a git repository.
- **`vaultify where`** — print the deployment folder (artifacts) and the data home.
- **`vaultify remote <url>`** — bind (or show) the git remote of the data home, so
  `vaultify sync` backs up and transfers the board, memory and settings.
- **`vaultify vault target <name> --type T`** — print the resolved target folder for a
  note type (layout mapping, fallback included). Skills write where this command says —
  never inventing a path.
- **`vaultify vault set <name>`** — rename (`--name`), move (`--path`) or change the
  git strategy (`--git`) of a vault without hand-editing the config; paths under the
  user profile are stored in `~/…` form, so the config survives machine moves.
- **`vaultify deploy <dir>`** — deploy the artifacts (`vaultify.exe`, the UI exe when
  built) into one folder; the data home then lives at `<dir>/home`. Moving the folder
  moves everything; after a move re-run `integrations install` from the new location
  (hook paths are absolute by design).
- **`vaultify doctor`** — environment diagnostics: node, git, the meta-repo,
  index freshness, the MCP wiring.

### Vaults

- **`vault add <path>`** — register a markdown folder as a vault. `--name` sets
  the registry name (derived from the folder name by default); `--git` picks the
  sync strategy `own | shared | none` (auto-detected by default); `--layout`
  picks the layout preset `strict | flex | single` (`flex` is the default and
  auto-detects the existing folder structure).
- **`vault list`** — the registry: name, path, git mode, layout preset.
- **`vault show <name>`** — one vault's details; with `--json` the full entry
  including the layout mapping (this is what a distilling skill reads).
- **`vault rm <name>`** — remove a vault from the registry (files are not touched).
- **`vault map <name>`** — revise the note-type → folder mapping by hand:
  `--preset` switches the preset, `--set type=folder` sets individual mappings.

### Index & search

- **`vaultify scan`** — rebuild `index.db` (a derived, rebuildable index, kept
  out of git) over all registered vaults; `--vault` limits the scan to one vault.
- **`vaultify search <query>`** — prefix full-text-style search over title,
  tags and body across the vaults, plus tasks and memory; returns path, title,
  snippet. `--vault` limits the scope, `--limit` caps the number of results.

### Status & sync

- **`vaultify status`** — the summary: vaults (git state: dirty, ahead/behind),
  the task board, memory counts.
- **`vaultify sync`** — commit + pull --rebase + push for every git-mode store
  (each vault per its `own/shared/none` strategy, and the meta-repo);
  `--dry` shows what would happen without touching anything.

### Task board (kanban)

Statuses: `inbox / doing / waiting / someday / done`. Task ids look like `VT-1`,
`VT-2`, …

- **`task new "title"`** — create a task in the inbox. `--kind` sets the kind —
  the accepted values are the Russian words `работа` (work) or `вопрос`
  (question); `--urgency` sets a P0–P3 priority; `--link` attaches a path to a
  note/vault; `--source` records where the task came from; `--body` adds text.
- **`task board`** — render the kanban board.
- **`task show VT-N`** — one task's card with its transition history.
- **`task move VT-N <status>`** — move a task across the kanban; `--note`
  records the reason for the transition (expected for `waiting`).
- **`task set VT-N`** — edit fields: `--title`, `--body`, `--urgency`, `--link`,
  `--kind`.
- **`task rm VT-N --yes`** — remove a task (the flag confirms).

### Memory

The raw layer lives in the meta-repo as markdown:
`memory/sessions/S-NNNN.md` (immutable session summaries) and
`memory/handoffs/H-NNNN.md` (structured handoffs).

- **`memory save "text"`** — save a raw session note: a valuable conclusion or
  decision. `--topic` names the theme/project, `--harness` records which harness
  wrote it (zcode, claude-code, …).
- **`memory handoff`** — write a structured handoff for the next session or the
  next harness: `--done` (repeatable), `--left` (repeatable), `--gotchas`
  (repeatable), `--next` (where to continue), plus `--topic` and `--harness`.
- **`memory resume [H-NNN]`** — pick up a handoff: the latest open one, or a
  specific `H-NNN` by id.
- **`memory close H-NNN`** — mark a handoff as done.
- **`memory list`** — the recorded sessions and handoffs.

### Agents

- **`vaultify mcp`** — start the MCP server (stdio JSON-RPC); the same binary
  in its MCP mode. See the tool list below.
- **`vaultify integrations install <zcode|dsh>`** — install the integration
  into a harness (see below).
- **`vaultify help`** — the built-in help (in Russian).

## Harness integrations

`vaultify integrations install zcode` (or `dsh`) installs, from canonical
templates with machine tokens substituted:

- the **skill** — `vaultify` (the two-layer memory etiquette, distillation by
  layout mapping, search and board usage);
- the **slash-commands** (zcode target) — `/vt-today`, `/vt-remember`,
  `/vt-distill`;
- the **SessionStart hook** — feeds the board, the freshest memory and the dirty
  vaults into the session context; the CLI path is baked in as the hook default;
- the **config merge** (zcode target) — the hook is added to the `SessionStart`
  group and the MCP server is registered; a backup of the harness config is
  written first (`config.json.bak-vaultify`), foreign entries are not touched;
- a **marker file** (`vaultify.json`) in the harness directory, so `doctor` and
  reinstall can find the wiring.

Paths can be redirected for testing or non-standard setups: `VAULTIFY_HOME`
(the meta-repo), and for the installer `VAULTIFY_USER_HOME` (the user home) and
`VAULTIFY_ZCODE` / `VAULTIFY_DSH` (the harness directories).

`vaultify doctor` verifies the result: node, git, the meta-repo, the marker,
the MCP registration.

## MCP tools (8)

The MCP server (`vaultify mcp`) exposes eight read/write tools over stdio
JSON-RPC:

| Tool | What it does |
|---|---|
| `vault_search` | Full-text-style search across all connected Obsidian vaults (notes only); returns path, title, snippet. Params: `query` (required), `vault` (limit to one vault), `limit` (default 5) |
| `memory_save` | Save a valuable conclusion/decision of the current session into the raw memory layer; write compressed conclusions, not a log. Params: `text` (required), `topic`, `harness` |
| `handoff` | Write a handoff: what is done, what is left, the gotchas, where to continue; another harness picks it up via `resume`. Params: `done`, `left`, `gotchas` (string arrays), `next` (required), `topic`, `harness` |
| `resume` | Pick up the latest open handoff (or a specific `H-NNN`) and return its text. Param: `id` (optional) |
| `task_list` | The kanban board: all open tasks plus the recently closed ones |
| `task_add` | Create a task (inbox). Params: `title` (required), `kind` (`работа`/`вопрос`), `urgency`, `link`, `source` |
| `task_move` | Move a task across the kanban: `inbox\|doing\|waiting\|someday\|done`. Params: `id`, `status` (required), `note` (the reason, for `waiting`) |
| `vault_status` | The summary: git state of the vaults and the meta-repo, board and memory counters |
