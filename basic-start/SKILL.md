---
name: basic-start
description: Use certain tools before starting to work.
---

## Wiki Knowledge Capture

For any **non-programming task** — research, configuration, system administration, homelab management, learning sessions — after completing the task, load the `wiki-knowledge-manager` skill and store lessons, tips, notes, or findings to the wiki. The wiki is the primary knowledge base for non-dev work.

## Codebase Knowledge Graph Guidance

When `codebase-memory-mcp` tools are available, they are the recommended first choice for structural code discovery. They provide context that raw text search cannot match and are extremely fast.

### Priority Order for Code Discovery

1. `codebase-memory-mcp:search_graph` — find functions, classes, routes, variables by pattern
2. `codebase-memory-mcp:trace_path` — trace who calls a function or what it calls
3. `codebase-memory-mcp:get_code_snippet` — read specific function/class source code
4. `codebase-memory-mcp:query_graph` — run Cypher queries for complex patterns
5. `codebase-memory-mcp:get_architecture` — high-level project summary

### When to Use Other Tools

| Situation | Use |
| --- | --- |
| Searching for exact string literals, config values, error messages | `codebase-memory-mcp:search_code` (graph-augmented grep) |
| Searching non-code files (Dockerfiles, shell scripts, configs) | MCP IDE search, or `read`/`list` for targeted paths |
| Reading known file contents | Native `read` (when permitted) |
| Graph tools return insufficient results | Narrow with `file_pattern`/`path_filter`; if still insufficient, use IDE MCP search or `read`/`list` with targeted paths |
| `codebase-memory-mcp` is not available | Use IDE MCP search, LSP symbols/references, or `read`/`list` |

### Usage Examples

- Find a handler: `codebase-memory-mcp:search_graph(query="OrderHandler", project="<project-root>")`
- Who calls it: `codebase-memory-mcp:trace_path(function_name="OrderHandler", direction="inbound", project="<project-root>")`
- Read source: `codebase-memory-mcp:get_code_snippet(qualified_name="pkg/orders.OrderHandler", project="<project-root>")`
