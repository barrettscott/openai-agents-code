# M08B: Real-World MCP Servers -- Recording Prep

## What this notebook teaches

This notebook moves from the single-server hello-world in M08A to real, practical MCP servers: the filesystem server (read/write local files) and the web fetch server (retrieve URLs as clean markdown). Students learn to combine multiple MCP servers in one agent via `mcp_servers=[fs_server, fetch_server]`, apply tool filtering with `create_static_tool_filter` for least-privilege access, and test servers before trusting them. The overarching lesson is that MCP servers are powerful but need scoping, filtering, and distrust of returned content.

## Concepts to explain on camera

- **Filesystem MCP server (`@modelcontextprotocol/server-filesystem`)**
  - **Plain English:** A standalone process that exposes local file operations (read, write, list, search) as MCP tools. You launch it with `npx` and pass a directory path to scope its access.
  - **Analogy:** Like giving someone a key to one room in your house instead of the front door -- they can do anything inside that room but nothing outside it.
  - **What students need to understand:**
    - Launched via `npx -y @modelcontextprotocol/server-filesystem <absolute-path>`
    - The path argument is the boundary -- the server cannot access anything outside it
    - Passing `/` or `~` would expose your entire system; always scope to a dedicated workspace
    - The server exposes write tools by default, so the agent can create and modify files on disk

- **Web fetch MCP server (`mcp-server-fetch`)**
  - **Plain English:** A standalone process that exposes a `fetch` tool. The agent sends a URL, the server retrieves the page, converts the HTML to clean markdown, and returns it.
  - **Analogy:** Like hiring a research assistant who reads a web page for you and hands you a typed summary instead of a printout full of ads and navigation menus.
  - **What students need to understand:**
    - Launched via `uvx mcp-server-fetch` -- requires `uv` to be installed (`pip install uv`)
    - No API key needed; it just fetches URLs
    - Returns markdown, not raw HTML, so the agent gets immediately usable text
    - Content comes from the open web -- treat it as untrusted input (prompt injection risk from M07B)

- **Tool filtering (`create_static_tool_filter`)**
  - **Plain English:** A mechanism that hides specific tools from the agent. You provide an allowlist of tool names, and the agent never sees the rest -- it literally cannot call them.
  - **Analogy:** Like removing certain buttons from a remote control -- the TV still has those features, but the person holding the remote has no way to activate them.
  - **What students need to understand:**
    - Imported from `agents.mcp` alongside `MCPServerStdio`
    - Passed as `tool_filter=create_static_tool_filter(allowed_tool_names=[...])` when creating the server
    - Filtering happens at the SDK level, before tools reach the model
    - Use it for read-only access: allow `read_file`, `list_directory`, `get_file_info` and block everything else

- **`cache_tools_list=True`**
  - **Plain English:** Tells the SDK to call `list_tools()` once at startup and cache the result, instead of asking the server for its tool list on every agent turn.
  - **Analogy:** Like reading a restaurant menu once and remembering it, instead of asking the waiter to recite it before every dish.
  - **What students need to understand:**
    - Set on every `MCPServerStdio` in this notebook
    - Reduces latency for multi-turn interactions
    - Safe when the server's tool list does not change during the session
    - If you add or remove tools dynamically, do not cache

- **`uvx`**
  - **Plain English:** A command-line tool (part of `uv`) that runs Python packages directly without installing them into your environment -- similar to what `npx` does for Node.js packages.
  - **Analogy:** Like a vending machine for Python tools -- you pick one, use it, and don't have to store it in your kitchen.
  - **What students need to understand:**
    - Installed via `pip install uv`
    - Used in this notebook to launch `mcp-server-fetch`
    - The prerequisite check cell verifies `uvx --version` before proceeding

## Key SDK pattern

The primary pattern is combining multiple MCP servers in one agent:

```python
async with (
    MCPServerStdio(
        name="Filesystem",
        params={
            "command": "npx",
            "args": ["-y", "@modelcontextprotocol/server-filesystem", str(workspace.resolve())]
        },
        cache_tools_list=True
    ) as fs_server,
    MCPServerStdio(
        name="WebFetch",
        params={
            "command": "uvx",
            "args": ["mcp-server-fetch"]
        },
        cache_tools_list=True
    ) as fetch_server
):
    agent = Agent(
        name="ResearchAssistant",
        instructions=research_assistant_instructions,
        model=MODEL,
        mcp_servers=[fs_server, fetch_server]
    )
```

- **`async with (...) as fs_server, ... as fetch_server`** -- Python 3.10+ syntax for opening multiple async context managers together. Both servers start up, and both shut down cleanly when the block exits.
- **`name="Filesystem"` / `name="WebFetch"`** -- human-readable labels for logging and debugging; they do not affect tool names.
- **`params={"command": ..., "args": [...]}`** -- the shell command the SDK runs to start each server process via stdio.
- **`mcp_servers=[fs_server, fetch_server]`** -- the agent sees all tools from both servers in one flat list and picks the right one per step.
- **`cache_tools_list=True`** -- avoids redundant `list_tools()` calls on every agent turn.

The secondary pattern is tool filtering for least-privilege:

```python
MCPServerStdio(
    ...
    tool_filter=create_static_tool_filter(
        allowed_tool_names=["read_file", "list_directory", "get_file_info"]
    )
)
```

This strips write and delete tools before they reach the agent.

## The demos and what each shows

**Part 1 -- Filesystem Agent Demo:** Creates a `mcp_workspace_b` directory with two seed files (`product_specs.txt` and `customer_feedback.txt`), connects the filesystem MCP server scoped to that directory, and asks a `ProductAnalyst` agent to read both files and write an `analysis_report.txt`. The demo proves the agent can discover filesystem tools automatically, read multiple files, reason across them, and write a new file -- all through MCP. Emphasize on camera that the server is scoped to `workspace.resolve()` (absolute path) and that write access is intentional here but dangerous if scoped too broadly.

**Part 2 -- Web Fetch Demo:** Connects the `mcp-server-fetch` server via `uvx`, prints the tool list from `list_tools()` (showing students exactly what tools the fetch server exposes), then asks a `WebResearcher` agent to fetch `https://openai.com` and produce a 3-sentence summary. The demo proves the fetch server converts HTML to clean markdown and that the agent can reason about web content. Emphasize that the agent gets readable text, not raw HTML, and that web content is untrusted input.

**Part 3 -- Combined Servers Demo:** Opens both filesystem and fetch servers in a single `async with` block, creates a `ResearchAssistant` agent with `mcp_servers=[fs_server, fetch_server]`, and asks it to read `product_specs.txt`, fetch `https://www.python.org`, and save a `combined_research.txt` file. The demo proves the agent can seamlessly interleave local file tools and web fetch tools in one task. Emphasize the security warning: combining untrusted web content with filesystem write access means a malicious page could drive unexpected file writes.

**Part 4 -- Tool Filtering Demo:** Connects the filesystem server with `create_static_tool_filter(allowed_tool_names=["read_file", "list_directory", "get_file_info"])`, prints the filtered tool list to show only three tools are visible, then runs a `ReadOnlyAgent` that lists files and reads `customer_feedback.txt`. The demo proves that filtered tools are invisible to the agent -- it cannot write or delete. Emphasize that this is a SDK-level constraint, not a prompt-level one.

**Part 5 -- Testing MCP Servers (Markdown only):** No runnable code; this is a checklist and failure-mode reference. Walk through each bullet on camera: start the server, call `list_tools()`, test a tool directly outside an agent, treat returned content as untrusted, restrict permissions, and check server logs. Highlight the four common failure modes (server won't start, tool not in list, timeout on first `npx -y` call, unexpected tool behavior).

## Gotchas worth knowing before recording

- The first `npx -y @modelcontextprotocol/server-filesystem` call downloads the package and can take 15-30 seconds. Subsequent runs are fast. Mention this on camera so students don't think the notebook is frozen.
- Similarly, the first `uvx mcp-server-fetch` call may download dependencies. If the cell seems slow, that is why.
- The `async with (server1, server2):` parenthesized syntax requires Python 3.10+. If a student is on 3.9, they need nested `async with` blocks. The troubleshooting section covers this.
- `workspace.resolve()` must produce an absolute path -- relative paths can cause silent failures or the server accessing the wrong directory.
- The `truncate_response` helper clips output at 1200 characters. If the live demo output looks cut off, that is the helper, not a bug.
- The web fetch demo targets `https://openai.com` and `https://www.python.org`. If either site is down or blocks the request, the demo will fail. Have a backup URL ready (the troubleshooting section suggests documentation sites work reliably).
- The combined demo has genuine security risk: the agent has both web fetch and file write access simultaneously. Acknowledge this explicitly on camera -- it is the setup for the security warning.
- `create_static_tool_filter` is imported from `agents.mcp` -- it is imported at the top of the notebook and again in Part 4. Students may miss the import if they jump to Part 4 directly.
- The workspace cleanup cell at the end (`shutil.rmtree`) deletes everything. If students want to inspect output files, they should do so before running cleanup.

## How it connects to adjacent notebooks

**Builds on M08A (MCP Fundamentals):** M08A introduced the MCP concept (the "USB-C analogy"), the `MCPServerStdio` class, and connecting to a single server. This notebook assumes students already understand what MCP is and how `MCPServerStdio` works. On camera, say something like "In the last notebook we connected to our first MCP server -- now we're going to use real servers that do useful things."

**Builds on M07B (Prompt Injection & Tool Safety):** The security notes throughout this notebook explicitly reference M07B. The idea that MCP tool output is untrusted input (web pages, file contents) is a direct application of prompt injection awareness from M07B. On camera, reference this: "Remember in Module 7 when we talked about treating tool output as untrusted? That applies here -- the web fetch server returns content from the open internet."

**Leads into M08C (Capstone #4 -- MCP-Powered Personal Assistant):** The capstone combines filesystem, web fetch, and a third MCP server into a full assistant with server-level approval before writes and `REASONING_MODEL` for decision-making. On camera, tease: "Next, we'll take everything from this notebook and build a real personal assistant that can research, read files, and save results -- with proper approval gates before it writes anything."
