---
name: mcp-tool-registry
description: Use when an MCP-only workflow needs exact cached server:tool names, mcpproxy wrappers, argument templates, guard notes, or a known-tool retry after mcpproxy retrieval misses a stable tool.
---

# MCP Tool Registry

## Purpose

This skill is the exact-tool cache for MCP-augmented agents. It provides verified `server:tool` names, cached mcpproxy wrapper routes, minimal argument templates, and guard notes for stable MCP tools.

Use it with `prefer-mcp-for-nonnative-tools`. The policy skill decides **when MCP should be used** vs native OpenCode tools and how to handle MCP failures. This registry helps agents avoid guessing when MCP use is required.

## Relationship to `prefer-mcp-for-nonnative-tools`

- `prefer-mcp-for-nonnative-tools` should load this skill before first MCP-backed action.
- `prefer-mcp-for-nonnative-tools` is the policy source of truth: when MCP vs native tools should be used, discovery, health checks, quarantine checks, intent metadata, user-confirmation gates, and no fallback to disabled tools.
- This registry is the lookup source of truth for known stable exact MCP tools and first-call argument shapes.
- Fresh `mcpproxy_retrieve_tools` output wins over this registry for current wrapper/schema details.
- Successful calls in the current session may be cached, but never invent a tool name or argument from patterns.

## Registry File

Primary data lives in:

```text
mcp-tool-registry.yaml
```

Each entry uses this shape:

```yaml
"server:tool":
  call: read | write | destructive
  risk: optional-risk-category
  args: { minimal: "<placeholder-template>" }
  guard: "When to require extra care or explicit user intent."
  verified:
    source: retrieve_tools | direct_call | manual_schema_check
    notes: "Optional short freshness note."
```

`call` is only the cached mcpproxy wrapper route:

| `call` | Wrapper |
| --- | --- |
| `read` | `mcpproxy_call_tool_read` |
| `write` | `mcpproxy_call_tool_write` |
| `destructive` | `mcpproxy_call_tool_destructive` |

**Important:** `call` is not a safety label. Some tools that edit files, stage commits, execute code, or mutate browser/remote state may currently route through `call: read`. Always obey `risk`, `guard`, and `prefer-mcp-for-nonnative-tools` safety gates.

## Fast Lookup Workflow

1. Start from the policy/routing decision in `prefer-mcp-for-nonnative-tools`.
2. Open `mcp-tool-registry.yaml`.
3. Find the exact `server:tool` key for the capability.
4. Copy the `args` template and replace every `<placeholder>`.
5. Use the wrapper indicated by `call`, unless fresh discovery says otherwise.
6. Add `intent_reason` and `intent_data_sensitivity` to the mcpproxy call.
7. Read and obey `risk`/`guard` before edits, process execution, VCS operations, browser actions, remote writes, or destructive operations.
8. Cache successful exact tool names and wrappers for the rest of the session.

## Argument Template Rules

Registry `args` are **minimum viable first-call templates**, not exhaustive schemas.

- Use the template to avoid missing common required fields.
- Add optional fields only when fresh discovery/schema confirms them or the tool has already accepted them in this session.
- Prefer safe defaults: limited output, no overwrite, no force, no push, no destructive flags.
- Replace every `<placeholder>` before calling.
- If a needed field is absent from the template, use fresh discovery to confirm the field name before adding it.
- If fresh discovery disagrees with the template, fresh discovery wins and the registry should be refreshed.

## When Retrieval Misses a Known Tool

Use this registry as a deterministic tie-breaker, not as a replacement for discovery.

Required sequence before declaring a known capability blocked:

1. Run broad, server-scoped, and action-scoped `mcpproxy_retrieve_tools` queries.
2. Check `mcp-tool-registry.yaml` for likely exact entries.
3. Check server health with `mcpproxy_upstream_servers`.
4. Check quarantine/tool state with `mcpproxy_quarantine_security`.
5. Attempt one direct call to the exact registry entry only if the server is healthy and the action is read-only or already allowed by current user intent and the registry guard.
6. If it fails, report the raw tool error through the `prefer-mcp-for-nonnative-tools` Failure Protocol.

Never use a registry tie-breaker to perform destructive, remote-write, VCS-write, browser-mutation, or code-execution actions unless the user explicitly requested that action in the current task.

## Stale Entry Recovery

When a registry entry fails because a schema, wrapper, server name, or argument changed:

1. Stop retrying the same stale call.
2. Run fresh discovery using the exact tool name plus capability words.
3. If discovery finds the tool, use the fresh `call_with` and schema.
4. If discovery does not find it, run the health/security gate.
5. Report that the registry entry appears stale and include the exact failing field/error.
6. Update YAML only after the new schema is verified through discovery or a successful direct call.

## Common Action Map

Use this table to choose likely registry keys, then use YAML for exact args and guards.

| Capability | Likely registry area |
| --- | --- |
| Backend/.NET IDE file/search/edit/VCS/run/test state | `rider-official:*` |
| Frontend/Vue/TS/JS/CSS IDE file/search/edit/VCS/run/test state | `webstorm-official:*` |
| PHP IDE file/search/edit/VCS/run/test state | `phpstorm-official:*` |
| .NET test execution outside IDE test UI | `dotnet-test-mcp:*` |
| .NET SDK/EF project operations | `dotnet-mcp:*` |
| Git status/diff/commit via dedicated server | `git-mcp-server:*` |
| GitHub repository, issue, PR, Actions, remote file operations | `github:*` |
| Browser inspection/automation through Chrome DevTools | `chrome-devtools:*` |
| Browser inspection/automation through Playwright | `playwright:*` |
| Library documentation lookup | `context7:*` |
| Codebase knowledge graph / code discovery | `codebase-memory-mcp:*` |

## Golden Discovery Queries

Use these when retrieval needs help finding stable tools:

- `rider-official backend file read edit search`
- `webstorm-official frontend file read edit search`
- `rider-official vcs changes stage commit log`
- `webstorm-official vcs changes stage commit log`
- `rider-official get_file_text_by_path replace_text_in_file`
- `webstorm-official get_file_text_by_path replace_text_in_file`
- `rider-official search_in_files_by_text search_in_files_by_regex`
- `webstorm-official search_in_files_by_text search_in_files_by_regex`
- `dotnet-test-mcp list run single class project tests`
- `dotnet-test-mcp run_single_test testName project projectPath workingDirectory`
- `github pull request issue actions file contents search code`
- `chrome-devtools console network snapshot screenshot navigate evaluate`
- `playwright browser snapshot console network navigate evaluate`
- `codebase-memory-mcp search graph trace path code snippet architecture`
- `codebase-memory-mcp search_graph query name_pattern label project`
- `codebase-memory-mcp trace_path function_name direction mode calls data_flow`

## Placeholder Rules

Replace every placeholder before calling a tool:

| Placeholder | Meaning |
| --- | --- |
| `<project-root>` | Absolute local project root, for example `/mnt/PROJECTS/<repo>` |
| `<relative-path>` | Path relative to project root |
| `<relative-directory>` | Directory relative to project root |
| `<repo-path>` | Path inside remote repository |
| `<owner>`, `<repo>`, `<branch>` | Remote repository identifiers |
| `<url>`, `<sha-or-ref>`, `<workflow-id-or-file>` | Remote/API identifiers |
| `<fully-qualified-test-name>` | Fully qualified test method name, or short method name when tool supports it |
| `<test-project.csproj>` | Test project file path relative to project root |

Do not leave placeholder values in calls. If a required value is unknown, discover it through MCP or ask the user.

## Maintenance Rules

Keep the registry small, current, and boring:

- Add only exact `server:tool` names verified against current mcpproxy schemas.
- Store the minimal first-call args that prevent common mistakes.
- Prefer placeholders over project-specific values.
- Add `risk` and `guard` whenever a tool can mutate files, run code, alter VCS, write remote state, manipulate browser state, or delete anything.
- Add `verified` only when it helps future agents understand schema freshness; avoid noisy timestamps unless actively maintained.
- Remove obsolete entries instead of preserving historical variants.
- If discovery disagrees with the registry, verify schema freshness before editing the YAML.
- Do not run `skillshare` sync/deployment commands during normal lookup, validation, or maintenance of this registry.

## Validation Checklist

After editing this skill or YAML:

1. Read both `SKILL.md` and `mcp-tool-registry.yaml` back.
2. Confirm the frontmatter description only states when to use the skill.
3. Confirm every YAML tool key is an exact `server:tool` string.
4. Confirm every tool has `call` and `args`.
5. Confirm risky tools include `risk` and `guard`.
6. Confirm direct-call retry rules block unapproved mutation/execution.
7. Confirm `prefer-mcp-for-nonnative-tools` still references this registry.
8. Do not stage or commit unless the current user explicitly asked.

## Common Mistakes

| Mistake | Fix |
| --- | --- |
| Treating this registry as permission to skip `prefer-mcp-for-nonnative-tools` | Load and obey `prefer-mcp-for-nonnative-tools`; this is only exact-tool data. |
| Assuming `call: read` means harmless | Check `risk`/`guard`; wrapper route is not safety classification. |
| Treating args as exhaustive schemas | Templates are minimum viable first calls; fresh schema wins. |
| Copying args without replacing placeholders | Replace every `<...>` value or discover/ask for it. |
| Guessing a similar tool name because the pattern looks obvious | Run discovery or add the exact verified entry first. |
| Retrying a stale registry entry repeatedly | Run exact-name discovery, use fresh schema, and mark registry refresh needed. |
| Staging, committing, pushing, deleting, executing code, or remote-writing because an entry exists | Require explicit current user intent and follow the guard. |
| Declaring MCP unavailable after one failed lookup | Follow the retrieval-miss sequence and Failure Protocol. |
