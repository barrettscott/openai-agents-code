# M08C: Capstone #4 --- MCP-Powered Personal Assistant --- Recording Prep

## What this notebook teaches

This notebook is the Module 8 capstone. It brings together three MCP servers -- filesystem, web fetch, and time -- under a single agent powered by `REASONING_MODEL` (`gpt-5`), showing students how multi-server coordination works in practice. The key production concern it drives home is that saving results to a file is a write side effect, so it must go through the same approval controls as any other destructive operation.

## Concepts to explain on camera

- **Multi-server MCP architecture (three servers, one agent):**
  - **Term:** Connecting multiple MCP servers to a single agent so it sees all their tools as one flat list. The agent decides which server's tool to call at each step.
  - **Analogy:** A personal assistant who has a filing cabinet, a web browser, and a clock on the desk -- they pick whichever tool fits the task without you telling them which drawer to open first.
  - **What students need to understand:**
    - You pass all servers in `mcp_servers=[fs_server, fetch_server, time_server]` and the agent sees every tool from every server
    - The agent plans tool order itself -- that is why `REASONING_MODEL` is used here instead of `MODEL`
    - Tools from different servers can be chained in a single task (e.g., get the time, read a file, then write a summary)
    - Adding or removing a server is a config change, not a code rewrite

- **Server-level approval (`require_approval`):**
  - **Term:** A per-tool policy on an `MCPServerStdio` that pauses the agent run before it can execute a destructive tool, giving you a chance to approve or reject.
  - **Analogy:** A hotel safe that opens freely for reading the contents but requires the front desk to authorize any deposits or removals.
  - **What students need to understand:**
    - Set with a dict: `"always"` lists tools that always pause, `"never"` lists tools that run without asking
    - In this notebook, `write_file`, `create_directory`, `move_file`, and `delete_file` require approval; `read_file`, `list_directory`, `get_file_info`, and `search_files` do not
    - When approval is needed, `result.interruptions` is truthy -- you loop, inspect each interruption, and call `state.approve()` or `state.reject()`
    - Tool names must exactly match what the server exposes or the approval policy silently does nothing

- **Saving results as a side effect:**
  - **Term:** When the agent writes research output or a summary to a file, that write operation carries the same risk as any other file mutation -- it is a side effect, not just "returning a result."
  - **Analogy:** Asking someone to look something up is safe; asking them to look something up and staple the answer into your filing cabinet is a write action.
  - **What students need to understand:**
    - Task 2 explicitly highlights this: fetching a URL is read-only, but saving the summary to `mcp_research.txt` triggers the write approval
    - In production, auto-approving writes (as the demo does for Task 2) bypasses the controls you just built in Task 1 -- call that out on camera
    - The same prompt injection concern from M07B applies: if the fetched web content contains malicious instructions, the agent might write something unexpected

- **`REASONING_MODEL` for coordination:**
  - **Term:** Using `gpt-5` (the reasoning model) instead of `gpt-5-mini` when the agent must plan across multiple servers and decide tool order.
  - **Analogy:** Using a senior employee for a task that requires judgment across departments, not just speed in one department.
  - **What students need to understand:**
    - `REASONING_MODEL = "gpt-5"` is assigned at the top and passed to the `Agent` via `model=REASONING_MODEL`
    - Worth the higher cost for multi-step tasks that span servers; for single-tool tasks, `MODEL` is cheaper and faster
    - The troubleshooting section suggests swapping to `MODEL` during testing, then switching back for the final demo

## Key SDK pattern

The primary pattern is creating multiple `MCPServerStdio` contexts with `require_approval` and wiring them into a single agent:

```python
async with (
    MCPServerStdio(
        name="Filesystem",
        params={
            "command": "npx",
            "args": ["-y", "@modelcontextprotocol/server-filesystem", str(workspace.resolve())]
        },
        cache_tools_list=True,
        require_approval={
            "always": {"tool_names": ["write_file", "create_directory", "move_file", "delete_file"]},
            "never": {"tool_names": ["read_file", "list_directory", "get_file_info", "search_files"]}
        }
    ) as fs_server,
    MCPServerStdio(
        name="WebFetch",
        params={"command": "uvx", "args": ["mcp-server-fetch"]},
        cache_tools_list=True
    ) as fetch_server,
    MCPServerStdio(
        name="Time",
        params={"command": "uvx", "args": ["mcp-server-time"]},
        cache_tools_list=True
    ) as time_server
):
    assistant = Agent(
        name="PersonalAssistant",
        instructions=assistant_instructions,
        model=REASONING_MODEL,
        mcp_servers=[fs_server, fetch_server, time_server]
    )
```

Parameter breakdown:

- **`name`** -- human-readable label for each server (appears in traces and logs)
- **`params.command` / `params.args`** -- the CLI command that starts the server process. `npx -y` for Node-based servers, `uvx` for Python-based servers
- **`cache_tools_list=True`** -- caches the tool list after first discovery so subsequent runs skip the discovery round-trip
- **`require_approval`** -- dict with `"always"` and `"never"` keys, each containing `{"tool_names": [...]}`. Tools not listed in either key follow the default policy
- **`mcp_servers=[...]`** -- the agent receives all three servers and sees all their tools as one merged list

The approval loop pattern:

```python
result = await Runner.run(assistant, input="...")
while result.interruptions:
    state = result.to_state()
    for interruption in state.get_interruptions():
        state.approve(interruption)  # or state.reject(interruption, message="...")
    result = await Runner.run(assistant, state)
```

## The demos and what each shows

**Phase 1 -- Prepare the Workspace:** Two seed files (`reading_list.txt` and `project_notes.txt`) are created in an `assistant_workspace` directory using plain `Path.write_text()`. This is not an MCP operation -- it is Python file I/O to stage realistic content the agent will read later. Emphasize on camera that the workspace directory path is passed to the filesystem server so the agent can only access files inside it (least-privilege scoping from M07B).

**Phase 2, Task 1 -- Time-stamped daily summary:** The agent is asked to get the current time, read both seed files, and create `daily_summary.txt` combining the date with a status overview. This task exercises all three servers: Time (get current time), Filesystem read (read the two files), and Filesystem write (create the summary). Because `write_file` is in the `"always"` approval list, the run pauses and the demo shows the interactive approval prompt (`input("Approve? (yes/no): ")`). On camera, stress that this is the same human-in-the-loop pattern from M07C, now enforced at the MCP server level rather than in custom tool code.

**Phase 2, Task 2 -- Research and save:** The agent fetches `https://modelcontextprotocol.io` via the WebFetch server and saves a 3-sentence summary to `mcp_research.txt`. The write triggers an approval interruption, but this time the code auto-approves with `state.approve(interruption)` instead of prompting the user. On camera, call out the inline comment explicitly: auto-approving in production bypasses the controls you just demonstrated in Task 1. Saving research output is a write side effect -- treat it with the same care as any destructive operation.

**Cleanup:** `shutil.rmtree(workspace)` removes the workspace. Quick and straightforward.

## Gotchas worth knowing before recording

- The three MCP servers start as child processes. First run downloads packages via `npx -y` and `uvx`, which can take 15-30 seconds and produce console noise -- run the notebook once before recording so packages are cached.
- All three servers start in parallel inside the single `async with` block, but if any server fails to start, the entire context manager fails. If one server hangs on camera, you will not get a partial startup -- you will get an error.
- `require_approval` silently does nothing if the tool name does not exactly match what the server exposes. If you typo `"write_files"` instead of `"write_file"`, writes will go through without approval and it will look like the feature is broken.
- The interactive `input()` prompt in Task 1 will block the notebook cell. If you accidentally hit Enter without typing "yes" or "no", the empty string will not match `["yes", "y"]` and the write will be rejected -- the agent will report the file was not created.
- Task 2 auto-approves writes. Students may wonder why Task 1 asks for approval but Task 2 does not -- explain that this is a deliberate design choice to show both patterns, and that in production you would choose one policy.
- The `truncate_response` helper is defined but never called in the demo cells -- it is there as a utility for students who get very long outputs. Do not be confused if you notice it unused.
- The security note after the demo cell references M07B prompt injection awareness. Briefly remind students that all MCP tool output (file contents, fetched web pages) is untrusted input -- a malicious web page could try to trick the agent into writing something unexpected.
- `workspace.resolve()` is passed to the filesystem server as an absolute path string. If you move or rename the notebook directory, this path changes and the server will scope to a different (possibly nonexistent) folder.

## How it connects to adjacent notebooks

**Builds on M08B (Real-World MCP Servers):** M08B introduced the filesystem and web fetch servers individually and covered combining two servers. This capstone adds a third server (Time), layers on `require_approval` for write protection, and uses `REASONING_MODEL` for multi-server coordination. On camera: "In M08B we connected servers one at a time -- now we are combining all three with production-grade approval controls."

**Builds on M07B (Prompt Injection & Tool Safety):** The `require_approval` pattern and the "saving results is a side effect" theme directly continue the read-vs-write tool safety discussion from M07B. On camera: "Remember in M07B when we talked about write tools needing stronger controls? This is that principle applied at the MCP server level."

**Builds on M07C (Human-in-the-Loop):** The approval loop (`result.interruptions` / `state.approve()` / `state.reject()`) is the MCP equivalent of the human-in-the-loop pattern from M07C. On camera: "Same idea as M07C -- pause before dangerous actions -- but now the pause is configured on the server, not coded into the tool."

**Leads into M09A (Project Structure & CLI):** The next step tells students to move from notebooks to a structured Python project. On camera: "You have built a working multi-server assistant in a notebook. Next module, we package this into a real project you can run from the command line."
