# cc_example.md — Example Notebook (example_nb.ipynb)

**Notebook:** `example/example_nb.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## How to apply these changes

Each change specifies an **Action** — use the matching NotebookEdit operation:

| Action | NotebookEdit operation | When to use |
|--------|----------------------|-------------|
| `EDIT_CELL` | `edit` — modify source of existing cell | Changing or extending cell content |
| `INSERT_CELL` | `insert` — add a new cell at a position | Adding new cells that don't exist yet |
| `FIX_TYPE` | `change_cell_type` — change `cell_type` only | Fixing code↔markdown mismatches |
| `DELETE_TEXT` | `edit` — remove specific text from cell source | Deleting sentences within an existing cell |
| `DELETE_CELL` | `delete` — remove the cell entirely | Removing a cell with no replacement |

**Critical rules:**
- `EDIT_CELL` and `DELETE_TEXT` always target an **existing** cell — use `edit`, never `insert`
- `INSERT_CELL` always creates a **new** cell — use `insert` with the specified `cell_type`
- `FIX_TYPE` changes only the `cell_type` — do **not** modify the source
- For `⚠️ VERIFY` steps: **read the cell content first**, then decide whether to apply
- After applying all changes to a notebook, confirm cell count matches expectation: **Before: 23 cells → After: 24 cells**

---

## Change 1: Fix Part 1 header cell type

The cell containing the Part 1 section header is a `code` cell — it will not render as markdown and cannot be executed. Convert it to `markdown`.

**Action: FIX_TYPE**
**Locate cell by:** first line is `## 📋 Part 1: Direct Answer vs Tool Call`
**Change `cell_type`:** `code` → `markdown`
**Do not modify the source.**

---

## Change 2: Extend the Part 1 "Why This Works" cell

Add the instructions-vs-docstrings hierarchy sentence. This is the transferable design principle behind the tool-constraint pattern.

**Action: EDIT_CELL**
**Cell type: markdown**
**Locate cell by:** `### 💡 Why This Works` (in the Part 1 section, after the weather agent runner)

**Find within that cell:**
```
The agent calls the tool because the question requires live data the model cannot answer from training.
```

**Replace with:**
```
The agent calls the tool because the question requires live data the model cannot answer from training. Instructions sit above tool docstrings in the decision hierarchy — the agent checks instructions first to decide whether a tool is allowed, then reads the docstring to decide which tool to call.
```

---

## Change 3: Remove redundant sentence from Part 2 intro

The second sentence says the same thing as the parameter-naming rule already stated in Key Takeaways. Delete it.

**Action: DELETE_TEXT**
**Cell type: markdown**
**Locate cell by:** `## 🔢 Part 2: Tools with Multiple Parameters`

**Find within that cell:**
```
 Parameter names and type hints are the only thing guiding that extraction, so name them clearly.
```

**Delete that sentence** (remove it entirely — do not replace with anything).

**Expected cell source after deletion:**
```
## 🔢 Part 2: Tools with Multiple Parameters

Tools can accept more than one argument — the agent extracts all of them from natural language.
```

---

## Change 4: Add a "Note:" cell before the Part 2 runner

Students may assume the currency agent calls a live API. One sentence prevents the confusion.

**Action: INSERT_CELL**
**Cell type: markdown**
**Position: BEFORE the cell whose first line is `#### Run the Agent`** (the one in Part 2, after the `convert_currency` tool definition — not the Part 1 runner)

**Content:**
```markdown
**Note:** This agent uses fixed demo exchange rates — it does not call a live API. We're focused on parameter extraction here, not data accuracy.
```

> ⚠️ VERIFY: There are two `#### Run the Agent` cells in this notebook (one in Part 1, one in Part 2). Insert before the **second** one (after the `currency_agent` definition). Confirm by checking that the cell immediately before the insert point is the `currency_agent = Agent(...)` code cell.

---

## Change 5: Replace "load-bearing" with clearer language in Key Takeaways

"Load-bearing" is jargon. Replace with plain English.

**Action: EDIT_CELL**
**Cell type: markdown**
**Locate cell by:** `## 🎯 Key Takeaways`

**Find within that cell:**
```
Parameter names and type hints are load-bearing — they are the agent's only guide to what values to pass
```

**Replace with:**
```
Parameter names and type hints are the primary signal to the agent — they are the agent's only guide to what values to pass
```

---

## Expected result

After applying all 5 changes:
- **Cell count:** 24 (was 23 — Change 4 adds 1)
- **Cell[5]:** `cell_type` is now `markdown` (was `code`)
- **Cell[9]:** Why This Works contains the instructions-vs-docstrings sentence
- **Cell[11]:** Part 2 intro ends after the first sentence (second sentence removed)
- **Cell[13]:** New `markdown` Note cell inserted before the Part 2 runner
- **Cell[21]:** Key Takeaways contains "primary signal to the agent" (not "load-bearing")

Compare against `example_nb_expected.ipynb` to verify.
