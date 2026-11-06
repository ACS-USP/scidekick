# Scidekick Wiki

Last updated: 2026-11-06 (source branch: `main`)

## What this project is

Scidekick is a terminal-first AI coding and research assistant maintained by the Agentic Complex Systems group (ACS-USP). It is a direct fork of Oh My Pi (itself built on Mario Zechner's Pi) and keeps the full coding-agent harness (read/search/edit tools, shell, LSP, DAP debugger, browser, subagents, MCP, multi-provider model routing). On top of it, Scidekick adds a research layer that lives inside the repository: a typed research wiki, an append-only journal, a scientific-skills installer and a model-tier guard for scientific work. The CLI is `sk` and the config/workspace root is `.sk`.

- TypeScript on Bun (`bun@1.3.14`), Bun workspaces monorepo, Biome for lint/format, `tsgo` for type checking
- Rust native addon (N-API) for grep, shell/PTY, AST, image and text operations (`crates/`, `packages/natives`)
- Research surfaces: `sk wiki`, `sk journal`, `sk install-skills` / `sk skills`
- Editor integration via the Agent Client Protocol (`sk acp`, `sk acp-install zed|vscode`) and embedding via `--mode rpc`
- Release binaries `sk-<os>-<arch>` built by GitHub Actions; user guide published to GitHub Pages

## Pages

| Page | What it covers |
|---|---|
| [Architecture Overview](Architecture-Overview) | Package layers, fork structure, where the Scidekick code lives, request flow |
| [Research Wiki and Journal](Research-Wiki-and-Journal) | `sk wiki` and `sk journal`: storage layout, entity types, frontmatter, lint rules |
| [Scientific Skills and Model-Tier Guard](Scientific-Skills-and-Model-Tier-Guard) | `sk install-skills`, the SQLite skill registry, the skill validation gate and the tier classifier |
| [CLI and Integration Modes](CLI-and-Integration-Modes) | TUI, one-shot, sessions, ACP editor setup, RPC, subcommand inventory |
| [Configuration and Identity](Configuration-and-Identity) | `.sk` directories, `SK_*` / `PI_*` env vars, model roles, settings keys |
| [Upstream Fork and Vendor Sync](Upstream-Fork-and-Vendor-Sync) | `vendor/oh-my-pi` submodule, `@oh-my-pi/*` versions, merge tooling |
| [Build, CI and Release](Build-CI-and-Release) | Local dev commands, CI job graph, binary release and GitHub Pages site |
| [Troubleshooting](Troubleshooting) | Symptom-first fixes for setup, submodule, tier warnings and wiki/journal errors |
| [Publishing to GitHub Wiki](Publishing-to-GitHub-Wiki) | How this bundle is synced to the GitHub wiki |

## Quick local run

```bash
git clone https://github.com/ACS-USP/scidekick.git
cd scidekick
bun install
export ANTHROPIC_API_KEY=...   # or OPENAI_API_KEY / GEMINI_API_KEY
bun run dev                    # runs packages/coding-agent/src/cli.ts
# optional: link `sk` globally
bun run install:dev && sk --version
```

## Repository map

- `packages/coding-agent/`: the `sk` CLI, TUI, SDK, tools, sessions and all wired Scidekick commands (`src/commands/`, `src/scidekick/`)
- `packages/ai/`: multi-provider LLM client and model/provider registry (`@oh-my-pi/pi-ai`)
- `packages/agent/`: agent runtime with tool calling and state (`@oh-my-pi/pi-agent-core`)
- `packages/tui/`: terminal UI library with differential rendering
- `packages/natives/`: TypeScript side of the Rust N-API addon
- `packages/utils/`: shared utilities, including the product identity and path helpers in `src/dirs.ts`
- `packages/scidekick-guard/`, `packages/scidekick-science/`, `packages/scidekick-skills/`: standalone `@scidekick/*` packages (earlier copies of the guard, wiki backend and installer; not imported by the CLI)
- `packages/stats/`, `packages/mnemopi/`, `packages/hashline/`, `packages/swarm-extension/`, `packages/typescript-edit-benchmark/`: inherited observability, memory, patch-language, swarm and benchmark packages
- `crates/`: Rust crates (`pi-natives`, `pi-shell`, `pi-ast`, `pi-iso`, vendored `brush-core` / `brush-builtins`)
- `python/robomp/`: self-hosted GitHub triage bot driving the agent in RPC mode
- `vendor/oh-my-pi/`: git submodule with the upstream Oh My Pi snapshot used for merges
- `docs/`: user guide, science surface, architecture and inherited harness docs
- `scripts/`: installers, release/build tooling, vendor-sync and session-analysis scripts
- `site/`: static site deployed to GitHub Pages
- `.sk/`: this repository's own Scidekick workspace (themes and a dogfooding research wiki)

## Wiki maintenance policy

When code changes, update these pages in the same PR:

1. New or changed `sk wiki` / `sk journal` behavior, entity type or template -> [Research Wiki and Journal](Research-Wiki-and-Journal)
2. Changes to `install-skills`, `skill-registry.ts` or `model-tier.ts` -> [Scientific Skills and Model-Tier Guard](Scientific-Skills-and-Model-Tier-Guard)
3. New CLI subcommand or mode -> [CLI and Integration Modes](CLI-and-Integration-Modes)
4. New env var, settings key or directory -> [Configuration and Identity](Configuration-and-Identity)
5. Upstream merge or submodule pointer change -> [Upstream Fork and Vendor Sync](Upstream-Fork-and-Vendor-Sync)
6. Workflow, script or release-target change -> [Build, CI and Release](Build-CI-and-Release)
7. New package or moved module -> [Architecture Overview](Architecture-Overview) and the repository map above
