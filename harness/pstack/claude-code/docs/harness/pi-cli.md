# pstack in Pi CLI

pstack skills use Cursor names. In Pi CLI, read this file once per session, then map each Cursor name in a skill with the tables below. `<pstack>` means the directory that holds `skills/` and `docs/`. Follow the tools, agents, commands, and parameters available in the current session. Do not send Cursor, Claude Code, Codex, or Antigravity tool names or model settings the host does not provide.

## Tools

| Cursor | Pi CLI |
| --- | --- |
| `Task` | Use an available native subagent/delegation capability when exposed. pstack ships `agents/poteto-agent.md` and `agents/comment-sicko.md`; make the child read the matching agent file at its absolute path and provide `poteto-mode/SKILL.md` as required. If no native delegation exists, report required lanes unavailable instead of counting the parent as an independent reviewer. |
| `AskQuestion` | Use an available interactive question mechanism in interactive sessions; in non-interactive/print mode ask in plain text or report the required question as blocked. Playbook rules about product/preference choices still apply. |
| Open a todolist, or copy steps into the todolist tool | Use the session's plan/task tracking if exposed; otherwise keep the playbook steps in the reply and update their state at each phase. |
| Run another pstack skill (`/name` or "the **name** skill") | Read `<pstack>/skills/<name>/SKILL.md` and follow it. For explicit launch, Pi accepts a skill path via `pi --skill <file-or-directory>` or discovers skills through its configured skill directories; use the installed/skill name when exposed, otherwise read it directly. `/pstack:name` is not assumed to be a Pi command. |
| `Read`, `Grep`, `Glob`, `Bash` | Use Pi's built-in `read`, `grep`, `find`, `ls`, and `bash` tools when exposed. Enable only the tools the task needs; keep their permission scopes intact. |
| `Write`, `Edit` | Use Pi's built-in `write` and `edit` tools when the task authorizes file changes. Keep pstack's scope and backup rules; do not run plugin scripts the user has not authorized. |
| Fetch a URL or search the web | Use an exposed web/search capability only when present. Otherwise report web access unavailable instead of fabricating sources. |
| Launch another Pi command | Use `pi --extension <path>` and `pi --skill <path>` only when the caller explicitly asks to install or run pstack in a child run. Do not install packages/extensions during ordinary skill execution. |

## Subagent fields

| Cursor | Pi CLI |
| --- | --- |
| `subagent_type: generalPurpose` | Use an available general-purpose/default native role. Omit an unsupported role field rather than inventing a registered type. |
| `poteto-agent`, `comment-sicko` | Give the delegate the matching `agents/*.md` instructions at their absolute paths. Do not pass `pstack:` or Claude plugin namespace names as built-in Pi agent types. |
| Worker kickoff | Preserve the required first line pointing at the absolute `<pstack>/skills/poteto-mode/worker.md`. Give a poteto-agent delegate the absolute `poteto-mode/SKILL.md` path too. The parent runs the playbook; delegates do their scoped work. |
| The `model:` line in `poteto-agent`'s file | It is a Cursor model hint, not a Pi binding. Resolve the Pi model with the Models table and pass it through the actual model/thinking controls. |
| `readonly: true` | Require read-only work in the brief. Pi's tool allowlist can expose only `read`, `grep`, `find`, `ls` for an authorized child when that control exists. A brief alone is not a sandbox; inspect the resulting diff. |
| `readonly: false` | Use a role capable of the required work within the session's permission policy. It grants no additional permissions. |
| `environment: "cloud"` | Use an isolated local Git worktree for a code-writing delegate, with the intended base named in the brief. This does not create a Cursor cloud agent and does not keep the child running after the host exits. |
| `environment: "local"` | Omit. The child already runs through the local host. |
| `cloud_base_branch: <b>` | Name the base branch/commit in the brief and create the worktree from that base before writing. Do not pass a Cursor field the tool does not accept. |
| `run_in_background: true` | Use native async/background delegation when exposed. Track the native child/agent ID and use the native wait/report path. Do not substitute shell PIDs for agent IDs. |
| `resume` | Use Pi's `--resume`, `--continue`, `--fork`, `--session`, or a native child-resume mechanism only where the playbook permits state-dependent reuse. Otherwise start a fresh child with the consolidated brief, prior result, and branch. Never assume a Cursor `resume` parameter exists. |
| `file_attachments` | Put absolute readable paths in the brief and ensure the child's workspace can read them. Do not imply inaccessible files were attached. |

Native APIs and available agents can differ by Pi version and installed extensions. The session's own tool and agent schema takes precedence over the examples above.

## Models

| Cursor | Pi CLI |
| --- | --- |
| `claude-<family>-<version>-<effort>[-fast]`, such as `claude-opus-5-5-xhigh-fast` | Use only a model pattern/ID that `pi --list-models` reports and that the dispatch supports, unless the user configured that role. A Cursor family slug is not automatically a valid Pi model. Without a mapped choice, inherit the session model and report the requested model did not run. |
| A `gemini-*`, `gpt-*`, `grok-*`, `anthropic/*`, `openai/*`, or other provider slug | Use it only when the installed provider/catalog offers that exact model or pattern and the dispatch accepts it. Otherwise inherit the parent and report the substitution. This adapter does not launch external provider CLIs. |
| `auto` or `inherit-parent` | Omit the model override and use an effort that does not break the role's family requirement. Check native agent/model defaults before claiming inheritance. |
| The `pstack-models.mdc` rule, `~/.cursor/rules/pstack-models.mdc` | Read `<pi-home>/pstack-models.md` explicitly when resolving roles. `<pi-home>` is `$PI_CODING_AGENT_DIR`, or `$HOME/.pi/agent` when unset. Pi does not load Cursor `.mdc` or Claude `rules/` files. |
| A role value `<model> <effort>`, such as a configured Pi model followed by `high` | Resolve `<model>` against `pi --list-models` and `<effort>` against Pi's supported `--thinking` levels (`off`, `minimal`, `low`, `medium`, `high`, `xhigh`, `max`). In an authorized child launch, the CLI equivalents are `pi --model <model>` and `pi --thinking <level>`; do not start a billed fallback run without authorization. |

Drop Cursor's `-fast` suffix; it is not a Pi model ID or a promise of fast mode. A request for another vendor does not prove cross-vendor diversity. When the host cannot run a required independent/diverse lane, name the missing capability and leave its verdict unproved; do not count the parent as another reviewer.

## setup-pstack

Follow the original setup steps, with these Pi equivalents:

- Detect. Use `pi --list-models`, `pi --models`, `--thinking`, and the exposed agent capabilities to determine model patterns, thinking levels, and available roles. A CLI flag accepting a value does not prove that model or role is usable in a child.
- Defaults. Convert each role's default to `inherit-parent` unless a valid Pi model was chosen for it. Preserve the original role labels and budget selection. Explain any reduction in model/vendor diversity before saving choices.
- Write. Write `<pi-home>/pstack-models.md` with the setup comment lines, `# budget`, and one line per original role. Values are `<model> <effort>` or `inherit-parent`. This explicitly selected setup operation may update that sheet; it does not edit the user's `AGENTS.md`, `CLAUDE.md`, `rules/`, hooks, provider credentials or existing plugin installation.
- Load. Read the sheet at poteto-mode startup and whenever a standalone skill needs a role. Pass resolved choices in delegate briefs. No `@` include is required or assumed.

## Paths

| Cursor | Pi CLI |
| --- | --- |
| `.cursor/skills/`, `~/.cursor/skills/` | Use the actual installed/linked pstack skill directories, Pi's configured skill discovery paths, or explicit `pi --skill <path>` for an installed tree. `AGENTS.md`/`CLAUDE.md` context files are not substitutes for a missing `SKILL.md`. |
| `~/.cursor/plugins/`, `.cursor-plugin/plugin.json` | Use the actual installed/linked location. A Cursor plugin manifest is not proof of a Pi install and does not imply a global instruction install. This adapter does not supply a Pi plugin manifest. |
| `.cursor/rules/*.mdc`, always-applied rules | Pi's `AGENTS.md`, `CLAUDE.md`, or explicit `--system-prompt`/`--append-system-prompt` context. Do not overwrite them during ordinary invocation. The role sheet is read explicitly as described above. |
| `https://github.com/cursor/plugins/blob/main/pstack/<path>` | Use `<pstack>/<path>` when present locally. Read the local shipped copy instead of fetching it. Give the URL when a user needs a source link. |
| Transcripts in `~/.cursor/projects/<slug>/agent-transcripts/` | Use `--session`, `--resume`, or the session storage location exposed by the current host, commonly under `<pi-home>/sessions/` or `--session-dir`. Verify the session ID and working directory before reading. Do not search unrelated project chats or reuse Cursor/Claude encodings. |
| `/tmp/<slug>` | Use the host's temporary-directory facility, such as `$env:TEMP` on Windows. Quote paths and distinguish native Windows paths from Git Bash `/c/...` paths. |

## Cursor-only features

When a step needs a Cursor-only feature, write `Skipped in Pi CLI: <step>. <reason>.` and proceed with work that does not depend on it. If a later step needs its output, identify that output as unavailable; a skipped verification is not a pass.

| Cursor | Pi CLI |
| --- | --- |
| Cursor Automations: routines, `update_state`, automation webhooks, routine wakes, `SendToUser` | Not supplied by this adapter. Skip with the line. Do not create schedulers, webhooks, or external writes merely to emulate them. |
| Bugbot | Existing GitHub review comments are ordinary review threads, handled through the forge command in the playbook. Skip starting, configuring, or waiting for a Cursor-only Bugbot service when absent. |
| Cursor cloud agents | Use local native/worktree agents as mapped above. Skip cloud dashboard, VM management and other cloud-service-specific steps. |
| Built-in PR tool | Use the playbook's forge CLI, such as `gh pr create`, under the same task authorization. If unavailable, report the missing capability rather than fabricating a PR URL. |
| `control-ui`, `control-cli`, `/deslop` from `cursor-team-kit` | Read the installed sibling skill when available and map its tools through this adapter. Otherwise report that dependency unavailable; a project verification driver can supply evidence only when it actually exercises the required surface. Do not claim the missing skill ran. |
| `/loop` | Use a scheduler only if the current Pi host provides it and the task authorizes recurring execution. Otherwise report unattended wakeups unavailable; do not substitute an unbounded shell sleep loop. |

## Installation and verification

The generated Pi tree is `harness/pstack/pi-cli/`. It contains the original skills, worker and agent instructions, guide, this adapter, and the complete `AGENTS.pi-cli.md` reference file.

For explicit use, invoke a pstack skill through an installed/linked skill directory or `pi --skill <path>`. Keep the checkout's `docs/` and `agents/` resources adjacent to the linked skills, and preserve `worker.md` and both agent instruction files at their absolute paths. Do not overwrite an existing skill directory. Refresh/restart discovery as required by the host, then invoke `/poteto-mode` or load the matching skill explicitly.

This adapter does not add automatic routing hooks, session-start sync, global model settings, or Codex/Claude/Antigravity manifests. Runtime dispatch, extension loading, and model availability must be checked against the live host; a static mapping check cannot prove a billed child or agent run succeeded.

Reference facts were checked with `pi` 0.83.0 (`--help`, `--list-models`, model/thinking, skill, extension, session, and tool flags). Available models depend on configured providers; read `pi --list-models` rather than assuming a fixed catalog.
