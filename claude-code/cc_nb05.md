# cc_nb05.md — 05_Writing_Agent_Instructions.ipynb

**Notebook:** `/Users/scott/Dropbox/Notebooks/openai-agents/Week_1_Foundations/05_Writing_Agent_Instructions.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Connect back to the Lesson 04 assistant

Three reviewers flagged this. The outline explicitly promises "take the multi-tool assistant from Lesson 04 and make it behave better." Instead, every demo starts from scratch with a different scenario. The minimal fix is one sentence after the durable summarization demo that bridges back — students don't need a new demo, they need confirmation that the principle applies to what they just built.

Add one sentence after the durable agent run cell in Part 1:

```
The same before/after improvement applies to the multi-tool assistant you built in Lesson 04 — replacing its vague instructions with these durable rules makes its tool choices far more predictable.
```

---

## Change 2: Add a "Why This Works" cell to Part 1

Three reviewers flagged this. Parts 2, 3, and 4 each end with a 💡 Why This Works cell. Part 1 stops after the durable output with no explanation of *which lines* caused the change or what "completion criteria" (promised in the Part header) actually means. A student finishing Part 1 has the term but no working definition.

Add a new `### 💡 Why This Works` markdown cell after the durable agent run, before the Part 2 divider:

```markdown
### 💡 Why This Works

The durable instructions specify three things that the vague version left open:

- **Format** — `produce exactly one sentence`
- **Length** — `use plain language`
- **Stop condition** — `Do not add commentary or follow-up questions`

These are *completion criteria* — they tell the agent when it's done. A durable instruction usually answers four things explicitly: role, task, output format, and what not to do.
```

---

## Change 3: Name the mechanism in Part 2's "Why This Works" cell

Two reviewers flagged this. The existing Why This Works cell describes *what happened* (missing → ask, complete → proceed) but not the transferable design principle. A student writing their own clarifying instructions might write "ask the user if anything is unclear" and get inconsistent results — because the trick is enumerating specific required fields first.

**Find (in the Part 2 "### 💡 Why This Works" cell):**
```
Same instructions, two behaviors — depending on what the user provides:
```

Add the following lines at the end of that same cell (after the existing bullets):

```
The mechanism: list the specific required fields explicitly. Without that list, "ask if unclear" produces inconsistent behavior. Use this any time acting on missing details would create the wrong calendar event, send the wrong email, or trigger any other wasted action.
```

---

## Change 4: Add a note before the Part 2 agent run — no tools, text-only

One reviewer (Gemini) flagged this, and it's correct. The `agent_clarify` is defined without tools but its instructions imply it can book meetings. This directly contradicts the tool-vs-instruction mental model from Lessons 03–04. A one-sentence note prevents the confusion.

Add a markdown note before the "Run Agent with Incomplete Request" cell:

```
Note: this agent has no real scheduling tool — it responds *as if* it can schedule, but it's only generating text. We're focused on the conversational logic here.
```

---

## Change 5: Add real-world payoff and instructions-vs-docstrings hierarchy to Part 3

Three reviewers flagged this from different angles (tokens/latency payoff, and which control layer wins). Both belong in the Why This Works cell since they answer the same student question: "when would I use this, and how does it interact with what Lesson 04 taught me?"

**Find (in the Part 3 "### 💡 Why This Works" cell):**
```
The instruction names the condition for tool use explicitly. Without it, the agent decides on its own — and may call the tool on questions it could answer directly.
```

**Replace with:**
```
The instruction names the condition for tool use explicitly. Without it, the agent decides on its own — and may call the tool on questions it could answer directly. On a real project this saves tokens, reduces latency, and stops the agent from calling paid or slow tools for questions it already knows how to answer. Instructions sit above tool docstrings in the decision chain — the agent first checks instructions for whether a tool is allowed, then looks at docstrings to pick which one.
```

---

## Change 6: Define refusal and escalation before the Part 4 code cell

Three reviewers flagged this. "Refusal" and "escalation" are introduced as technical terms in the Part 4 header, and the very next cell jumps into code with three tests. Students are juggling new vocabulary at the same time as new code. Two one-line definitions before the code cell resolve it.

**Find (in the Part 4 opening markdown cell):**
```
Agents need to know what they should not do, and what to do instead:
```

Add two lines immediately after the existing bullet list in that cell:

```
**Refusal** = the agent says "I can't do that."
**Escalation** = the agent says "I can't do that, but here's who can."
```

---

## Change 7: Replace "load-bearing" with "critical" in Key Takeaways

One reviewer flagged this. "Load-bearing" hasn't appeared anywhere in the course and reads as insider jargon at the moment of summary. "Critical" is already used in the outline and earlier in this notebook.

**Find:**
```
Instructions become load-bearing in Lessons 14–18
```

**Replace with:**
```
Instructions become critical in Lessons 14–18
```
