---
name: basic-start
description: Use certain tools before starting to work.
---

## Mode Detection: Software Development vs General Scripting

Before starting any task that involves code, classify the mode.

| Mode | Description | Tooling |
|---|---|---|
| **Software Development** | Working on a specific repository — coding, refactoring, debugging, building, testing. Has a project directory with tracked source files. | `codebase-memory-mcp` for code discovery, JetBrains IDE + MCP tools |
| **General Scripting** | Shell scripts, network scripts, config files, system admin, one-off automation, homelab management, pfSense work. Not tied to a specific software project. | Standard bash/read/write tools, `ffgrep` / `fffind`, wiki |

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
   /home/jason/.local/share/JetBrains/Toolbox/apps/phpstorm/bin/phpstorm /path/to/project &
   ```
   Wait a few seconds for the IDE to register its MCP server, then verify connection.

4. **Call JetBrains MCP tools** through the MCP server:
   ```
   mcp({ tool: "rider_official_search_symbol", args: '{"query": "SomeClass"}' })
   mcp({ tool: "webstorm_official_get_file_problems", args: '{"filePath": "src/index.ts"}' })
   ```

5. **Follow the JetBrains workflow** from the loaded skills: edit → check problems → fix → repeat.

> **PHP Storm note:** There is no phpstorm MCP server configured. For PHP/WordPress work, open PHP Storm via bash for manual use, but do all agent-driven file operations (read, edit, search) through standard tools.

### General Scripting — Standard Tooling

When mode is **General Scripting**, do not open a JetBrains IDE. Use standard tools:

- `bash` — run shell commands, scripts
- `read` / `write` / `edit` — file operations
- `ffgrep` / `fffind` — file content/path search (**only in this mode**; never use for software development)
- `web_fetch` / `web_search` — research
- `wiki_wikijs_*` — store/retrieve knowledge via wiki

The wiki (`wiki-knowledge-manager` skill) is the primary knowledge base for non-dev work.

## Wiki Knowledge Capture

For any **non-programming task** — research, configuration, system administration, homelab management, learning sessions — after completing the task, load the `wiki-knowledge-manager` skill and store lessons, tips, notes, or findings to the wiki. The wiki is the primary knowledge base for non-dev work.

Skill path: `/mnt/PROJECTS/skills/wiki-knowledge-manager/SKILL.md`

## ⛔ No Decompilation — Always Search Open Source First

**Decompiling is NEVER allowed.** Under no circumstances may you decompile, disassemble, or reverse-engineer binaries, assemblies, DLLs, `.class` files, `.pyc` files, `.so` / `.dylib` shared objects, or any compiled artifact — regardless of the mode (software development or general scripting).

### The Rule (both modes)

When you need to understand how a library, tool, or dependency works internally:

1. **Identify the package name and version** from the project files (`.csproj`, `package.json`, `requirements.txt`, `go.mod`, `Cargo.toml`, `composer.json`, `Gemfile`, `PKGBUILD`, etc.)
2. **Find the official open-source repository**:
   - Search GitHub: `github_search_repositories(query="<package-name> language:<lang>")` or `github_search_code(query="<signature-or-symbol>")`
   - Check the package metadata (NuGet.org, npmjs.com, PyPI, crates.io, etc.) for a "Source repository" or "Repository" link
   - Use `web_fetch` on the package registry page to find the source URL if needed
3. **Search the source code directly on GitHub** using `github_search_code`, `gitmcp_docs_search_generic_code`, or `gitmcp_docs_fetch_generic_documentation`
4. **Read the source** — every open-source library's true behavior is in its public source repository. There is no reason to decompile what is already available as source.

### What to NEVER do

| Forbidden Action | Do This Instead |
|---|---|
| `ildasm` / `dotnet-ildasm` / ILSpy / dnSpy on a .NET assembly | `github_search_code` on the package's source repo |
| `javap` / CFR / Procyon on `.class` files | `github_search_code` on the library's source repo |
| `uncompyle6` / `decompyle3` on `.pyc` | Find the repo on GitHub/PyPI and read the `.py` source |
| `strings` / `objdump` / `ghidra` / `radare2` on binaries | Find the source repo and read the actual code |
| Any "decompile", "disassemble", "reverse engineer" tool or workflow | Search open-source repos for the real source |

### If no source repo exists (closed-source / proprietary)

- Use **official documentation** (`context7_query-docs`, `microsoft_learn_mcp_microsoft_docs_search`, `web_fetch` on docs site)
- Use the library's **public API surface** only — never peek inside
- If the internal behavior is undocumented and critical, **ask the user** — do not decompile

## Code Discovery: Always Use `codebase-memory-mcp` First

> ⛔ **Never use `ffgrep` or `fffind` for software development.** Those are for general scripting only.

For any software development task, `codebase-memory-mcp` is the **mandatory first choice** for code discovery. It provides structural context (call graphs, symbols, architecture) that raw text search cannot match and is extremely fast.

### Priority Order for Code Discovery

1. `codebase-memory-mcp:search_graph` — find functions, classes, routes, variables by pattern
2. `codebase-memory-mcp:trace_path` — trace who calls a function or what it calls
3. `codebase-memory-mcp:get_code_snippet` — read specific function/class source code
4. `codebase-memory-mcp:query_graph` — run Cypher queries for complex patterns
5. `codebase-memory-mcp:get_architecture` — high-level project summary
6. `codebase-memory-mcp:search_code` — graph-augmented grep for exact strings/literals

### Fallback Chain (only if `codebase-memory-mcp` is unavailable or insufficient)

1. IDE MCP search (rider-official / webstorm-official symbol/reference tools)
2. `ffgrep` / `fffind` — **only as last resort** when neither codebase-memory-mcp nor IDE MCP is available

### When Code-Memory-MCP Doesn't Cover It

| Situation | Use |
| --- | --- |
| Searching non-code files (Dockerfiles, shell scripts, configs) | MCP IDE search, or `read`/`list` for targeted paths |
| Reading known file contents | Native `read` |
| Graph tools return insufficient results | Narrow with `file_pattern`/`path_filter`; if still insufficient, fall back to IDE MCP search |

### Usage Examples

- Find a handler: `codebase-memory-mcp:search_graph(query="OrderHandler", project="<project-root>")`
- Who calls it: `codebase-memory-mcp:trace_path(function_name="OrderHandler", direction="inbound", project="<project-root>")`
- Read source: `codebase-memory-mcp:get_code_snippet(qualified_name="pkg/orders.OrderHandler", project="<project-root>")`
