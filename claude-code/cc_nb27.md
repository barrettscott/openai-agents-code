# cc_nb27.md — 27_MCP_Fundamentals.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_5_MCP_And_Production/27_MCP_Fundamentals.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Fix "external server" wording to avoid remote-vs-local confusion

One reviewer (Anthropic) flagged this, but it directly contradicts content two cells later where the server is a local subprocess. The fix is a single word swap and costs nothing.

> ⚠️ NOTE: The Find text below may have drifted. Apply semantically — locate the "Three ways" comparison cell or the MCP bullet in the introductory section, then apply the replacement.

**Find:**
```
MCP — an external server hosts the tool; your agent connects and uses it automatically
```

**Replace with:**
```
MCP — a separate MCP server hosts the tool (running locally or remotely); your agent connects and uses it automatically
```

---

## Change 2: Append ecosystem payoff to the MCP bullet in "Three ways"

2 reviewers (Grok, DeepSeek) flagged that the MCP bullet doesn't answer "when would I reach for this over a simple `@function_tool`?" This is the "why MCP at all" beat the outline requires.

**Find:**
```
MCP — a separate MCP server hosts the tool (running locally or remotely); your agent connects and uses it automatically
```

**Replace with:**
```
MCP — a separate MCP server hosts the tool (running locally or remotely); your agent connects and uses it automatically. This is the third path: instant access to a growing ecosystem of community-maintained servers (GitHub, databases, Slack, calendars) with zero integration code.
```

---

## Change 3: Add one sentence explaining `MCPServerStdio` and `list_tools()` purpose

2 reviewers (Gemini on the class name, OpenAI on `list_tools()` purpose) flagged this. One sentence in the Part 2 intro covers both.

Add the following sentence to the Part 2 intro markdown cell, after the existing opening sentence:

```
We'll use `MCPServerStdio` — the SDK's client for local servers that communicate over standard input/output (stdio). Calling `list_tools()` yourself is how you inspect a new MCP server's surface before trusting an agent with it.
```

---

## Change 4: Add "resources" clarifying clause

3 reviewers (OpenAI, Gemini, Grok) flagged the "resources" sentence as landing without enough definition. OpenAI and Gemini prefer a clause; Grok prefers deletion. The clause is cheaper and preserves the foreshadowing. If resources don't appear in any later notebook, revisit deletion.

**Find:**
```
MCP servers can also expose **resources** — readable data like files or database records — though this notebook focuses on tools.
```

**Replace with:**
```
MCP servers can also expose **resources** — read-only data objects the agent can access directly, unlike tools which perform actions — though this notebook focuses on tools.
```

---

## Change 5: Add "transfer to other servers" sentence after Part 3 demo

3 reviewers (OpenAI, Grok, Gemini) flagged that after the demo, students can't picture what they'd change to use a different MCP server. This is the single most consensus-backed comprehension gap.

Add the following sentence after the demo output cell (after the final `print("=" * 60)` line and before the Cleanup header):

```
To use any other MCP server, you only change the `command` and `args` passed to `MCPServerStdio` — the `mcp_servers=[server]` line and the rest of the agent code stay exactly the same. The command and args for any server come from its documentation.
```

---

## Change 6: Add approval-gate caveat to the security note

1 reviewer (Anthropic), but high-impact: it directly contradicts the safety lesson from Notebook 26 and the outline explicitly calls for Notebook 29 to cover this. Without a pointer, students will build MCP write agents with no approval layer and not realize it's missing.

Add the following sentence to the existing security note cell (append to the end of that cell):

```
Note: MCP tools don't carry a `needs_approval` flag the way `@function_tool` does — Notebook 29 adds server-level approval for write and delete actions.
```

---

## Change 7: Remove `MCPServerSse` bullet from Key Takeaways

4 reviewers (Anthropic, OpenAI, Gemini, DeepSeek) flagged this. SSE appears nowhere else in the notebook. Gemini and DeepSeek recommend deletion; OpenAI recommends anchoring with a clause. Deletion is cleaner — SSE hasn't been taught yet.

**Find:**
```
- `MCPServerSse` — remote server over HTTP/SSE
```

**Replace with:**
*(delete the line entirely)*

> ⚠️ VERIFY: Confirm the Key Takeaways cell lists `MCPServerSse` as a standalone bullet before deleting. If it's been changed in a prior edit, skip this change.
