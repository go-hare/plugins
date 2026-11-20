# pstack in OpenCode

pstack skills use Cursor names. In OpenCode, read this file once per session, then map each Cursor name in a skill with the tables below. `<pstack>` means the directory that holds `skills/` and `docs/`. Follow the tools, agents, commands, and parameters available in the current session. Do not send Cursor, Claude Code, Codex, or Antigravity tool names or model settings the host does not provide.

## Tools

| Cursor | OpenCode |
| --- | --- |
| `Task` | The native `subagent` tool, when available through the active agent's `subagent` permission. It takes an agent ID, short description, and complete prompt; it returns a `sessionID` immediately, and passing that ID back continues the same child conversation. Only subagent-mode agents can be launched. If it is unavailable, report required lanes unavailable instead of counting the parent as an independent reviewer. |
| `AskQuestion` | The available interactive user-question mechanism; in non-interactive/headless mode, ask in plain text. Playbook rules about product/preference choices still apply. |
| Open a todolist, or copy steps into the todolist tool | Use the session's plan/task tracking if exposed; otherwise keep the playbook steps in the reply and update their state at each phase. |
| Run another pstack skill (`/name` or "the **name** skill") | Read `<pstack>/skills/<name>/SKILL.md` and follow it. When OpenCode advertises it, load it through the `skill` tool by ID, or invoke it explicitly with `@skill-id` (for example `@poteto-mode`). `/pstack:name` is not assumed to be an OpenCode command. |
| `Read`, `Grep`, `Glob`, `Bash` | Use the exposed file/search tools when present; otherwise the exposed shell tool with the platform's expected syntax. Use `rg` for searches. |
| `Write`, `Edit` | Use the exposed patch/edit tool or shell edits under the task's permission. Keep pstack's scope and backup rules; do not run plugin scripts the user has not authorized. |
| Fetch a URL or search the web | Use `webfetch`/`websearch` only when the session exposes them. Otherwise report web access unavailable instead of fabricating sources. |

## Subagent fields

| Cursor | OpenCode |
| --- | --- |
| `subagent_type: generalPurpose` | Use an available general-purpose/default agent. Omit an unsupported role field rather than inventing a registered type. |
| `poteto-agent`, `comment-sicko` | Give the delegate the matching `agents/*.md` instructions at their absolute paths or register equivalent subagent definitions. Do not pass `pstack:` or Claude plugin namespace names as built-in OpenCode agent IDs. |
| Worker kickoff | Preserve the required first line pointing at the absolute `<pstack>/skills/poteto-mode/worker.md`. Give a poteto-agent delegate the absolute `poteto-mode/SKILL.md` path too. The parent runs the playbook; delegates do their scoped work. |
| The `model:` line in `poteto-agent`'s file | It is a Cursor model hint, not an OpenCode binding. Resolve the OpenCode `provider/model#variant` with the Models table and pass it through the agent or `subagent` configuration. |
| `readonly: true` | Use a real read-only agent/sandbox only when exposed. A brief alone is not a sandbox; inspect the resulting diff. |
| `readonly: false` | Use a role capable of the required work within the session's permission policy. It grants no additional permissions. |
| `environment: "cloud"` | Use an isolated local Git worktree for a code-writing delegate, with the intended base named in the brief. This does not create a Cursor cloud agent and does not keep the child running after the host exits. |
| `environment: "local"` | Omit. The child already runs through the local host. |
| `cloud_base_branch: <b>` | Name the base branch/commit in the brief and create the worktree from that base before writing. Do not pass a Cursor field the tool does not accept. |
| `run_in_background: true` | Native subagents run in the background in child sessions. Track returned session IDs and use the native report/notification path; do not substitute shell PIDs for session IDs. |
| `resume` | Pass an existing `sessionID` back to `subagent` only where the playbook permits state-dependent reuse. Otherwise start a fresh child session with the consolidated brief, prior result, and branch. Never assume a Cursor `resume` parameter exists. |
| `file_attachments` | Put absolute readable paths in the brief and ensure the child's workspace can read them. Do not imply inaccessible files were attached. |

Subagents inherit model and effort by default when `model` is unset. Use agent/permission configuration to control whether they can launch further subagents, run commands, edit files, or access external directories; do not let convenience roles bypass those rules.

## Models

| Cursor | OpenCode |
| --- | --- |
| `claude-<family>-<version>-<effort>[-fast]`, such as `claude-opus-5-5-xhigh-fast` | Use only a `provider/model#variant` name that the current OpenCode catalog supports, unless the user configured that role. A Cursor family slug is not automatically a valid OpenCode model. Without a mapped choice, inherit the session model and report the requested model did not run. |
| A `gemini-*`, `gpt-*`, `grok-*`, `anthropic/*`, `openai/*`, or other provider slug | Use it only when the installed provider/catalog offers that exact model or variant and the dispatch accepts it. Otherwise inherit the parent and report the substitution. This adapter does not launch external provider CLIs. |
| `auto` or `inherit-parent` | Omit the model override and use an effort that does not break the role's family requirement. Check native agent/model defaults before claiming inheritance. |
| The `pstack-models.mdc` rule, `~/.cursor/rules/pstack-models.mdc` | Read `<opencode-home>/pstack-models.md` explicitly when resolving roles. `<opencode-home>` is `$OPENCODE_CONFIG`, or `$HOME/.config/opencode` when unset. OpenCode does not load Cursor `.mdc` or Claude `rules/` files. |
| A role value `<model> <effort>`, such as a configured OpenCode model followed by `high` | Resolve `<model>` against the active provider/model catalog and `<effort>` against that model's supported levels. In an authorized child launch, use the host's model/effort controls; do not start a billed fallback run without authorization. |

Drop Cursor's `-fast` suffix; it is not an OpenCode model ID or a promise of fast mode. A request for another vendor does not prove cross-vendor diversity. When the host cannot run a required independent/diverse lane, name the missing capability and leave its verdict unproved; do not count the parent as another reviewer.

## setup-pstack

Follow the original setup steps, with these OpenCode equivalents:

- Detect. Use the session's model picker/catalog and agent/effort controls to determine available model IDs, variants, and available roles. A CLI flag accepting a value does not prove that model or role is usable in a child.
- Defaults. Convert each role's default to `inherit-parent` unless a valid OpenCode model was chosen for it. Preserve the original role labels and budget selection. Explain any reduction in model/vendor diversity before saving choices.
- Write. Write `<opencode-home>/pstack-models.md` with the setup comment lines, `# budget`, and one line per original role. Values are `<model> <effort>` or `inherit-parent`. This explicitly selected setup operation may update that sheet; it does not edit the user's `AGENTS.md`, `GEMINI.md`, `rules/`, hooks, provider credentials or existing plugin installation.
- Load. Read the sheet at poteto-mode startup and whenever a standalone skill needs a role. Pass resolved choices in delegate briefs. No `@` include is required or assumed.

## Paths

| Cursor | OpenCode |
| --- | --- |
| `.cursor/skills/`, `~/.cursor/skills/` | Project `.opencode/skills/`, `.agents/skills/`, `.claude/skills/`, global `~/.config/opencode/skills`, or registered plugin skills. `AGENTS.md`/`GEMINI.md` context files are not substitutes for a missing `SKILL.md`. |
| `~/.cursor/plugins/`, `.cursor-plugin/plugin.json` | Use the actual installed/linked plugin location or `.opencode/` plugin configuration. A Cursor plugin manifest is not proof of an OpenCode install and does not imply a global instruction install. |
| `.cursor/rules/*.mdc`, always-applied rules | Workspace/global `AGENTS.md`, `GEMINI.md`, `.opencode/rules`, or explicit `--system-prompt`/`--append-system-prompt` context. Do not overwrite them during ordinary invocation. The role sheet is read explicitly as described above. |
| `https://github.com/cursor/plugins/blob/main/pstack/<path>` | Use `<pstack>/<path>` when present locally. Read the local shipped copy instead of fetching it. Give the URL when a user needs a source link. |
| Transcripts in `~/.cursor/projects/<slug>/agent-transcripts/` | Use the current session's actual transcript/session store when exposed; otherwise report missing history and do not search unrelated project chats. Do not reuse Cursor/Claude encodings. |
| `/tmp/<slug>` | Use the host's temporary-directory facility, such as `$env:TEMP` on Windows. Quote paths and distinguish native Windows paths from Git Bash `/c/...` paths. |

## Cursor-only features

When a step needs a Cursor-only feature, write `Skipped in OpenCode: <step>. <reason>.` and proceed with work that does not depend on it. If a later step needs its output, identify that output as unavailable; a skipped verification is not a pass.

| Cursor | OpenCode |
| --- | --- |
| Cursor Automations: routines, `update_state`, automation webhooks, routine wakes, `SendToUser` | Not supplied by this adapter. Skip with the line. Do not create schedulers, webhooks, or external writes merely to emulate them. |
| Bugbot | Existing GitHub review comments are ordinary review threads, handled through the forge command in the playbook. Skip starting, configuring, or waiting for a Cursor-only Bugbot service when absent. |
| Cursor cloud agents | Use local native/worktree agents as mapped above. Skip cloud dashboard, VM management and other cloud-service-specific steps. |
| Built-in PR tool | Use the playbook's forge CLI, such as `gh pr create`, under the same task authorization. If unavailable, report the missing capability rather than fabricating a PR URL. |
| `control-ui`, `control-cli`, `/deslop` from `cursor-team-kit` | Read the installed sibling skill when available and map its tools through this adapter. Otherwise report that dependency unavailable; a project verification driver can supply evidence only when it actually exercises the required surface. Do not claim the missing skill ran. |
| `/loop` | Use a scheduler only if the current OpenCode host provides it and the task authorizes recurring execution. Otherwise report unattended wakeups unavailable; do not substitute an unbounded shell sleep loop. |

## Installation and verification

The generated OpenCode tree is `harness/pstack/opencode/`. It contains the original skills, worker and agent instructions, guide, this adapter, and the complete `AGENTS.opencode.md` reference file.

For a skills-only setup, expose each directory under that tree's `skills/` in `.opencode/skills/`, `.agents/skills/`, or `~/.config/opencode/skills` using supported directory links while keeping `docs/` and `agents/` resources adjacent. For a plugin-shaped setup, use OpenCode's plugin/agent registration to map `poteto-agent` and `comment-sicko` to `agents/*.md` only after inspecting them. Do not overwrite an existing skill directory. Refresh/restart skill discovery as required, then invoke `@poteto-mode` or load the matching `skill` tool.

This adapter does not add automatic routing hooks, session-start sync, global model settings, or Codex/Claude/Antigravity manifests. Runtime dispatch and model availability must be checked against the live host; a static mapping check cannot prove a billed child or agent run succeeded.

Reference facts were checked with OpenCode 2.x docs and current installation guides. The `subagent` tool and agent `mode: subagent` are documented separately; this adapter uses them rather than assuming `/loop`, `@skill`, or Claude plugin namespaces.
