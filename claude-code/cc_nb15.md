# cc_nb15.md — 15_Agents_As_Tools.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_3_Multi_Agent_Systems/15_Agents_As_Tools.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Add "Why This Works" cell after the Part 3 demo

4 reviewers (Anthropic, OpenAI, Gemini, DeepSeek) flag the absence of a closing explanation after the multi-specialist review pipeline demo. Part 2 has a Why This Works cell; Part 3 has none. Students miss what's actually new: the orchestrator can hold intermediate outputs in context and pass them forward, and the `verdict_writer` is one option, not a requirement.

Add the following new markdown cell immediately after the "Run the Review Pipeline" demo cell in Part 3 (before Practice Exercises):

```markdown
### 💡 Why This Works

The orchestrator doesn't just call tools in parallel — within a single turn it holds the results of earlier tool calls in context, so it can summarize features and pricing before calling `verdict_writer`. The `verdict_writer` step is one option, not a requirement: the general pattern is gather specialist outputs first, then either synthesize them in the orchestrator or pass them to one final specialist.
```

---

## Change 2: Explain how input reaches the specialist and give the application rule

3 reviewers (Anthropic, OpenAI, DeepSeek) flag this. Students see `.as_tool()` with no typed input parameter (unlike `@function_tool` from Lesson 04) and don't know how the specialist receives input — or how to apply the pattern in their own code.

**Find** the `.as_tool()` introduction paragraph in Part 2 (the one beginning "Any agent can be exposed as a tool using `.as_tool()`..."). Append the following two sentences:

```
The model phrases the input as a string, the specialist runs on that string as its message, then returns its output. In your own projects, the pattern is: build a specialist agent first, then expose it inside another agent's `tools=[...]` list with `specialist_agent.as_tool(...)` — and keep the specialist's instructions general enough to handle any input the orchestrator phrases.
```

---

## Change 3: Reword the Step 3 orchestrator instruction to remove implied manual passing

1 reviewer (DeepSeek), partially blocks transfer. The phrase "with a summary of both" implies the student must construct and inject a string, when in fact the orchestrator relies on conversation context already holding prior tool outputs. Students may over-engineer Exercise 2 as a result.

**Find:**
```
3. Call verdict_writer with a summary of both to get a final recommendation
```

**Replace with:**
```
3. Call verdict_writer to produce a final recommendation — it has the feature and pricing analyses available from the prior steps
```

---

## Change 4: Explain `tool_name=` argument purpose

1 reviewer (Gemini). Students coming from Lesson 14 know the SDK can derive a tool name from the agent's `name` — they'll wonder why an explicit `tool_name` is provided here.

Add the following sentence to the Part 2 prose near where `pros_agent.as_tool(tool_name="pros_analyst", ...)` is introduced:

```
The `tool_name` gives the orchestrator a stable, function-like name to call — independent of the specialist agent's display name, which keeps your orchestrator prompts clean.
```

---

## Change 5: Add use-case sentence to Part 2 "Why This Works"

1 reviewer (OpenAI). Students understand who stayed in control but not what real problem that solves — why pick this pattern over one bigger agent.

**Find** the Part 2 Why This Works cell (the one containing "`result.last_agent.name` is the orchestrator in this pattern — not a specialist — because control never transferred away."). Append:

```
This matters when you need a single final response that combines multiple specialist views — comparing pros and cons, gathering checks from different reviewers, or enforcing one consistent output format.
```

---

## Change 6: Replace "Coordinating" in the comparison table

1 reviewer (Anthropic). "Routing" is concrete from Lesson 14; "Coordinating" is abstract until the demo runs. Minor but easy fix.

**Find:**
```
| Best for | Routing | Coordinating |
```

**Replace with:**
```
| Best for | Routing | Combining specialist outputs |
```
