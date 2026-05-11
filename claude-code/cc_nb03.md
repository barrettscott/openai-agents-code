# cc_nb03.md — 03_How_Agents_Work.ipynb

**Notebook:** `/Users/scott/Dropbox/Notebooks/openai-agents/Week_1_Foundations/03_How_Agents_Work.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Fix the REASONING_MODEL comment — remove "Orchestrators"

All five reviewers flagged this. A student in Lesson 03 has never encountered multi-agent systems (Week 3). "Orchestrators" reads as dropped-in jargon and makes them feel behind. The fix is a one-word change that keeps the useful forward reference while replacing the jargon with beginner-legible language.

**Find:**
```
REASONING_MODEL = "gpt-5"  # Orchestrators and complex reasoning tasks (full framework: Lesson 31)
```

**Replace with:**
```
REASONING_MODEL = "gpt-5"  # Complex planning and reasoning tasks (full framework: Lesson 31)
```

---

## Change 2: Give tracing a "when to use" reason

All five reviewers flagged this. The notebook tells students *what* tracing is and *where* to find it — but not *why* they should open it. Without a concrete payoff, the feature reads as a dashboard curiosity. When a real agent misbehaves in a later lesson, they won't think to check. One sentence anchors it as a debugging tool.

**Find:**
```
Every agent run is automatically traced — no configuration needed. You can inspect a trace in the OpenAI platform to see the input, the agent instructions, and the final output together.
```

**Replace with:**
```
Every agent run is automatically traced — no configuration needed. You can inspect a trace in the OpenAI platform to see the input, the agent instructions, and the final output together. When an agent does something unexpected, this is the first place to look — traces show you exactly what input and instructions produced that output, so you don't have to guess.
```

---

## Change 3: Name the three core pieces explicitly in the "Why This Works" cell

Three reviewers flagged this. The lesson promises students will "see exactly how `Agent`, `Runner`, and `result.final_output` fit together" — but the Why This Works cell only addresses `RunResult` and `final_output`. The three-piece pattern is never named in order. Replace the existing cell with one sentence that names all three roles.

**Find (in the first "### 💡 Why This Works" cell after the opening demo):**
```
`Runner.run()` returns a `RunResult` object. `final_output` is one of its fields — the agent's final text response. Later notebooks will use other fields too.
```

**Replace with:**
```
You just used the three core pieces: `Agent` defines who and how, `Runner.run()` executes it, and `result.final_output` is the response. Every notebook in this course follows this pattern.
```

---

## Change 4: Remove duplicate cells at the end of the notebook

Both the NBR review and the delivery notes flag duplicate "Key Takeaways" and duplicate Troubleshooting link cells at the end of the notebook. The duplication makes the ending feel unpolished.

Find and delete the **second** occurrence of the "Key Takeaways" markdown cell (the duplicate).

Find and delete the **second** occurrence of the Troubleshooting link cell (the duplicate).

> ⚠️ Keep the first occurrence of each — only remove the duplicates.

---

## Change 5: Extend the instructions takeaway — they're more than a tone lever

Two reviewers flagged this. Seeing two outputs differ in tone leaves students thinking instructions are mainly for persona. They don't connect this to controlling when an agent asks, refuses, escalates, or uses a tool. One sentence sets up everything from Lessons 05 onward.

> ⚠️ IMPORTANT: Apply this change AFTER Change 4 above. The target cell is in the **kept** (first) "Why This Works" cell for the instructions section — cell 15, not cell 16 which will have been deleted by Change 4. Locate this cell by finding the first occurrence of the following text in the notebook.

**Find (in the kept "Why This Works" cell for the instructions demo — cell 15):**
```
The only change was `instructions`, which defines the agent's persona and behavior for every run.
```

**Replace with:**
```
The only change was `instructions`, which defines the agent's persona and behavior for every run. This is the most powerful lever you have — good instructions decide how your agent behaves when a real task is ambiguous, risky, or needs a tool.
```
