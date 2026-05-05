---
name: only-use-mcp-tools
description: ESSENTIAL, ALWAYS LOAD FIRST in every session. Enforce MCP-proxy-only execution for IDE, GitHub, browser, docs, APIs, automation, and any external capability. Use this skill whenever a task could involve tools, integrations, remote services, or MCP-backed actions—even if the user does not explicitly mention MCP. Require mcpproxy tool discovery, exact tool-name routing, correct call_with wrapper usage, and mandatory intent metadata on every proxied call.
---

# MCP Tools Mandatory (Global)

## Non-Negotiable Purpose

This skill is **mandatory for every task** and must be loaded first.

It enforces one invariant:

> **If a capability can be accessed through MCP, it must be accessed through `mcpproxy`.**

This includes (but is not limited to):

- IDE and code intelligence
- GitHub and remote repository operations
- Browser automation
- API calls
- Documentation lookup
- Service introspection
- Runtime/debug tooling
- Any third-party integration exposed through MCP

---

## Hard Enforcement Rules

1. **Always use `mcpproxy` first.**
2. **Never guess tool names.**
3. **Always run `mcpproxy_retrieve_tools` before any upstream MCP call.**
4. **Use the exact discovered `name` value (`server:tool`) from retrieval output.**
5. **Use the exact discovered `call_with` route as source of truth** (this overrides naming heuristics):
   - `mcpproxy_call_tool_read`
   - `mcpproxy_call_tool_write`
   - `mcpproxy_call_tool_destructive`
6. **Every proxy call must include**:
   - `intent_reason`
   - `intent_data_sensitivity` (`public` | `internal` | `private` | `unknown`)
7. **If response is truncated, continue with `mcpproxy_read_cache`.**
8. **If discovery results are sparse, retry with narrower and more specific queries.**
9. **Do not bypass proxy due to one failed/sparse lookup.**
10. **Fallback to non-MCP/native tools is exception-only and must be explicitly justified in user-visible text.**

---

## Mandatory Startup Sequence (Every Task)

Before meaningful execution:

1. Run `mcpproxy_retrieve_tools` for the immediate capability needed.
2. If availability is uncertain, check `mcpproxy_upstream_servers`.
3. If security restrictions are suspected, inspect with `mcpproxy_quarantine_security`.
4. Execute only through the wrapper specified by discovered `call_with`.

---

## Required Routing Protocol

For each intended action:

1. **Discover**  
   Call `mcpproxy_retrieve_tools` with a focused query.

2. **Select**  
   Choose one returned tool entry and copy its exact `server:tool` name.

3. **Route**  
   Invoke through the matching wrapper from `call_with`:
   - Read-only → `mcpproxy_call_tool_read`
   - State-modifying → `mcpproxy_call_tool_write`
   - Irreversible/high-risk → `mcpproxy_call_tool_destructive`

4. **Annotate intent**  
   Set:
   - `intent_reason`: concrete reason this call is needed now
   - `intent_data_sensitivity`: narrowest honest classification

5. **Handle truncation**  
   If truncated, paginate with `mcpproxy_read_cache`.

---

## Discovery Retry Policy (Strict)

If discovery is weak, retry with smaller, explicit queries. Examples:

- `github get file contents`
- `jetbrains list files in project`
- `playwright click element by text`
- `context7 resolve library id`

Do not treat one broad failed query as evidence that tooling is unavailable.

---

## Security & Risk Policy

1. **Default to read operations.**
2. **Writes require explicit task need.**
3. **Destructive actions require explicit user intent and clear risk acknowledgement.**
4. Respect `session_risk` and tool annotations (`destructiveHint`, `openWorldHint`, `readOnlyHint`).
5. If server/tool is quarantined, inspect via `mcpproxy_quarantine_security` before declaring it unusable.

### High-Risk Session Gating

If `session_risk.level` is `high` or `lethal_trifecta=true`:

1. Prefer read-only discovery and inspection first.
2. Minimize open-world and write operations unless necessary for user-requested outcome.
3. Require explicit user confirmation before destructive operations.
4. State why the selected operation is necessary and proportionate.

---

## Fallback Policy (Exception Only)

You may use non-proxy/native alternatives only if all are true:

1. Proxy discovery and retries were attempted.
2. Relevant upstream availability/security state was checked.
3. Proxy path is actually unavailable or insufficient for required output.
4. You explicitly state:
   - why proxy could not satisfy the need,
   - what fallback is used,
   - why fallback is safe.

No silent fallback is allowed.

---

## Strict Mode: Violation Recovery

If any rule in this skill is violated:

1. Stop further execution immediately.
2. Acknowledge the violation briefly and explicitly.
3. Re-run the flow correctly:
   - `mcpproxy_retrieve_tools`
   - exact discovered tool selection
   - wrapper chosen by `call_with`
   - required intent metadata
4. Continue only after compliant state is restored.

Do not continue with mixed compliant/non-compliant execution.

---

## Common Violations (Do Not Do)

- Calling upstream tools directly without discovery
- Guessing `server:tool` names
- Ignoring `call_with` and picking wrapper manually
- Omitting `intent_reason` or `intent_data_sensitivity`
- Switching to native file/system tooling too early
- Treating sparse results as definitive failure
- Performing destructive actions without explicit user request
- Falling back without stating why proxy path failed

---

## Compliance Checklist (Before Completion)

- [ ] Skill loaded at task start
- [ ] Tools discovered via `mcpproxy_retrieve_tools`
- [ ] Exact discovered tool names used
- [ ] Correct wrapper used per `call_with`
- [ ] Intent fields provided on all proxy calls
- [ ] Truncation handled with `mcpproxy_read_cache` (if needed)
- [ ] Fallback (if any) justified in user-visible text
- [ ] Any policy violation recovered via Strict Mode
