# cc_nb12.md — 12_Code_Interpreter.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_2_Reliability_And_Built_In_Tools/12_Code_Interpreter.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Fix Part 5 → Part 4 numbering

2 of 5 reviewers (Grok, DeepSeek); also called out in the delivery notes as the highest-priority edit. The notebook jumps from Part 3 to Part 5 with no Part 4. Students wonder if they missed a section.

**Find** (the section header):
```
## 📁 Part 5: File Input
```

**Replace with:**
```
## 📁 Part 4: File Input
```

Also find and update the matching divider cell if one exists above this section.

---

## Change 2: Explain the `CodeInterpreterTool` config dictionary in Part 2

3 of 5 reviewers (Gemini, Grok, DeepSeek). Students coming from the clean `FileSearchTool(vector_store_ids=[...])` API see a nested dict with magic strings and no explanation. They can copy but can't adapt.

Add the following sentence to the Part 2 "Why This Works" cell (or as a one-line markdown cell before the first `CodeInterpreterTool(...)` usage):

```
Unlike `FileSearchTool`, `CodeInterpreterTool` takes a `tool_config` dict — `"container": {"type": "auto"}` tells the SDK to spin up a temporary Python sandbox automatically.
```

---

## Change 3: Set expectations before the "Generate a Plot" demo in Part 3

3 of 5 reviewers (Anthropic, Gemini, Grok). Students run a cell titled "Generate a Plot" and see only text. The explanation that the chart renders inside the sandbox comes *after* the confusing experience.

Move the existing sentence about chart rendering from after the code cell to before it — do NOT duplicate it. Then add the following new sentence after it (still before the code cell):

```
The agent can also save the plot as an image file (e.g., `plot.png`) inside the sandbox — useful for automated reporting where charts get exported, not displayed inline.
```

---

## Change 4: Add inline/upload decision rule in Part 3

1 of 5 reviewers (OpenAI), but directly answers the student question "when do I use inline data vs. file upload?" — a practical choice they'll face on every real project.

Add the following sentence to the Part 3 intro cell, after "Code Interpreter can analyze inline data — no file upload needed":

```
Use inline data for small examples or generated data; switch to file upload when the dataset is too large or awkward to paste into the prompt.
```

---

## Change 5: Explain `purpose="assistants"` in Part 4 (formerly Part 5)

4 of 5 reviewers (Anthropic, Gemini, Grok, DeepSeek) — strongest consensus in the notebook. Students are confused by the term "assistants" here because it suggests the deprecated Assistants API, and they don't know how this upload differs from the vector store upload in Lesson 11.

Add the following as a markdown cell **before** the file upload code cell in Part 4:

```
`FileSearchTool` and `CodeInterpreterTool` use different file backends. In Lesson 11, we uploaded files to a vector store for semantic search. Here, we upload to OpenAI's general file store — `purpose="assistants"` is the upload tier required for Code Interpreter and is unrelated to the (separate) Assistants API.
```

Also add the following as an inline comment on the upload line itself:

```python
uploaded_file = openai_client.files.create(
    file=f,
    purpose="assistants"  # Required for Code Interpreter files — unrelated to the Assistants API
)
```

---

## Change 6: Resolve the "we just deleted it" contradiction in Part 4

2 of 5 reviewers (Anthropic, Grok). After the local CSV is deleted, the next prompt asks the agent to read `sales_report.csv` — students think "but we just deleted it." A single print-line change resolves the contradiction.

**Find** (the print statement after `csv_path.unlink()`):
```
print("✅ Local file cleaned up")
```

**Replace with:**
```
print("✅ Local copy removed — the file now lives in the container via uploaded_file.id")
```

---

## Change 7: Clarify "container config" jargon in Part 4

1 of 5 reviewers (OpenAI), but the term blocks the student from knowing what to edit in their own project. One-sentence fix.

> ⚠️ NOTE: Apply after Change 1 (renumbering) — target the cell that will be Part 4 after renumbering.

**Find** (in the Part 4 intro or Why This Works prose):
```
then pass that ID into the container config.
```

**Replace with:**
```
then pass that ID into the container config (the dictionary you pass into `CodeInterpreterTool(...)` to tell the sandbox which files to mount before execution).
```
