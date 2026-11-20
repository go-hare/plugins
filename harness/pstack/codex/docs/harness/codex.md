# pstack in Codex

pstack skills use Cursor names. In Codex, read this file once per session, then map each Cursor name in a skill with the tables below. `<pstack>` means the folder that holds `skills/` and `docs/`. Use the tools and parameter schema exposed by the current Codex session. Do not pass Cursor fields, model slugs, or tool arguments that schema does not accept. Report an unavailable capability rather than claiming it ran.

## Tools

| Cursor | Codex |
| --- | --- |
| `Task` | The session's native subagent tool, commonly `spawn_agent`. Read its live schema before dispatch. If subagents are unavailable, report that lane unavailable; do not invent an independent reviewer. |
| `AskQuestion` | The session's user-input tool, commonly `request_user_input`, when available in the current mode; otherwise ask in plain text. Keep the playbook's limits on when a question belongs to the user. |
| Open a todolist, or copy steps into the todolist tool | Use `update_plan` when available. Otherwise keep the playbook steps in the reply and update their state at each phase. |
| Run another pstack skill (`/name` or "the **name** skill") | Read `<pstack>/skills/<name>/SKILL.md` and follow it. For explicit invocation use Codex's skill picker or `$name`, for example `$poteto-mode`; `/pstack:name` is not assumed to be a Codex command. |
| `Read`, `Grep`, `Glob`, `Bash` | Use the session's file/search tools when available; otherwise its shell tool, commonly `exec_command` or `shell_command`. Use `rg` for search. Choose PowerShell or POSIX syntax for the actual shell. |
| `Write`, `Edit` | Use the exposed patch or file-edit tool, commonly `apply_patch`, with its actual schema. |
| Fetch a URL or search the web | Use the exposed web tool, if present, or an available read-only HTTP client. Do not assume a tool with a Cursor name exists. |

## Subagent fields

| Cursor | Codex |
| --- | --- |
| `subagent_type: generalPurpose` | Use an available general-purpose/default native role. Omit an unsupported role field rather than inventing a registered type. |
| `poteto-agent`, `comment-sicko` | Read the corresponding `<pstack>/agents/<name>.md` and provide its role instructions to the native subagent. The `poteto-agent` frontmatter contains a Cursor model, not a Codex model setting. Do not assume these names are registered Codex agent types. |
| Worker kickoff | Preserve the required first line pointing at the absolute `<pstack>/skills/poteto-mode/worker.md`. Give a poteto-agent delegate the absolute `poteto-mode/SKILL.md` path too. The parent runs the playbook; delegates do their scoped work. |
| The `model:` line in `poteto-agent`'s file | Resolve the Codex role model with the Models table below. Pass a model and reasoning effort only through fields supported by the live dispatch schema or a configured native role. |
| `readonly: true` | Require read-only work in the brief. Use a native read-only role/sandbox only when actually supported. A brief is not a sandbox; report the distinction and inspect the resulting diff. |
| `readonly: false` | Use a role capable of the required work within the current session's sandbox and approval policy. It grants no additional permissions. |
| `environment: "cloud"` | Use an isolated local Git worktree for a code-writing delegate, with the intended base specified. This does not create a Cursor cloud agent or keep it running after the host exits. |
| `environment: "local"` | Omit. The native child already runs through the Codex host. |
| `cloud_base_branch: <b>` | State the base branch/commit in the brief; create the worktree from that base before writing. Do not pass an unsupported Cursor field. |
| `run_in_background: true` | Native subagent dispatch is asynchronous. Track returned IDs and wait with the exposed wait tool, commonly `wait_agent`; release completed agents with `close_agent` if provided. Shell process handles are not subagent IDs. |
| `resume` | Use the exposed follow-up tool, commonly `send_input`, only in the cases the playbook permits reuse. Otherwise start a fresh child with the consolidated brief, prior result, and branch. Never assume a Cursor `resume` parameter exists. |
| `file_attachments` | Put absolute paths in the brief and ensure the child's workspace can read them. Do not imply inaccessible files were attached. |

If the native subagent feature is disabled, enable it only under the user's configuration authorization. On Codex CLI 0.146.0, `codex features list` exposes `multi_agent`; a session can be launched with `codex --enable multi_agent`. Do not change user configuration merely to execute a skill. Native APIs and available roles can differ by Codex version; the session schema takes precedence over the example names above.

## Models

| Cursor | Codex |
| --- | --- |
| `claude-<family>-<version>-<effort>[-fast]`, such as `claude-opus-5-5-xhigh-fast` | Look up that workflow role in the Codex override sheet. A Claude family or full ID is not automatically a valid Codex model. Without a Codex override, omit the model override, inherit the session model, and say the requested family did not run. |
| A `gemini-*`, `grok-*`, `gpt-*`, or other provider slug | Use it only if the current Codex provider/catalog actually offers that exact model and the dispatch supports selecting it. Otherwise inherit the parent and report the substitution. This adapter does not launch external provider CLIs. |
| `auto` or `inherit-parent` | Omit model and effort overrides so the native child inherits. Check configured native roles/defaults if the host gives them precedence; do not claim inheritance without that check. |
| The `pstack-models.mdc` rule, `~/.cursor/rules/pstack-models.mdc` | Read `<codex-home>/pstack-models.md` explicitly when resolving roles. `<codex-home>` is `$CODEX_HOME`, or `$HOME/.codex` when unset. Codex does not automatically load Cursor `.mdc` or Claude `rules/` files. |
| A role value `<model> <effort>`, such as a configured Codex model followed by `high` | Resolve the model using the active provider/catalog and effort using that model's supported levels. Use the live subagent schema/configuration. In a separate authorized `codex exec` run, the CLI equivalents are `--model <model>` and `-c model_reasoning_effort=<effort>`. Do not start a billed fallback run without authorization. |

Drop the Cursor `-fast` suffix; it is not a Codex model ID or a promise of fast mode. A role's request for another vendor does not prove cross-vendor diversity. When the host cannot run a required independent or diverse panel, name the missing capability and leave the corresponding verdict unproved; do not count the parent as another reviewer.

## setup-pstack

Follow the original setup steps, with these Codex equivalents:

- Detect. Use the active Codex model picker/provider and live tool schema to determine available model IDs, effort levels, subagent fields and concurrency support. CLI `--model` accepts an ID but does not prove that model is available. Do not fabricate a catalog from Cursor defaults.
- Defaults. Convert each role's default to `inherit-parent` unless a valid Codex model has been chosen for it. Preserve the original role labels and budget selection. Explain any reduction in model/vendor diversity before saving the choices.
- Write. Write `<codex-home>/pstack-models.md` with the setup comment lines, `# budget`, and one line per original role. Values are `<model> <effort>` or `inherit-parent`. This explicitly selected setup operation may update that sheet; it does not edit the user's `AGENTS.md`, `config.toml`, hooks, provider credentials or existing plugin installation.
- Load. Read the sheet at the start of poteto-mode and whenever a standalone skill needs a role. Pass resolved choices in delegate briefs. No `@` include is required or assumed in Codex.

## Paths

| Cursor | Codex |
| --- | --- |
| `.cursor/skills/`, `~/.cursor/skills/` | Project `.agents/skills/` and user `$HOME/.agents/skills/`. Keep the generated tree's shared resources reachable; do not copy a lone SKILL.md away from its references. |
| `~/.cursor/plugins/`, `.cursor-plugin/plugin.json` | Use the actual installed/linked skill location. A Cursor manifest is not proof of a Codex plugin install. This adapter's skills-only installation does not require a Codex manifest or register native agents. |
| `.cursor/rules/*.mdc`, always-applied rules | Codex's project `AGENTS.md` and global `<codex-home>/AGENTS.md`. Do not overwrite them or paste the plugin into them during ordinary invocation. The role sheet is read explicitly as described above. |
| `https://github.com/cursor/plugins/blob/main/pstack/<path>` | Use `<pstack>/<path>` when present locally. Read the local shipped copy instead of fetching it. Give the URL when a user needs a source link. |
| Transcripts in `~/.cursor/projects/<slug>/agent-transcripts/` | Use the current Codex session's actual rollout path/ID when exposed; CLI histories commonly live under `<codex-home>/sessions/`. Verify the record's session ID and working directory before reading. Do not search unrelated project chats or assume Cursor/Claude filename encodings. |
| `/tmp/<slug>` | Use the host's temporary-directory facility, such as `$env:TEMP` on Windows. Quote paths, and distinguish native Windows paths from Git Bash `/c/...` paths. |

## Cursor-only features

When a step needs a Cursor-only feature, write `Skipped in Codex: <step>. <reason>.` and proceed with work that does not depend on it. If a later step needs its output, identify that output as unavailable; a skipped verification is not a pass.

| Cursor | Codex |
| --- | --- |
| Cursor Automations: routines, `update_state`, automation webhooks, routine wakes, `SendToUser` | Not supplied by this adapter. Skip with the line. Do not create schedulers, webhooks or external writes merely to emulate them. |
| Bugbot | Existing GitHub review comments are ordinary review threads, handled through the forge command in the playbook. Skip starting, configuring or waiting for a Cursor-only Bugbot service when absent. |
| Cursor cloud agents | Use local isolated native agents as mapped above. Skip cloud dashboard, VM management and other cloud-service-specific steps. |
| Built-in PR tool | Use the playbook's forge CLI, such as `gh pr create`, under the same task authorization. If unavailable, report the missing capability rather than fabricating a PR URL. |
| `control-ui`, `control-cli`, `/deslop` from `cursor-team-kit` | Read the installed sibling skill when available and map its tools through this adapter. Otherwise report that dependency unavailable; project verification drivers can supply evidence only when they actually exercise the required surface. Do not claim the missing skill ran. |
| `/loop` | Use a scheduler only if the current Codex host provides it and the task authorizes recurring execution. Otherwise report unattended wakeups unavailable; do not substitute an unbounded shell sleep loop. |

## Installation and verification

The generated Codex tree is `harness/pstack/codex/`. It contains the original skills, worker and agent instructions, guide and this adapter. Skills that use Cursor primitives include an adapter-read line so direct invocation also finds this file.

For a skills-only installation, expose each directory under that tree's `skills/` in project `.agents/skills/` or user `$HOME/.agents/skills/` using the host's supported directory links. Keep the checkout and its `docs/` and `agents/` alongside the linked skills. On Windows use a real directory symlink or junction supported by the filesystem. Resolve the physical skill path before reading sibling resources. Do not overwrite an existing skill directory. Restart or refresh skill discovery, then use the skill picker or `$poteto-mode`.

This is a skills-only adapter, not an automatic Codex marketplace/hook installation. Do not assume ordinary requests auto-enter poteto-mode. Core dispatch and configuration must be checked against the live Codex session; a static mapping check cannot prove a billed agent run succeeded.

Reference facts were checked with Codex CLI 0.146.0 (`--help`, `plugin --help`, `exec --help`, `features list`) and the official [skills](https://learn.chatgpt.com/docs/build-skills) and [subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents) documentation. Read current tool schemas rather than treating those example names as fixed APIs.
