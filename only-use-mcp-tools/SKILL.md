---
name: only-use-mcp-tools
description: Use when MCP-backed tools, IDE servers, repository automation, or external integrations are available and local fallback must be blocked.
---

# MCP Tools Mandatory (Global)

## Core Invariant

If a capability is available through MCP, it MUST be executed through `mcpproxy`.

---

## First 30 Seconds (Startup Checklist)

1. Run `mcpproxy_retrieve_tools` once for the immediate action, unless an exact known-good tool for this session/capability is already cached below.
2. Select exact returned `server:tool` and cache its wrapper (`call_tool_read/write/destructive`) for reuse.
3. Reuse cached exact tool names for the same server/capability; do not repeatedly rediscover stable tools.
4. Include `intent_reason` and `intent_data_sensitivity` on every proxied call.
5. If cached tools fail or tools are missing/unclear, run health and quarantine checks before any further action.

---

## Operational Contract (Fail-Closed)

1. You MUST use `mcpproxy` first.
2. You MUST NOT guess tool names.
3. You MUST run `mcpproxy_retrieve_tools` before first use of an unknown capability.
4. You MAY reuse exact known-good tool names from this skill index, project skills, or earlier successful calls in the same session without rediscovery.
5. You MUST use exact discovered/cached `server:tool` names.
6. You MUST route by discovered/cached `call_with` only:
   - `call_tool_read` → `mcpproxy_call_tool_read`
   - `call_tool_write` → `mcpproxy_call_tool_write`
   - `call_tool_destructive` → `mcpproxy_call_tool_destructive`
7. Every proxy call MUST include:
   - `intent_reason`
   - `intent_data_sensitivity` (`public|internal|private|unknown`)
8. You MUST continue truncated responses via `mcpproxy_read_cache`.
9. You MUST NOT use local/native fallback tools.
10. You MUST NOT stage or commit changes unless the user explicitly asks for a commit/staging operation in the current task.

---

## Discovery Policy (Deterministic, Cached)

Use discovery for unknown capabilities, not for every call. Maintain a session-local cache of exact `server:tool` names and wrappers after successful discovery, direct-call success, or tool/security inspection.

When capability is known from the cache/index, call it directly with the cached wrapper.

When capability is uncertain, run 3 narrowing discovery queries:
1. Broad capability query
2. Server-scoped query
3. Action-scoped query

If still unclear, run one sanity pass with `include_stats=true` and retry with explicit likely names.
You MUST NOT infer unavailability from one sparse query.

### Discovery False-Negative Guardrail (Mandatory)

If all 3 narrowing discovery queries return empty/irrelevant results, but the target server is healthy and you have a known exact tool name from this skill hard-coded index, you MUST attempt a direct MCP call to that exact `server:tool` before declaring blockage.

Required sequence before any "blocked" claim:
1. Complete the 3 narrowing discovery queries (+ optional `include_stats=true` sanity pass).
2. Confirm server health via `mcpproxy_upstream_servers`.
3. Confirm quarantine status via `mcpproxy_quarantine_security`.
4. Attempt at least one direct call to a known exact tool name (for example `rider-official:get_file_text_by_path` with `projectPath` + path args).
5. If the direct call fails, include the raw tool error in Failure Protocol.

Do NOT treat retrieval ranking misses as proof that tools are unavailable. Retrieval can be noisy; direct invocation of known exact tool names is the required tie-breaker. After a direct invocation succeeds, cache that tool and wrapper for the rest of the session.

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

`rider-official` and `webstorm-official` share similar tool layout for many read/edit/search actions, but they are not interchangeable when project-specific guidance assigns ownership. If AGENTS.md or a loaded project skill says backend = Rider and frontend = WebStorm, that routing overrides generic equivalence. Only swap server prefixes after retry, health, quarantine checks, and an explicit note that the preferred IDE MCP is unavailable.

### projectPath Rule

When `projectPath` is known, you MUST pass it on Rider/WebStorm tool calls.

Repository-root convention for this environment:
- Projects are one level deep under `/mnt/PROJECTS`.
- Default project root format: `/mnt/PROJECTS/<repo-name>`.
- If repo name is known, you MUST derive and pass `projectPath` using this format.

---

## Common Actions (Hard-Coded Tool Index)

Run one discovery call first per session/capability, then cache and reuse these stable names:

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

### Purpose-Built .NET Test Tools

When `dotnet-test-mcp` is healthy, use it for .NET test execution before Rider run configurations, IDE test sessions, or any terminal-style fallback. After one discovery/health check, these exact tool names may be cached and invoked directly.

Important: treat these tools as read-wrapper tools unless discovery explicitly returns a different `call_with` value. In the current environment, direct calls to these exact names are known-good through `mcpproxy_call_tool_read`. Always pass `workingDirectory` when the MCP server was launched outside the target repo or when working in a worktree.

| Action | Tool | Wrapper | Key args |
|---|---|---|---|
| List test projects | `dotnet-test-mcp:list_test_projects` | `mcpproxy_call_tool_read` | `workingDirectory` optional |
| List tests summary | `dotnet-test-mcp:list_tests_summary` | `mcpproxy_call_tool_read` | `prefix`, `workingDirectory` optional |
| Run one test | `dotnet-test-mcp:run_single_test` | `mcpproxy_call_tool_read` | `qualifiedMethodName`, `project`, `workingDirectory`, `includeStackTrace` |
| Run class tests | `dotnet-test-mcp:run_all_tests_in_class` | `mcpproxy_call_tool_read` | `className`, `project`, `workingDirectory`, `includeStackTrace` |
| Run project tests | `dotnet-test-mcp:run_all_tests_for_project` | `mcpproxy_call_tool_read` | `projectPath`, `workingDirectory`, `includeStackTrace` |
| Run all tests | `dotnet-test-mcp:run_all_tests` | `mcpproxy_call_tool_read` | `workingDirectory`, `includeStackTrace` optional |

For TUnit/Microsoft Testing Platform projects, `run_single_test` and `run_all_tests_in_class` should use MTP `--treenode-filter` internally. If they fail with a generic invocation error, first reproduce with `list_tests_summary`, then patch against the exact command/assertion failure before falling back to broader project runs.

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

Use this playbook ONLY when the user explicitly asks to commit/stage changes. Do not infer commit intent from finishing edits, plans, tests, or verification.

1. Discover VCS tools.
2. Inspect changes with `rider-official:get_vcs_changes` (MUST pass `projectPath` when known).
3. Stage with `rider-official:vcs_stage_files` (MUST pass `projectPath` when known).
4. Commit with `rider-official:vcs_commit` (MUST pass `projectPath` when known).
5. Verify via `rider-official:get_vcs_log` (MUST pass `projectPath` when known).
6. If commit fails, execute Failure Protocol.

### Playbook D — Commit Flow (WebStorm Convention)

Use this playbook ONLY when the user explicitly asks to commit/stage changes. Do not infer commit intent from finishing edits, plans, tests, or verification.

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

### Playbook F — New File Creation

Create and verify files only. Stage/commit steps require an explicit current user request to stage or commit.

1. Discover file tools.
2. Create file with `create_new_file` (MUST pass `projectPath` when known).
3. Verify file by reading it (MUST pass `projectPath` when known).
4. Stop after readback verification unless the user explicitly requested staging or committing.
5. If explicit commit intent exists, switch to Playbook C or D.

### Playbook G — Backend Search-to-Verify (Rider)

1. Discover Rider search/edit/VCS tools.
2. Run `rider-official:search_in_files_by_text` (MUST pass `projectPath` when known).
3. Read target with `rider-official:get_file_text_by_path` (MUST pass `projectPath` when known).
4. Edit with `rider-official:replace_text_in_file` (MUST pass `projectPath` when known).
5. Verify edit by re-reading file.
6. Stop after readback verification unless the user explicitly requested staging or committing.
7. If explicit commit intent exists, switch to Playbook C.

### Playbook H — Frontend Search-to-Verify (WebStorm)

1. Discover WebStorm search/edit/VCS tools.
2. Run `webstorm-official:search_in_files_by_text` (MUST pass `projectPath` when known).
3. Read target with `webstorm-official:get_file_text_by_path` (MUST pass `projectPath` when known).
4. Edit with `webstorm-official:replace_text_in_file` (MUST pass `projectPath` when known).
5. Verify edit by re-reading file.
6. Stop after readback verification unless the user explicitly requested staging or committing.
7. If explicit commit intent exists, switch to Playbook D.

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

1. If `dotnet-test-mcp` is healthy and the task is ordinary .NET test execution, prefer the purpose-built `dotnet-test-mcp:*` tools from the hard-coded index and skip Rider run/test sessions. Use Rider here only for IDE-specific debugging, run configuration inspection, or when structured IDE state is required.
2. Discover Rider test/run tools, then call `rider-official:get_mcp_companion_overview` with `projectPath`.
3. Call `rider-official:list_run_configurations` and inspect the target with `rider-official:get_run_configuration_xml`.
4. If XML is `type="DotNetProject"`, treat it as a project launch config, not a native Rider Unit Test session. `get_test_results` may stay empty.
5. For native Unit Test sessions, navigate to the test method/class with `rider-official:navigate_to`, then preflight run-point discovery:
   `rider-official:get_run_configurations` with `filePath` and `projectPath`.
6. Proceed with context actions only when `runPoints` is non-empty:
   - run: `rider-official:execute_ide_action` with `actionId="RiderUnitTestRunContextAction"`
   - debug: `rider-official:execute_ide_action` with `actionId="RiderUnitTestDebugContextAction"`
   - poll: `rider-official:get_test_results`, `rider-official:get_console_output`
7. If `runPoints` is empty, do not claim native Rider test execution is working. For TUnit/Microsoft Testing Platform projects this can happen. Use project run/debug config for debugger access, or report that structured Rider Unit Test results are unavailable through current MCP.
8. For project-level debugging, use `rider-official:debug_run_configuration`, then inspect with `rider-official:get_debug_variables`. Resume/stop with `execute_ide_action` IDs `Resume` and `Stop`.
9. If first-chance exceptions pause tests, inspect `$exception`, `get_open_editors`, and source with `get_file_text_by_path`; use `StopOnException`, `OpenExceptionSettings`, `Resume`, or `Stop` actions as needed.

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
- Staging or committing without explicit current user request

### MAJOR

- Claiming MCP unavailable without health + quarantine checks
- Using ambiguous project selection when `projectPath` is known
- Treating one sparse query as definitive failure

### MINOR

- Skipping `mcpproxy_read_cache` for truncated responses
- Using broad discovery queries without narrowing passes

---
