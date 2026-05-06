---
name: only-use-mcp-tools
description: Use when MCP-backed tools, IDE servers, repository automation, or external integrations are available and local fallback must be blocked.
---

# MCP Tools Mandatory (Global)

## Core Invariant

If a capability is available through MCP, it MUST be executed through `mcpproxy`.

---

## First 30 Seconds (Startup Checklist)

1. Run `mcpproxy_retrieve_tools` for the immediate action.
2. Select exact returned `server:tool`.
3. Route with exact returned `call_with`.
4. Include `intent_reason` and `intent_data_sensitivity` on every proxied call.
5. If tools are missing/unclear, run health and quarantine checks before any further action.

---

## Operational Contract (Fail-Closed)

1. You MUST use `mcpproxy` first.
2. You MUST NOT guess tool names.
3. You MUST run `mcpproxy_retrieve_tools` before upstream calls.
4. You MUST use exact discovered `server:tool` names.
5. You MUST route by discovered `call_with` only:
   - `call_tool_read` → `mcpproxy_call_tool_read`
   - `call_tool_write` → `mcpproxy_call_tool_write`
   - `call_tool_destructive` → `mcpproxy_call_tool_destructive`
6. Every proxy call MUST include:
   - `intent_reason`
   - `intent_data_sensitivity` (`public|internal|private|unknown`)
7. You MUST continue truncated responses via `mcpproxy_read_cache`.
8. You MUST NOT use local/native fallback tools.

---

## Discovery Policy (Deterministic)

When capability is uncertain, you MUST run 3 narrowing discovery queries:
1. Broad capability query
2. Server-scoped query
3. Action-scoped query

If still unclear, you MUST run one sanity pass with `include_stats=true` and retry with explicit likely names.
You MUST NOT infer unavailability from one sparse query.

### Golden Query Pack (Copy/Paste)

- `rider-official backend file read edit search`
- `webstorm-official frontend file read edit search`
- `rider-official vcs changes stage commit log`
- `webstorm-official vcs changes stage commit log`
- `rider-official get_file_text_by_path replace_text_in_file`
- `webstorm-official get_file_text_by_path replace_text_in_file`
- `rider-official search_in_files_by_text search_in_files_by_regex`
- `webstorm-official search_in_files_by_text search_in_files_by_regex`

---

## Health and Security Gate

When discovery is missing/unclear, you MUST run:
1. `mcpproxy_upstream_servers`
2. `mcpproxy_quarantine_security`

You MUST proceed only after these checks are evaluated.

---

## IDE Routing Policy (Strict Split)

- **Backend C#/.NET work MUST default to `rider-official`.**
- **Frontend TS/JS/Vue/CSS/HTML work MUST default to `webstorm-official`.**

`rider-official` and `webstorm-official` share similar tool layout for many read/edit/search actions. You MAY swap server prefixes for equivalent actions when needed, while keeping split defaults for consistency.

### projectPath Rule

When `projectPath` is known, you MUST pass it on Rider/WebStorm tool calls.

Repository-root convention for this environment:
- Projects are one level deep under `/mnt/PROJECTS`.
- Default project root format: `/mnt/PROJECTS/<repo-name>`.
- If repo name is known, you MUST derive and pass `projectPath` using this format.

---

## Common Actions (Hard-Coded Tool Index)

Run one discovery call first, then use these stable names:

| Action | Backend / Rider | Frontend / WebStorm |
|---|---|---|
| Read file | `rider-official:get_file_text_by_path` | `webstorm-official:get_file_text_by_path` |
| Read file (range) | `rider-official:read_file` | `webstorm-official:read_file` |
| Edit file | `rider-official:replace_text_in_file` | `webstorm-official:replace_text_in_file` |
| Edit file (undoable) | `rider-official:replace_text_undoable` | `webstorm-official:replace_text_undoable` |
| Search text | `rider-official:search_in_files_by_text` | `webstorm-official:search_in_files_by_text` |
| Search regex | `rider-official:search_in_files_by_regex` | `webstorm-official:search_in_files_by_regex` |
| Create file | `rider-official:create_new_file` | `webstorm-official:create_new_file` |
| List repos | `rider-official:get_repositories` | `webstorm-official:get_repositories` |
| Check VCS changes | `rider-official:get_vcs_changes` | `webstorm-official:get_vcs_changes` |
| Stage/unstage files | `rider-official:vcs_stage_files` | `webstorm-official:vcs_stage_files` |
| Commit staged changes | `rider-official:vcs_commit` | `webstorm-official:vcs_commit` |
| Commit log | `rider-official:get_vcs_log` | `webstorm-official:get_vcs_log` |
| Show commit | `rider-official:vcs_show_commit` | `webstorm-official:vcs_show_commit` |
| File history | `rider-official:get_vcs_file_history` | `webstorm-official:get_vcs_file_history` |
| List run configs | `rider-official:list_run_configurations` | `webstorm-official:list_run_configurations` |
| Run/debug config | `rider-official:start_run_configuration` / `debug_run_configuration` | `webstorm-official:start_run_configuration` / `debug_run_configuration` |
| Console output | `rider-official:get_console_output` | `webstorm-official:get_console_output` |
| Test results | `rider-official:get_test_results` | `webstorm-official:get_test_results` |
| IDE actions | `rider-official:execute_ide_action` | `webstorm-official:execute_ide_action` |

---

## Detailed Fast Playbooks

### Playbook A — Backend File Edit (Rider)

1. Discover tools for backend edit action.
2. Read target file with `rider-official:get_file_text_by_path` (MUST pass `projectPath` when known).
3. Apply targeted change with `rider-official:replace_text_in_file` (MUST pass `projectPath` when known).
4. Re-read file with `rider-official:get_file_text_by_path` (MUST pass `projectPath` when known).
5. If edit fails, execute Failure Protocol.

### Playbook B — Frontend File Edit (WebStorm)

1. Discover tools for frontend edit action.
2. Read target file with `webstorm-official:get_file_text_by_path` (MUST pass `projectPath` when known).
3. Apply targeted change with `webstorm-official:replace_text_in_file` (MUST pass `projectPath` when known).
4. Re-read file with `webstorm-official:get_file_text_by_path` (MUST pass `projectPath` when known).
5. If edit fails, execute Failure Protocol.

### Playbook C — Commit Flow (Rider Convention)

1. Discover VCS tools.
2. Inspect changes with `rider-official:get_vcs_changes` (MUST pass `projectPath` when known).
3. Stage with `rider-official:vcs_stage_files` (MUST pass `projectPath` when known).
4. Commit with `rider-official:vcs_commit` (MUST pass `projectPath` when known).
5. Verify via `rider-official:get_vcs_log` (MUST pass `projectPath` when known).
6. If commit fails, execute Failure Protocol.

### Playbook D — Commit Flow (WebStorm Convention)

1. Discover VCS tools.
2. Inspect changes with `webstorm-official:get_vcs_changes` (MUST pass `projectPath` when known).
3. Stage with `webstorm-official:vcs_stage_files` (MUST pass `projectPath` when known).
4. Commit with `webstorm-official:vcs_commit` (MUST pass `projectPath` when known).
5. Verify via `webstorm-official:get_vcs_log` (MUST pass `projectPath` when known).
6. If commit fails, execute Failure Protocol.

### Playbook E — Text Search + Refine

1. Discover search tools for active IDE server.
2. Run `search_in_files_by_text` (MUST pass `projectPath` when known).
3. If noisy, run `search_in_files_by_regex` with scope refinement (MUST pass `projectPath` when known).
4. Read selected file with `get_file_text_by_path` (MUST pass `projectPath` when known).
5. Edit with `replace_text_in_file` (MUST pass `projectPath` when known) and re-read to verify.

### Playbook F — New File Then Commit

1. Discover file + VCS tools.
2. Create file with `create_new_file` (MUST pass `projectPath` when known).
3. Verify file by reading it (MUST pass `projectPath` when known).
4. Stage file via `vcs_stage_files` (MUST pass `projectPath` when known).
5. Commit via `vcs_commit` (MUST pass `projectPath` when known).
6. Verify in `get_vcs_log` (MUST pass `projectPath` when known).

### Playbook G — Backend Search-to-Commit (Rider)

1. Discover Rider search/edit/VCS tools.
2. Run `rider-official:search_in_files_by_text` (MUST pass `projectPath` when known).
3. Read target with `rider-official:get_file_text_by_path` (MUST pass `projectPath` when known).
4. Edit with `rider-official:replace_text_in_file` (MUST pass `projectPath` when known).
5. Verify edit by re-reading file.
6. Stage with `rider-official:vcs_stage_files` (MUST pass `projectPath` when known).
7. Commit with `rider-official:vcs_commit` (MUST pass `projectPath` when known).
8. Verify with `rider-official:get_vcs_log` (MUST pass `projectPath` when known).

### Playbook H — Frontend Search-to-Commit (WebStorm)

1. Discover WebStorm search/edit/VCS tools.
2. Run `webstorm-official:search_in_files_by_text` (MUST pass `projectPath` when known).
3. Read target with `webstorm-official:get_file_text_by_path` (MUST pass `projectPath` when known).
4. Edit with `webstorm-official:replace_text_in_file` (MUST pass `projectPath` when known).
5. Verify edit by re-reading file.
6. Stage with `webstorm-official:vcs_stage_files` (MUST pass `projectPath` when known).
7. Commit with `webstorm-official:vcs_commit` (MUST pass `projectPath` when known).
8. Verify with `webstorm-official:get_vcs_log` (MUST pass `projectPath` when known).

### Playbook I — Missing Tool Recovery (MCP-Only)

1. Run broad discovery query for intended action.
2. Run server-scoped discovery query.
3. Run action-scoped discovery query.
4. Run sanity discovery with `include_stats=true`.
5. Run `mcpproxy_upstream_servers` and inspect health for target server.
6. Run `mcpproxy_quarantine_security` for target server/tool state.
7. Retry discovery with exact likely tool names.
8. If still failing, execute Failure Protocol and STOP (no local fallback).

### Playbook J — Wrapper Compliance Check

1. Discover tool with `mcpproxy_retrieve_tools`.
2. Capture exact `call_with` value from discovery result.
3. Use matching wrapper only:
   - `call_tool_read` → `mcpproxy_call_tool_read`
   - `call_tool_write` → `mcpproxy_call_tool_write`
   - `call_tool_destructive` → `mcpproxy_call_tool_destructive`
4. If tool name semantics conflict with `call_with`, follow `call_with`.
5. Include intent metadata and execute.

### Playbook K — Rider Unit Tests (rider-official only)

Use this when asked to run/debug .NET unit tests through Rider MCP. Do NOT use removed/third-party IDE automation servers.

1. Discover Rider test/run tools, then call `rider-official:get_mcp_companion_overview` with `projectPath`.
2. Call `rider-official:list_run_configurations` and inspect the target with `rider-official:get_run_configuration_xml`.
3. If XML is `type="DotNetProject"`, treat it as a project launch config, not a native Rider Unit Test session. `get_test_results` may stay empty.
4. For native Unit Test sessions, navigate to the test method/class with `rider-official:navigate_to`, then preflight run-point discovery:
   `rider-official:get_run_configurations` with `filePath` and `projectPath`.
5. Proceed with context actions only when `runPoints` is non-empty:
   - run: `rider-official:execute_ide_action` with `actionId="RiderUnitTestRunContextAction"`
   - debug: `rider-official:execute_ide_action` with `actionId="RiderUnitTestDebugContextAction"`
   - poll: `rider-official:get_test_results`, `rider-official:get_console_output`
6. If `runPoints` is empty, do not claim native Rider test execution is working. For TUnit/Microsoft Testing Platform projects this can happen. Use project run/debug config for debugger access, or report that structured Rider Unit Test results are unavailable through current MCP.
7. For project-level debugging, use `rider-official:debug_run_configuration`, then inspect with `rider-official:get_debug_variables`. Resume/stop with `execute_ide_action` IDs `Resume` and `Stop`.
8. If first-chance exceptions pause tests, inspect `$exception`, `get_open_editors`, and source with `get_file_text_by_path`; use `StopOnException`, `OpenExceptionSettings`, `Resume`, or `Stop` actions as needed.

Useful Rider unit-test action IDs discovered by `execute_ide_action(search="Unit Tests")`:
`RiderUnitTestRunContextAction`, `RiderUnitTestDebugContextAction`, `RiderUnitTestRunSolutionAction`, `RiderUnitTestRunContextTwAction`, `RiderUnitTestDebugContextTwAction`, `RiderUnitTestNavigateToExplorerAction`, `RiderUnitTestNavigateToSessionAction`, `RiderUnitTestSessionAbortAction`, `RiderUnitTestSessionClearResultAction`.

---

## Failure Protocol (No Fallback)

When MCP execution fails, you MUST STOP and output this exact 5-part report:

1. **Discovery Queries:** list exact `mcpproxy_retrieve_tools` queries used.
2. **Server Health:** summarize relevant entries from `mcpproxy_upstream_servers`.
3. **Security Status:** summarize relevant `mcpproxy_quarantine_security` state.
4. **Failing Tool Evidence:** include exact `server:tool`, wrapper, and raw error text.
5. **Next MCP-Only Recovery Step:** specify one of auth / restart / enable / unquarantine / narrower rediscovery.

You MUST NOT switch to native/local tools.

---

## Common Mistakes (Blocked by Severity)

### BLOCKER

- Using local/native tools as fallback
- Calling tools without discovery
- Ignoring discovered `call_with`
- Missing intent metadata

### MAJOR

- Claiming MCP unavailable without health + quarantine checks
- Using ambiguous project selection when `projectPath` is known
- Treating one sparse query as definitive failure

### MINOR

- Skipping `mcpproxy_read_cache` for truncated responses
- Using broad discovery queries without narrowing passes

---
