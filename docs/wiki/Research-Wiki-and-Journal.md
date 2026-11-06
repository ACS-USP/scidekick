# Research Wiki and Journal

The two research-memory surfaces that are wired into the CLI today. Both store plain Markdown under the current project's `.sk/` directory (`getProjectAgentDir(getProjectDir())`), so research state is committed alongside code. The wiki holds durable, typed knowledge; the journal holds chronological decisions.

## `sk wiki`

Implemented in `packages/coding-agent/src/commands/wiki.ts` on top of `FilesystemWikiBackend` (`packages/coding-agent/src/scidekick/wiki-backend.ts`).

| Action | Usage | Behavior |
|---|---|---|
| `init` | `sk wiki init` | Creates `.sk/wiki/`, subfolders `papers/ runs/ notes/ claims/`, and `index.md` if missing |
| `new` | `sk wiki new <type> "<title>"` | Creates an entry of one of the entity types below from its template |
| `page` | `sk wiki page "<title>"` | Shortcut for a `note` entry |
| `ingest` | `sk wiki ingest <file> ["<title>"]` | Copies a file's text into a `note` entry and records `source_path`; title defaults to the file's basename |
| `list` | `sk wiki list` | Prints `slug<TAB>type<TAB>title` for every entry |
| `query` | `sk wiki query <text>` | Case-insensitive substring match on title and body |
| `show` | `sk wiki show <slug>` | Prints the raw file |
| `lint` | `sk wiki lint <slug>` | Prints `OK: <slug>` or the errors; exits `1` on errors |

### Storage layout

Entries are written **flat** at `.sk/wiki/<slug>.md` — the `papers/`, `runs/`, `notes/` and `claims/` subfolders created by `init` are suggestions for hand-written pages and are not used by `new`/`page`/`ingest`. `list` reads only top-level `*.md` files (excluding `index.md`), so pages placed in the subfolders are invisible to `list`, `query` and `lint`.

Slugs come from `slugify(title)`: lowercase, non-alphanumerics collapsed to `-`, trimmed, truncated to 64 characters, `untitled` as fallback. Saving an existing slug overwrites the body but preserves the original `created` timestamp and refreshes `updated`.

### Entity types and frontmatter

`WikiEntityType` = `paper | hypothesis | experiment | evidence | insight | note | run`. Every entry has `title`, `type`, `created`, `updated`, `tags`. Type-specific defaults (`defaultFrontmatter`):

| Type | Extra fields | Template |
|---|---|---|
| `paper` | `doi`, `authors`, `journal`, `year` | `wiki-schemas/paper.md` |
| `hypothesis` | `status: draft` (`draft/proposed/testing/confirmed/rejected`) | `wiki-schemas/hypothesis.md` |
| `experiment` | `base_protocol` | `wiki-schemas/experiment.md` |
| `evidence` | `source_hypothesis`, `source_experiment`, `strength: moderate` | `wiki-schemas/evidence.md` |
| `insight` | — | `wiki-schemas/insight.md` |
| `note` | `tags: [note]` | inline `# {{title}}` + `## Notes` |
| `run` | `tags: [run]` | inline `# Run: {{title}}` + Summary/Links |

`base_protocol` implements the base-delta pattern borrowed from ELF (Eli's Lab Framework): an experiment page references a base protocol and documents only what changed.

### Lint rules

`FilesystemWikiBackend.lint` checks:

- required: `title`, `type`, `created`
- `paper`: needs `doi` or at least one author
- `hypothesis`: needs `status`
- `experiment`: if `base_protocol` is set, that slug must exist
- `evidence`: needs `source_hypothesis` or `source_experiment`

## `sk journal`

Implemented in `packages/coding-agent/src/commands/journal.ts` on top of `FilesystemJournalBackend` (`packages/coding-agent/src/scidekick/journal-backend.ts`).

| Action | Usage | Behavior |
|---|---|---|
| `init` | `sk journal init` | Creates `.sk/journal/<YYYY>/` and `index.md` |
| `add` | `sk journal add "<text>"` | Appends an entry to today's file |
| `today` | `sk journal today` | Prints today's file |
| `link` | `sk journal link <entry-id> <target>` | Adds `- <target>` under the entry's `Links:` block (idempotent) |

Files are one per local day: `.sk/journal/<YYYY>/<YYYY-MM-DD>.md`. Each entry is a heading `## HH:MM:SS — <id>` where the id is `YYYY-MM-DD-HHMMSS` (local time), followed by the text and an empty `Links:` list. The journal is append-only by design — `add` uses `appendFile` and `link` only inserts link lines. Link targets are free text; the convention is `wiki:<slug>`.

## Example research trail

```bash
sk wiki init && sk journal init
sk wiki new hypothesis "Linear models are enough for Iris"
sk wiki new experiment "Iris baseline classification"
sk journal add "Started Iris baseline: linear vs tree model."
sk journal link 2026-05-31-140455 wiki:iris-baseline-classification
sk wiki lint iris-baseline-classification
```

After every successful `wiki` or `journal` action, the command calls `getScientificCommandTierWarning` and prints a warning on stderr when the configured default model is below the science tier (see [Scientific Skills and Model-Tier Guard](Scientific-Skills-and-Model-Tier-Guard)).

## Related pages

- [Scientific Skills and Model-Tier Guard](Scientific-Skills-and-Model-Tier-Guard)
- [CLI and Integration Modes](CLI-and-Integration-Modes)
- [Configuration and Identity](Configuration-and-Identity)
