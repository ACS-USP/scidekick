# CLI and Integration Modes

The ways to drive `sk`: interactive TUI, one-shot prompts, resumable sessions, editor integration over ACP, and stdio RPC for embedding. Subcommands are registered lazily in `packages/coding-agent/src/cli-commands.ts`.

## Interactive and one-shot

```bash
sk                                   # TUI
sk "Review the uncommitted changes"  # TUI with an initial message
sk -p "Explain this repository"      # one-shot, prints and exits
sk --model opus --smol haiku --slow opus --plan gemini-pro
```

The TUI renders tool calls as cards, previews edits before they land, and streams shell/eval output. Core tools come from `packages/coding-agent/src/tools/` (`read`, `find`, `search`, `ast_grep`, `ast_edit`, `edit`, `write`, `bash`, `eval`, `lsp`, `debug`, `browser`, `web_search`, `task`, memory tools, …); per-tool docs are in `docs/tools/`.

## Sessions

```bash
sk --continue "What did we decide last time?"
sk --resume                     # picker
sk --resume <session-id-prefix>
sk --export ~/.sk/agent/sessions/<path>/session.jsonl
```

Sessions are JSONL transcripts under the agent dir (see [Configuration and Identity](Configuration-and-Identity)).

## ACP editor integration

`sk acp` speaks the Agent Client Protocol over JSON-RPC on stdio, so editors route file writes, terminal output and permission prompts through their own UI.

`sk acp-install` (`commands/acp-install.ts`) writes the agent entry into the editor's settings for you:

| Command | Settings file |
|---|---|
| `sk acp-install zed` (default target) | `~/.config/zed/settings.json` (`agent_servers`) |
| `sk acp-install vscode` | VS Code user `settings.json` (`acp.agents`, used by an ACP client extension) |
| `sk acp-install vscode -w` / `--workspace` | `<cwd>/.vscode/settings.json` |

It resolves the absolute `sk` path, merges the entry, and first copies the existing file to `<file>.<timestamp>.bak`. Editors launched from Finder/Dock may not inherit shell env vars — put provider keys in the entry's `env`.

## RPC mode

```bash
sk --mode rpc --no-session
```

NDJSON frames over stdio for non-Node hosts. `python/robomp/` is a real consumer: a self-hosted GitHub triage bot that runs the agent in RPC mode inside per-issue git worktrees, with a separate `gh-proxy` container holding the GitHub token. Spec: `docs/rpc.md`.

## Subcommand inventory

| Subcommand | Purpose |
|---|---|
| `wiki`, `journal` | Research memory — [Research Wiki and Journal](Research-Wiki-and-Journal) |
| `install-skills` (`skills`) | Scientific skills — [Scientific Skills and Model-Tier Guard](Scientific-Skills-and-Model-Tier-Guard) |
| `acp`, `acp-install` | Editor integration |
| `search` (`q`) | Web search from the shell |
| `commit` | Model-written commit messages |
| `config`, `setup`, `doctor` | Configuration, first-run setup, environment diagnostics (`doctor --json` runs in CI) |
| `plugin`, `install` | Plugins/extensions and marketplace installs |
| `agents` | Subagent definitions |
| `worktree` (`wt`) | Git worktree helpers |
| `ssh`, `shell`, `read`, `grep` | Remote hosts and standalone tool front-ends |
| `stats` | Local usage/observability dashboard (`packages/stats`) |
| `update` | Self-update of release binaries |
| `completions` | Shell completion scripts (`zsh`, `bash`, `fish`) |
| `launch`, `auth-broker`, `auth-gateway`, `tiny-models`, `grievances` | Inherited utilities from Oh My Pi |

Not implemented yet (roadmap in `docs/scidekick-science-surface.md`): `experiment`, `run`, `loop`, `pipeline`, `team`.

## Related pages

- [Configuration and Identity](Configuration-and-Identity)
- [Research Wiki and Journal](Research-Wiki-and-Journal)
- [Troubleshooting](Troubleshooting)
