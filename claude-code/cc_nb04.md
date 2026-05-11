# cc_nb04.md — 04_Building_Tools_For_Agents.ipynb

**Notebook:** `/Users/scott/Dropbox/Notebooks/openai-agents/Week_1_Foundations/04_Building_Tools_For_Agents.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Define "schema" in the @function_tool explanation cell

All five reviewers flagged this. "Schema" appears three times in the notebook (Topics, Part 1 explanation, Key Takeaways) without definition. A Python-comfortable student new to agent tooling has no reference for the term — they can still write working tools, but the core claim ("no manual JSON required") doesn't mean anything to them. Adding a parenthetical gloss to the existing bullet line is the smallest possible fix.

**Find:**
```
Type hints — define the expected inputs; the parameter types generate the schema automatically. No manual JSON required.
```

**Replace with:**
```
Type hints — define the expected inputs; the parameter types generate the schema (the structured description the model reads to know what arguments the tool expects) automatically. No manual JSON required.
```

---

## Change 2: Add a real-projects bridge after the employee lookup demo

Two reviewers flagged this. The toy lookup runs, the student sees it works — but there's no sentence connecting this to why tools matter outside a classroom. The pattern here is exactly how you'd connect an agent to a live database, internal API, or business system. One sentence makes that explicit.

> ⚠️ NOTE: The Find text below may have drifted from the current notebook. Apply semantically — locate the markdown or explanation cell in Part 1 that introduces or follows the `@function_tool` decorator and the employee lookup demo, then add the new sentence at the end of its bullet list.

**Find (in the markdown or explanation cell after the employee lookup "After" demo):**
```
The `@function_tool` decorator turns any Python function into an agent tool:
```

Add the following sentence at the **end** of the bullet list in that same cell (after "No manual JSON required."):

```
This is the core reason tools matter in real projects — the same pattern lets an agent answer questions from your live business data, internal APIs, or services instead of guessing from model knowledge.
```

---

## Change 3: Add docstring guidance at the end of Part 3

Three reviewers flagged this. The lesson correctly names the principle ("a clear docstring helps the agent pick the right tool") but never tells students *what* makes a docstring clear. They leave without a checklist for their own tools. The outline promises "Writing good tool descriptions — naming, inputs, outputs," and this delivers on it.

Add a new markdown cell at the end of Part 3 (after the three test runs, before the Key Takeaways):

```markdown
### 💡 Writing a Good Tool Docstring

A docstring the agent can act on states three things:
1. **What it returns** — the specific data the tool gives back
2. **When to use it** — the trigger condition ("use this when the user asks about…")
3. **What each input means** — if it's not obvious from the parameter name

Example: `"Get employee info"` is too vague. A docstring that names the returned fields and states the use case works much better.
```

---

## Change 4: Point to traces after the first tool run

One reviewer flagged this. After the first "After" demo the student sees correct output — but has no direct evidence the agent actually called the tool versus model knowledge. The traces dashboard confirms the mechanism and reinforces the debugging habit from Lesson 03.

Add one sentence after the first "After: Agent with an employee lookup tool" run cell:

```
Open the traces dashboard to confirm the agent selected `get_employee`, called it with the employee ID from your message, and used the result — this is what tool invocation looks like in practice.
```
