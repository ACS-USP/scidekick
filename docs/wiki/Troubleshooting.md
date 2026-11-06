# Troubleshooting

Symptom-first. Each section names the owning page and file.

## `git submodule update --init` fails with `not our ref b6f9b29…`

**Cause.** The main repo pins `vendor/oh-my-pi` to `b6f9b298318bc54eda716fd4dfdee7222c3eb9fe` (set in `aef927377`, the 15.8.2 fast-forward), but that commit was never pushed to `felippe-alves/scidekick` or `ACS-USP/scidekick`. Git then leaves the submodule on the remote's default branch (`main`), which is just another copy of Scidekick, not the upstream snapshot.

**Resolution.** The submodule is not needed to build or run `sk`. For merge tooling, check out the newest available snapshot:

```bash
git -C vendor/oh-my-pi checkout -B vendor/oh-my-pi origin/vendor/oh-my-pi   # 4b237ce, upstream 15.7.3
```

`git status` will then show `M vendor/oh-my-pi`; do not commit that pointer. The lasting fix is for the maintainer who holds `b6f9b29` to push it to the `vendor/oh-my-pi` branch. See [Upstream Fork and Vendor Sync](Upstream-Fork-and-Vendor-Sync).

## `sync-vendor-version.ts` wants to change `15.8.2` to `15.7.3`

**Cause.** `scripts/sync-vendor-version.ts` copies versions from `vendor/oh-my-pi/package.json`, and the snapshot you can fetch is 15.7.3 (see above).

**Resolution.** Run it with `--dry-run` and do not apply it until the submodule points at the 15.8.2 snapshot.

## `Warning: … is classified as haiku-tier, below the configured sonnet-tier minimum …`

**Cause.** After `sk wiki`, `sk journal` or `sk install-skills`, `scidekick/command-tier-warning.ts` checks the `default` role model against `minScienceModelTier` (default `sonnet`). Haiku-class IDs (`haiku`, `flash`, `mini`, `nano`, `lite`, small open models) trigger it.

**Resolution.** Change the default model (`sk --model …`, or the `default` role in settings), or lower `minScienceModelTier` if you accept the risk. The command has already run; the warning is advisory. See [Scientific Skills and Model-Tier Guard](Scientific-Skills-and-Model-Tier-Guard).

## `scienceTierEnforcement: block` does not block anything

**Cause.** Scientific CLI commands always downgrade to `warn` (`evaluateScienceTierDecision`), and the prompt/skill contexts that would honor `block` are not called from the interactive session yet.

**Resolution.** Expected with the current code; tracked as roadmap. Owner: `packages/coding-agent/src/scidekick/model-tier.ts`.

## `Skill "<name>" not validated for model <id>. Performance may vary.`

**Cause.** The skill validation gate in `session/agent-session.ts` flags every loaded skill that is in the registry (`agent.db` → `skill_metadata`) without the current model ID in `validated_models`. Nothing records validations yet, so all skills installed by `sk install-skills` trigger it.

**Resolution.** Informational. It reminds you that unvalidated skill–model pairs can hurt results.

## A page I put in `.sk/wiki/papers/` does not show up in `sk wiki list` / `query`

**Cause.** `FilesystemWikiBackend.list` reads only top-level `.sk/wiki/*.md`. The `papers/ runs/ notes/ claims/` subfolders created by `init` are not scanned.

**Resolution.** Create entries with `sk wiki new|page|ingest` (they are written flat), or move the file to `.sk/wiki/<slug>.md` with YAML frontmatter. See [Research Wiki and Journal](Research-Wiki-and-Journal).

## `Base protocol not found: <slug>` from `sk wiki lint`

**Cause.** An `experiment` entry's `base_protocol` does not match any existing slug.

**Resolution.** Use the exact slug from `sk wiki list` (lowercase, hyphenated, up to 64 characters).

## `Journal entry not found: <id>` from `sk journal link`

**Cause.** The id must match a heading `## HH:MM:SS — <id>` in a file under `.sk/journal/<YYYY>/`. Ids are `YYYY-MM-DD-HHMMSS` in local time.

**Resolution.** Copy the id from `sk journal today` or the `add` output.

## `Failed to clone repository` / `No SKILL.md files found in the repository.`

**Cause.** `sk install-skills` shallow-clones the source with `git`; `owner/name` expands to a GitHub URL and any other string is treated as a local path.

**Resolution.** Check that `git` is on `PATH`, the repo is reachable, and it contains `SKILL.md` files outside dot-directories. Use `--list` to preview.

## Editor (Zed / VS Code) cannot start Scidekick over ACP

**Cause.** Usually a relative `sk` path, or provider keys missing because GUI-launched editors don't inherit shell env.

**Resolution.** Re-run `sk acp-install zed|vscode` (writes the absolute path; a `.bak` copy is kept), and add provider keys to the entry's `env`. In Zed, use **dev: open acp logs**. See [CLI and Integration Modes](CLI-and-Integration-Modes).

## Docs mention `~/.omp/…` but nothing is there

**Cause.** Inherited Oh My Pi docs use upstream defaults. Scidekick's config root is `~/.sk` (`CONFIG_DIR_NAME` in `packages/utils/src/dirs.ts`).

**Resolution.** Read `~/.omp` as `~/.sk`. See [Configuration and Identity](Configuration-and-Identity).

## Related pages

- [Configuration and Identity](Configuration-and-Identity)
- [Upstream Fork and Vendor Sync](Upstream-Fork-and-Vendor-Sync)
- [Scientific Skills and Model-Tier Guard](Scientific-Skills-and-Model-Tier-Guard)
