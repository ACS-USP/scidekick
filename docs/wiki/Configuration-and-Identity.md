# Configuration and Identity

How Scidekick names itself, where it stores state, and the knobs that change both. The single source of truth is `packages/utils/src/dirs.ts`; everything else (logs, sessions, `agent.db`, project `.sk/`) derives from it.

## Product identity

| Constant (`dirs.ts`) | Default | Override |
|---|---|---|
| `APP_NAME` | `scidekick` | `SK_APP_NAME` |
| `CONFIG_DIR_NAME` | `.sk` | `SK_CONFIG_DIR` |
| `VERSION` | `version` from `packages/utils/package.json` | `SK_VERSION` |
| `MIN_BUN_VERSION` | from `engines.bun` | — |

The executable is `sk` (the `bin` entry in `packages/coding-agent/package.json`); `APP_NAME` is the display name and is also used in log file names (`scidekick.<date>.log`) and XDG folder names. Identity is done through these defaults rather than renaming packages so that upstream Oh My Pi code merges cleanly (see [Upstream Fork and Vendor Sync](Upstream-Fork-and-Vendor-Sync)).

## Directories

| Directory | Default path | Contents |
|---|---|---|
| Config root | `~/.sk/` | global config, `.env` |
| Agent dir | `~/.sk/agent/` | sessions, `models.yml`, `agent.db` (skill registry and other state), user-level `skills/` |
| Project dir | `<cwd>/.sk/` | `wiki/`, `journal/`, project `skills/`, project settings |

Resolution rules:

- `getConfigDirName()` honors `SK_CONFIG_DIR`, then `PI_CONFIG_DIR`, then `.sk`.
- `PI_CODING_AGENT_DIR` overrides the agent directory entirely.
- On Linux/macOS, for the default agent dir only, if `$XDG_DATA_HOME/scidekick`, `$XDG_STATE_HOME/scidekick` or `$XDG_CACHE_HOME/scidekick` already exists, that category is redirected there (the `agent/` prefix is flattened). The existence check is the switch: nothing is migrated automatically. The header comment in `dirs.ts` refers to an `sk config migrate` step, but no such subcommand is registered in the current CLI, so moving data into the XDG folders is a manual step.

Inherited docs under `docs/` (for example `docs/environment-variables.md`, `docs/models.md`) still describe the upstream defaults `~/.omp/…` and `PI_CONFIG_DIR` defaulting to `.omp`. In Scidekick read those as `~/.sk/…`.

## Model providers and roles

Provider keys come from the environment (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GEMINI_API_KEY`, …) or `.env` files in the config root and agent dir. Custom providers and local OpenAI-compatible servers (Ollama, LM Studio, llama.cpp, vLLM, LiteLLM) are declared in `~/.sk/agent/models.yml`; the format is documented in `docs/models.md`.

Model roles (`config/model-registry.ts`) let one session use different models for different work:

| Role | Used for | CLI flag |
|---|---|---|
| `default` | normal turns; also the model checked by the science tier guard | `--model` |
| `smol` | lightweight background work, cheap subagent fan-out | `--smol` |
| `slow` | deeper reasoning | `--slow` |
| `plan` | architecture and planning | `--plan` |
| `commit` | commit messages and changelog work | — |

## Scidekick settings keys

| Key | Values | Default | Page |
|---|---|---|---|
| `minScienceModelTier` | `haiku` / `sonnet` / `opus` | `sonnet` | [Scientific Skills and Model-Tier Guard](Scientific-Skills-and-Model-Tier-Guard) |
| `scienceTierEnforcement` | `warn` / `block` | `warn` | [Scientific Skills and Model-Tier Guard](Scientific-Skills-and-Model-Tier-Guard) |
| `theme.dark` / `theme.light` | theme names | `titanium` / `light` | `docs/scidekick-tui-visual-identity.md` (visual identity spec) |

Both science keys are defined in `packages/coding-agent/src/config/settings-schema.ts` and appear on the Model tab of the settings UI.

## Related pages

- [CLI and Integration Modes](CLI-and-Integration-Modes)
- [Scientific Skills and Model-Tier Guard](Scientific-Skills-and-Model-Tier-Guard)
- [Troubleshooting](Troubleshooting)
