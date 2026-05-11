# M08A: MCP Fundamentals — Recording Prep

## What this notebook teaches

This notebook introduces the Model Context Protocol (MCP) as the third way to give an agent capabilities, alongside `@function_tool` and built-in tools. Students learn how to connect an agent to an external MCP server using `MCPServerStdio`, discover tools automatically, and use those tools to read and write files -- all without writing any custom integration code. It answers the question: "How do I plug my agent into an ecosystem of pre-built tool servers instead of coding every integration by hand?"

## Concepts to explain on camera

### MCP (Model Context Protocol)
- **Term:** An open standard that lets AI agents connect to external tool servers using a shared protocol. The server exposes tools in a standard format; any MCP-compatible agent can connect and use them without knowing the server's internals.
- **Analogy:** USB-C for AI. Before USB-C every device had its own connector and you needed a drawer full of adapters. USB-C gave you one plug that works with everything. MCP does the same for agent-to-tool connections.
- **What students need to understand:**
  - MCP replaces per-tool integration code with a single protocol
  - Any MCP client (like the Agents SDK) works with any MCP server
  - There is a growing ecosystem of pre-built MCP servers for filesystems, web fetch, databases, etc.
  - You still need to think about security -- an MCP server can expose write and delete tools, not just read

### MCP Server / MCP Client / Agent (the three pieces)
- **Term:** The server exposes tools (e.g., filesystem read/write). The client connects to the server (the Agents SDK acts as the client). The agent reasons about which tools to call and when.
- **Analogy:** A restaurant: the kitchen (server) prepares dishes, the waiter (client/SDK) carries orders back and forth, and the customer (agent) decides what to order.
- **What students need to understand:**
  - You don't write tool implementation code -- the server handles that
  - You don't write connection code -- the SDK handles that
  - You configure which server to connect to and which directory to scope it to
  - The agent automatically discovers every tool the server exposes

### MCPServerStdio
- **Term:** A transport class that launches an MCP server as a local subprocess (via a command like `npx`) and communicates with it over standard input/output.
- **Analogy:** Like opening a terminal window, running a program, and talking to it by typing commands and reading its replies -- except the SDK does all of that for you behind the scenes.
- **What students need to understand:**
  - It requires `async with` to manage the server's lifecycle (start and stop)
  - The `params` dict takes `command` (e.g., `"npx"`) and `args` (the package name plus the directory path)
  - `cache_tools_list=True` avoids re-fetching the tool list on every call
  - A second transport type, `MCPServerSse`, handles remote HTTP servers and is covered in M08B

### Resources (MCP concept)
- **Term:** Static, readable data that an MCP server can expose alongside its tools -- things like files, database records, or API responses. Resources are data you read; tools are functions you call.
- **Analogy:** In a library, the books on the shelves are resources (you read them), while the librarian is a tool (you ask her to look something up).
- **What students need to understand:**
  - Resources and tools are both part of the MCP spec but serve different roles
  - This notebook mentions resources briefly; the focus here is on tools
  - Not every MCP server exposes resources -- it depends on the server

### npx
- **Term:** A Node.js command that downloads and runs an npm package without installing it globally. The filesystem MCP server is distributed as an npm package and launched with `npx -y @modelcontextprotocol/server-filesystem`.
- **Analogy:** Like `pipx run` in Python -- it grabs the package, runs it once, and you don't have to manage the install yourself.
- **What students need to understand:**
  - Node.js must be installed for this demo to work (the notebook has a verification cell)
  - The first run downloads the package and can take 15-30 seconds; subsequent runs use the cache
  - The `-y` flag auto-confirms the download prompt
  - This is specific to this particular MCP server -- other servers may use Python, Docker, or something else

## Key SDK pattern

The primary pattern is connecting an MCP server and passing it to an agent:

```python
async with MCPServerStdio(
    name="Filesystem",
    params={
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-filesystem", str(workspace.resolve())]
    },
    cache_tools_list=True
) as fs_server:

    agent = Agent(
        name="FileAssistant",
        instructions=file_assistant_instructions,
        model=MODEL,
        mcp_servers=[fs_server]
    )

    result = await Runner.run(agent, input="List all files in the workspace...")
```

Parameter-by-parameter:

- **`name="Filesystem"`** -- a human-readable label for the server; appears in traces and logs.
- **`params={"command": "npx", "args": [...]}`** -- tells the SDK what subprocess to launch. `"command"` is the executable; `"args"` is the argument list. The last arg is the absolute path to the directory the server can access.
- **`cache_tools_list=True`** -- fetches the tool list once at connection time and reuses it, instead of re-querying the server before every agent run. Use this when the server's tools don't change between calls.
- **`mcp_servers=[fs_server]`** -- the one-line connection point. The agent automatically discovers every tool the server exposes and can call them during a run. Multiple servers can be passed in the list (covered in M08B).
- **`async with ... as fs_server:`** -- manages the server lifecycle. The server starts when the block opens and shuts down when it exits. The agent must run inside this block.

## The demos and what each shows

**Part 2 -- Connect and Discover Tools (cell 17).** This demo connects to the `@modelcontextprotocol/server-filesystem` MCP server via `MCPServerStdio`, then calls `fs_server.list_tools()` to print every tool the server exposes -- names and descriptions. The point is to prove that tool discovery is automatic: you write zero schema definitions, zero tool functions. Emphasize on camera that the student never wrote a `@function_tool` decorator or a JSON schema -- the server told the agent what it can do. Note the security callout in cell 18: the server may expose write and delete tools, so always scope directory access tightly.

**Part 3 -- Agent with MCP Tools (cell 22).** This demo creates an `Agent` named `"FileAssistant"` with `mcp_servers=[fs_server]` and asks it to list and summarize every file in the workspace (`notes.txt`, `tasks.txt`, `config.json`). The agent autonomously decides which filesystem tools to call, reads each file, and returns a summary. Emphasize that the only thing connecting the agent to the filesystem is `mcp_servers=[fs_server]` -- one parameter, no integration code.

**Part 4 -- Reading and Writing (cell 25).** This demo asks the agent to read `tasks.txt` and then create a new file called `summary.txt` with a one-sentence summary. After the `async with` block closes, the notebook verifies `summary.txt` exists on disk by reading it with plain Python (`summary_path.read_text()`). This proves the MCP write operation had real side effects on the filesystem. Emphasize the security note: write tools are powerful, scope the directory carefully, and recall the prompt injection discussion from M07B.

## Gotchas worth knowing before recording

- The first `npx -y` run downloads the npm package and can take 15-30 seconds of dead air. Either pre-run the cell before recording or warn students that the first launch is slow and subsequent ones are fast.
- The `async with` block must stay open for the entire agent interaction. If you accidentally put `Runner.run()` outside the block, the server will already be shut down and you'll get a confusing connection error.
- `workspace.resolve()` produces an absolute path. If you pass a relative path, the server may scope to the wrong directory or fail silently. Always use `str(workspace.resolve())`.
- The filesystem server exposes write and delete tools by default -- there is no read-only mode in this demo. Be ready for students to ask "can I make it read-only?" (answer: scoping is directory-level; for read-only, you'd need a different server or a guardrail layer).
- The `tools` returned by `list_tools()` are MCP tool objects, not the same as `@function_tool` objects. Don't mix up the two systems on camera.
- `cache_tools_list=True` is a performance optimization, not a functional requirement. The demo works without it, but mention it as a best practice.
- The workspace cleanup cell (`shutil.rmtree`) runs at the end. If you re-run Part 3 or Part 4 after cleanup, the workspace won't exist and the agent will error. Re-run the workspace creation cell (cell 15) first.
- Node.js must be installed. If it isn't, the `subprocess.check_output` cell gives a clear error, but this could derail a live recording if forgotten.

## How it connects to adjacent notebooks

**Before (M07 -- Safety, Control & Observability):** M07B introduced prompt injection and tool safety, including the principle that tool output is untrusted input and that write actions need stronger controls. Call back to this on camera when discussing the security notes in cells 18 and 26 -- "Remember in M07B we talked about write tools needing extra care? Same principle applies here: the filesystem server can write and delete, so scope it tightly." M07D introduced tracing; mention that MCP tool calls show up in traces on the OpenAI dashboard, which helps debug whether the agent actually called the right tool.

**After (M08B -- Real-World MCP Servers):** M08B takes the single-server pattern from this notebook and extends it to multiple MCP servers in one agent (filesystem + web fetch). It also covers `MCPServerSse` for remote HTTP servers, common failure modes, and debugging flaky servers. Tee it up on camera: "Now that you know how to connect to one MCP server, next we'll connect to multiple servers at once and handle the real-world issues that come up."
