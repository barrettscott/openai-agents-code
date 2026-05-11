# cc_nb28.md — 28_Real_World_MCP_Servers.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_5_MCP_And_Production/28_Real_World_MCP_Servers.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Add `uvx` one-line gloss in Prerequisites

2 reviewers (Anthropic, Gemini) flagged this. The cell tells students to install `uv` without saying what it is. The parallel explanation for `npx` exists in Notebook 27; this extends the same courtesy to `uvx`.

**Find:**
```
# Check uvx (for web fetch MCP server)
```

**Replace with:**
```
# Check uvx (for web fetch MCP server)
# uvx is a Python package runner from the 'uv' toolchain — the Python equivalent of npx.
# The web fetch MCP server ships as a Python package and uses it as its runtime.
```

---

## Change 2: Add `create_static_tool_filter` gloss before Part 4 demo

4 reviewers flagged the tool filtering section for various gaps. Grok's one-liner (what the filter actually does) is the cheapest way to anchor the section and covers Grok's own finding plus part of OpenAI's.

Add the following sentence to the Part 4 intro markdown cell, after the existing opening sentence about `create_static_tool_filter`:

```
`create_static_tool_filter(allowed_tool_names=[...])` tells the SDK to hide any tools not on your list — the agent literally cannot see or call them.
```

---

## Change 3: Add "two safety layers" sentence before the Part 4 demo

DeepSeek and Grok (2 reviewers) flagged that students don't understand why tool filtering adds value on top of directory scoping. This is the "why/when" gap for the whole section. Directly addresses the delivery note's "tool filtering as least privilege" priority.

Add the following sentence in the Part 4 intro markdown cell, after the `create_static_tool_filter` gloss from Change 2:

```
Scoping the directory limits *which files* the agent can reach; filtering tools limits *what the agent can do* with those files — two complementary safety layers. This is safer than instructions alone: even if a later prompt says "delete everything," the write tools are invisible to the agent.
```

---

## Change 4: Add `list_tools()`-first adaptation hint after the Part 4 demo

OpenAI flagged that students can copy the exact snippet but won't know what to put in `allowed_tool_names` for a different server. One sentence closes the adaptation gap.

Add the following sentence after `tools = await readonly_fs.list_tools()` in the Part 4 code cell, as a comment:

```
# In your own project, call list_tools() on the unfiltered server first to see
# the exact tool names, then put only the ones you want into allowed_tool_names.
```

---

## Change 5: Add MCP composition payoff sentence in Part 3 intro

1 reviewer (Anthropic), but this is the headline benefit of the standard and the outline explicitly lists "Combining multiple MCP servers" as a teaching beat. One sentence, zero risk. It also partially answers the "why MCP over `@function_tool`?" question for Part 2's web fetch demo.

Add the following sentence to the Part 3 intro markdown cell, after the existing "Pass multiple servers to `mcp_servers=[]`" sentence:

```
This is the payoff of the MCP standard — you can give an agent local file access and web access by composing two servers someone else wrote, with zero integration code on your side.
```

---

## Change 6: Replace "adversarial text" with named threat model in Part 5

1 reviewer (OpenAI), trivial wording change. Delivery notes also call for making "test before trust" a memorable production rule — naming the threat model sharpens the security section.

**Find** (apply to all occurrences — "adversarial text" appears in both the Part 5 checklist and Key Takeaways):
```
adversarial text
```

**Replace with** (replace_all):
```
text designed to manipulate the agent (prompt injection)
```

---

## Change 7: Make Part 5 "test a tool directly" advice actionable

5/5 reviewers flagged this as the top finding. The advice is currently abstract — students are told to test tools directly but never shown what that looks like in code. The inline-syntax fix (per Anthropic/Grok/OpenAI) is preferred over a full new demo cell. Verify the exact SDK method name before applying.

> ⚠️ VERIFY: Check the SDK to confirm the correct method name for directly invoking an MCP tool on a server object (`call_tool`, `run_tool`, or equivalent). The exact name must be verified before inserting the example.

**Find:**
```
- Test a tool directly (outside an agent) before putting it in `mcp_servers=[]`
```

**Replace with:**
```
- Test a tool directly (outside an agent) before putting it in `mcp_servers=[]`. In practice: connect to the server, call `list_tools()` to confirm what it exposes, optionally invoke one tool to see its raw output, then wire it into an `Agent`.
```
