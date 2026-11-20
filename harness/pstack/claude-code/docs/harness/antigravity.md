# pstack in Antigravity CLI

pstack skills use Cursor names. In Antigravity CLI (`agy`), read this file once per session, then map each Cursor name in a skill with the tables below. `<pstack>` means the directory that holds `skills/` and `docs/`. Follow the tools, agents, commands, and parameters available in the current session. Do not send Cursor, Claude Code, or Codex tool names or model settings the host does not provide.

## Tools

| Cursor | Antigravity CLI |
| --- | --- |
| `Task` | The native subagent/delegation capability. Plugin custom agents live in `agents/<name>.md` or `agents/<name>/agent.md`; pstack ships `agents/poteto-agent.md` and `agents/comment-sicko.md`. Use the corresponding registered agent when it is available, otherwise use an available general-purpose agent and include the agent file's instructions in the brief. When no native delegation exists, report required fan-out lanes unavailable instead of counting the parent as an independent reviewer. |
| `AskQuestion` | The available interactive user-question mechanism; in non-interactive or print mode, ask in plain text. Playbook rules about product/preference choices still apply. |
| Open a todolist, or copy steps into the todolist tool | Use the session's plan/task tracking if exposed; otherwise keep the playbook steps in the reply and update their state at each phase. |
| Run another pstack skill (`/name` or "the **name** skill") | Read `<pstack>/skills/<name>/SKILL.md` and follow it. When the plugin's skills are installed, Antigravity CLI exposes them as `/<name>`, for example `/poteto-mode`; pstack rules are separate always-on rules, not slash commands. |
| `Read`, `Grep`, `Glob`, `Bash` | Use the exposed file/search tools when present; otherwise the exposed shell/terminal tool. Use `rg` for searches and choose PowerShell or POSIX syntax for the actual shell. |
| `Write`, `Edit` | Use the exposed patch/edit tool or apply changes the way the current agent expects files to be written. Keep pstack's scope and backup rules; do not run plugin scripts the user has not authorized. |
| Fetch a URL or search the web | Use the exposed browser/web capability only when present. Otherwise report network/web access unavailable instead of fabricating sources. |

## Subagent fields

| Cursor | Antigravity CLI |
| --- | --- |
| `subagent_type: generalPurpose` | Use an available default/general-purpose agent. Only use a registered `poteto-agent` or `comment-sicko` when the plugin's `agents/` definitions are actually installed and selectable. |
| `poteto-agent`, `comment-sicko` | The corresponding `agents/*.md` definitions ship with the plugin. If the host does not register them, a general-purpose child must still read the matching agent file at its absolute path. Do not pass `pstack:` or Claude plugin namespace names as built-in agent types. |
| Worker kickoff | Preserve the required first line pointing at the absolute `<pstack>/skills/poteto-mode/worker.md`. Give a poteto-agent delegate the absolute `poteto-mode/SKILL.md` path too. The parent runs the playbook; delegates do their scoped work. |
| The `model:` line in `poteto-agent`'s file | It is a Cursor model hint, not an Antigravity model binding. Resolve the Antigravity model with the Models table and pass it through the actual agent/model and `--model`/`--effort` capabilities. |
| `readonly: true` | Require read-only work in the brief. Use a real read-only role/sandbox only if exposed. A brief is not a sandbox; inspect the resulting diff. |
| `readonly: false` | Use a role capable of the required work within the session's permission policy. It grants no additional permissions. |
| `environment: "cloud"` | Use an isolated local Git worktree for a code-writing delegate, with the intended base named in the brief. This does not create a Cursor cloud agent and does not keep the child running after the host exits. |
| `environment: "local"` | Omit. The child already runs through the local host. |
| `cloud_base_branch: <b>` | Name the base branch/commit in the brief and create the worktree from it before writing. Do not pass a Cursor field the tool does not accept. |
| `run_in_background: true` | Use native async/background delegation when the agent/framework exposes it. Track the native child/agent ID and use the native wait/report path. Do not substitute shell PIDs for agent IDs. |
| `resume` | Resume an existing child only where the playbook permits state-dependent reuse, using the exposed agent/resume mechanism. Otherwise start a fresh child with the consolidated brief, prior result, and branch. Never assume a Cursor `resume` parameter exists. |
| `file_attachments` | Put absolute readable paths in the brief and ensure the child's workspace can read them. Do not imply inaccessible files were attached. |

Custom agents in plugin `agents/*.md` are definitions for the Antigravity subagent framework, not a promise that every host exposes dispatch, model selection, independent concurrency, or a particular role name. Keep each subagent in the same permission and worktree isolation the playbook requires.

## Models

| Cursor | Antigravity CLI |
| --- | --- |
| `claude-<family>-<version>-<effort>[-fast]`, such as `claude-opus-5-5-xhigh-fast` | Use only a model name that `agy models` lists and that the dispatch supports, unless the user configured that role. A Cursor family slug is not automatically a valid Antigravity model. Without a mapped choice, inherit the session model and report the requested model did not run. |
| A `gemini-*`, `gpt-*`, `grok-*`, or other provider slug | Use it only when the installed provider/catalog offers that exact model and the dispatch accepts it. Otherwise inherit the parent and report the substitution. This adapter does not launch external provider CLIs. |
| `auto` or `inherit-parent` | Omit the model override and use an effort that does not break the role's family requirement. Check native agent/model defaults before claiming inheritance. |
| The `pstack-models.mdc` rule, `~/.cursor/rules/pstack-models.mdc` | Read `<gemini-home>/pstack-models.md` explicitly when resolving roles. `<gemini-home>` is `$GEMINI_HOME`, or `$HOME/.gemini` when unset. Antigravity does not load Cursor `.mdc` or Claude `rules/` files. |
| A role value `<model> <effort>`, such as a configured Antigravity model followed by `high` | Resolve `<model>` against `agy models` and `<effort>` against the supported CLI values (`low`, `medium`, `high`, `xhigh`, `max`, or the native agent setting). In an authorized child launch, CLI equivalents are `agy --model <model>` and `agy --effort <effort>`; do not start a billed fallback run without authorization. |

Drop Cursor's `-fast` suffix; it is not an Antigravity model ID or a promise of fast mode. A request for another vendor does not prove cross-vendor diversity. When the host cannot run a required independent/diverse lane, name the missing capability and leave its verdict unproved; do not count the parent as another reviewer.

## setup-pstack

Follow the original setup steps, with these Antigravity equivalents:

- Detect. Use `agy models`, the session model/effort controls, and the exposed agent capabilities to determine model IDs, effort levels, and available roles. A CLI flag accepting a value does not prove that model or role is usable in a child.
- Defaults. Convert each role's default to `inherit-parent` unless a valid Antigravity model was chosen for it. Preserve the original role labels and budget selection. Explain any reduction in model/vendor diversity before saving choices.
- Write. Write `<gemini-home>/pstack-models.md` with the setup comment lines, `# budget`, and one line per original role. Values are `<model> <effort>` or `inherit-parent`. This explicitly selected setup operation may update that sheet; it does not edit the user's `AGENTS.md`, `GEMINI.md`, `rules/`, hooks, provider credentials or existing plugin installation.
- Load. Read the sheet at poteto-mode startup and whenever a standalone skill needs a role. Pass resolved choices in delegate briefs. No `@` include is required or assumed.

## Paths

| Cursor | Antigravity CLI |
| --- | --- |
| `.cursor/skills/`, `~/.cursor/skills/` | Workspace `.agents/skills/<skill>/`; global `~/.gemini/antigravity-cli/skills/<skill>/`; or plugin `<plugin>/skills/` when installed. `AGENTS.md`/`GEMINI.md` and `rules/` are rules sources, not substitutes for a missing `SKILL.md`. |
| `~/.cursor/plugins/`, `.cursor-plugin/plugin.json` | `~/.gemini/antigravity-cli/plugins/<name>/`, or a manually linked plugin directory. `plugin.json` identifies the plugin package; it is not a manifest for Claude Code or Codex and does not imply a global instruction install. |
| `.cursor/rules/*.mdc`, always-applied rules | Workspace/global `AGENTS.md`, `GEMINI.md`, `.agents/rules/*.md`, and plugin `rules/*.md` per Antigravity's rule triggers. Do not overwrite them during ordinary invocation. The role sheet is read explicitly as described above. |
| `https://github.com/cursor/plugins/blob/main/pstack/<path>` | Use `<pstack>/<path>` when present locally. Read the local shipped copy instead of fetching it. Give the URL when a user needs a source link. |
| Transcripts in `~/.cursor/projects/<slug>/agent-transcripts/` | Use the current Antigravity session's actual conversation/rollout source when exposed; otherwise report missing history and do not search unrelated project chats. Do not reuse Cursor or Claude path encodings. |
| `/tmp/<slug>` | Use the host's temporary-directory facility, such as `$env:TEMP` on Windows. Quote paths and distinguish native Windows paths from Git Bash `/c/...` paths. |

## Cursor-only features

When a step needs a Cursor-only feature, write `Skipped in Antigravity: <step>. <reason>.` and proceed with work that does not depend on it. If a later step needs its output, identify that output as unavailable; a skipped verification is not a pass.

| Cursor | Antigravity CLI |
| --- | --- |
| Cursor Automations: routines, `update_state`, automation webhooks, routine wakes, `SendToUser` | Not supplied by this adapter. Skip with the line. Do not create schedulers, webhooks, or external writes merely to emulate them. |
| Bugbot | Existing GitHub review comments are ordinary review threads, handled through the forge command in the playbook. Skip starting, configuring, or waiting for a Cursor-only Bugbot service when absent. |
| Cursor cloud agents | Use local native/worktree agents as mapped above. Skip cloud dashboard, VM management and other cloud-service-specific steps. |
| Built-in PR tool | Use the playbook's forge CLI, such as `gh pr create`, under the same task authorization. If unavailable, report the missing capability rather than fabricating a PR URL. |
| `control-ui`, `control-cli`, `/deslop` from `cursor-team-kit` | Read the installed sibling skill when available and map its tools through this adapter. Otherwise report that dependency unavailable; a project verification driver can supply evidence only when it actually exercises the required surface. Do not claim the missing skill ran. |
| `/loop` | Use a scheduler only if the current Antigravity host provides it and the task authorizes recurring execution. Otherwise report unattended wakeups unavailable; do not substitute an unbounded shell sleep loop. |

## Installation and verification

The generated Antigravity tree is `harness/pstack/antigravity/`. It contains the original skills, worker and agent instructions, the guide, `plugin.json` for manual/plugin installation, this adapter, and the complete `AGENTS.antigravity.md` rule file.

For a skills-only setup, expose each directory under that tree's `skills/` in workspace `.agents/skills/` or global `~/.gemini/antigravity-cli/skills/` using the host's supported directory links, while keeping the adjacent `docs/` and `agents/` resources reachable. For a plugin-shaped setup, install the tree through Antigravity's documented plugin mechanism or link it into `~/.gemini/antigravity-cli/plugins/<name>/`. Put a short adapter-loader rule under the plugin's `rules/` directory and keep `AGENTS.antigravity.md` at the plugin root as the complete workflow reference; the loader sends agents to both only where the task calls for them. Do not copy the full AGENTS file into a truncated always-on rule or overwrite an existing skill directory. Use `agy plugin validate <path>` for a plugin tree before installation and `agy plugin list --json` after install. Refresh/restart discovery as the CLI requires, then invoke `/poteto-mode`.

Antigravity rules go under the plugin's `rules/` directory. `rules/AGENTS.md` is always loaded plain Markdown; any additional `rules/*.md` needs a valid `trigger`. `manual` trigger is not supported by the CLI, so put host-specific complete rules in `rules/AGENTS.md` rather than in a manual rule. Do not overwrite workspace `AGENTS.md`, `GEMINI.md`, or the user's global files during installation.

This adapter does not add automatic routing hooks, session-start sync, global model settings, or Codex/Claude manifests. Runtime dispatch and model availability must be checked against the live host; a static mapping check cannot prove a billed child or agent run succeeded.

Reference facts were checked with `agy` 1.4.10 (`--help`, `plugin --help`, `agy models`, `agy agents`) and the official [skills](https://antigravity.google/docs/skills), [rules](https://antigravity.google/docs/rules/), and [plugins](https://antigravity.google/docs/plugins) documentation. The CLI reports `claude-sonnet-4-6`, `claude-opus-4-6-thinking`, `gemini-3.8-flash-{low,medium,high}`, and `gpt-oss-120b-medium`; available models can change with account, provider, and version.
