---
name: mcp-tool-registry
description: Use when an MCP-only workflow needs an exact cached server:tool name, mcpproxy wrapper, or argument template for known MCP tools, especially after /only-use-mcp-tools discovery or when retrieval misses a stable tool.
---

# MCP Tool Registry

## Purpose

This skill is a small cache of verified MCP `server:tool` names and argument shapes. It works with `/only-use-mcp-tools`; it does **not** replace that skill's discovery, health, quarantine, intent-metadata, or fail-closed rules.

Use `mcp-tool-registry.yaml` when you already know the capability/server and need a reliable exact tool name or args template without guessing.

## Contract with `/only-use-mcp-tools`

- `/only-use-mcp-tools` is the policy source of truth.
- This registry is only a lookup table for known stable tools.
- `call` means the cached mcpproxy wrapper suffix from verified `call_with`:
  - `read` → `mcpproxy_call_tool_read`
  - `write` → `mcpproxy_call_tool_write`
  - `destructive` → `mcpproxy_call_tool_destructive`
- `call` is **not** a safety label. Some tools that edit files, stage git changes, or create commits currently route through `call_tool_read`; obey the `risk`/`guard` notes and user-confirmation rules anyway.
- Do not run `skillshare` commands as part of normal registry lookup, validation, or maintenance; target sync/deployment is outside this skill.
- Verification for this skill means checking MCP schema freshness and file diagnostics for this folder, not synchronizing skill targets.
- If fresh `mcpproxy_retrieve_tools` output disagrees with this registry, current discovery wins. Update the registry only after the tool schema is verified.
- Never invent tool names or args from registry patterns.

## Lookup Workflow

1. Choose the target capability and server from `/only-use-mcp-tools` routing rules.
2. If the capability is unknown or disputed, run `mcpproxy_retrieve_tools` first. On missing/unclear results, follow the `/only-use-mcp-tools` health + quarantine recovery path.
3. Open `.skillshare/skills/mcp-tool-registry/mcp-tool-registry.yaml`.
4. Find the exact `server:tool` key.
5. Copy the `args` shape and replace placeholders such as `<project-root>`, `<relative-path>`, `<owner>`, `<repo>`, and `<branch>`.
6. Use the wrapper indicated by `call` unless fresh discovery says otherwise.
7. Include `intent_reason` and `intent_data_sensitivity` on every proxied call.
8. Read and obey `guard` before any file edit, process execution, VCS operation, remote write, browser action, or destructive operation.

## Maintenance Rules

Keep this registry small and boring:

- Add only exact `server:tool` names that have been verified against current mcpproxy schemas.
- Store only the minimal args needed for a good first call.
- Prefer placeholders over project-specific values.
- Add `risk`/`guard` when the tool can mutate files, run code, touch VCS, change remote services, or execute browser JavaScript.
- Remove obsolete entries instead of preserving historical variants.

## Common Mistakes

| Mistake | Fix |
| --- | --- |
| Treating the registry as a replacement for discovery | Use it only for known stable tools; fresh discovery wins. |
| Assuming `call: read` means harmless | `call` is the wrapper route, not the permission model; check `risk` and `guard`. |
| Copying args without replacing placeholders | Replace every `<...>` value before calling the tool. |
| Staging, committing, pushing, or remote-writing because an entry exists | Do it only when the current user explicitly requested that action. |
| Declaring MCP unavailable after one failed lookup | Follow `/only-use-mcp-tools` failure protocol, including health/quarantine checks and direct known-tool retry. |
