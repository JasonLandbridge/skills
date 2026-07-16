---
name: basic-start
description: Use certain tools before starting to work.
---

## Mode Detection: Software Development vs General Scripting

Before starting any task that involves code, classify the mode.

| Mode | Description | Tooling |
|---|---|---|
| **Software Development** | Working on a specific repository — coding, refactoring, debugging, building, testing. Has a project directory with tracked source files. | JetBrains IDE + MCP tools |
| **General Scripting** | Shell scripts, network scripts, config files, system admin, one-off automation, homelab management, pfSense work. Not tied to a specific software project. | Standard bash/read/write/grep tools, wiki |

### Software Development — IDE Selection

When mode is **Software Development**, load the `jetbrains-skill` and `jetbrains-coding` skills, then pick the IDE:

| Project Type | IDE | MCP Server | Start Command |
|---|---|---|---|
| C# / .NET (solution files, .csproj) | Rider | `rider-official` | `rider <project-dir> &` |
| PHP / WordPress | PHP Storm | (no MCP server — use bash) | `/home/jason/.local/share/JetBrains/Toolbox/apps/phpstorm/bin/phpstorm <project-dir> &` |
| Web / JS / TS / Vue / React | WebStorm | `webstorm-official` | `webstorm <project-dir> &` |
| Anything else / fallback | WebStorm | `webstorm-official` | `webstorm <project-dir> &` |

#### Step-by-step

1. **Detect the project type** from the repo contents (.sln, .csproj, package.json, wp-content, composer.json, etc.)

2. **Check if the IDE MCP server is already connected:**
   ```
   mcp({ server: "rider-official" })
   mcp({ server: "webstorm-official" })
   ```
   If connected (shows tool list), skip step 3.

3. **Start the IDE via bash** if its MCP server isn't connected:
   ```bash
   # Rider (C#)
   rider /path/to/project &

   # WebStorm (web)
   webstorm /path/to/project &

   # PHP Storm (WordPress/PHP — no MCP server, opens for manual use)
   phpstorm /path/to/project &
   ```
   Wait a few seconds for the IDE to register its MCP server, then verify connection.

4. **Call JetBrains MCP tools** through the MCP server:
   ```
   mcp({ tool: "rider_official_search_symbol", args: '{"query": "SomeClass"}' })
   mcp({ tool: "webstorm_official_get_file_problems", args: '{"filePath": "src/index.ts"}' })
   ```

5. **Follow the JetBrains workflow** from the loaded skills: edit → check problems → fix → repeat.

### General Scripting — Standard Tooling

When mode is **General Scripting**, do not open a JetBrains IDE. Use standard tools:

- `bash` — run shell commands, scripts
- `read` / `write` / `edit` — file operations
- `ffgrep` / `fffind` — search files
- `web_fetch` / `web_search` — research
- `wiki_wikijs_*` — store/retrieve knowledge via wiki

The wiki (`wiki-knowledge-manager` skill) is the primary knowledge base for non-dev work.

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
