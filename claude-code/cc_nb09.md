# cc_nb09.md — 09_Testing_And_Evaluating_Agents.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_2_Reliability_And_Built_In_Tools/09_Testing_And_Evaluating_Agents.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Add missing `check_output()` helper function

4 of 5 reviewers flagged this as a hard blocker. The function is called repeatedly in Parts 2, 3, and 4 but is never defined anywhere in the notebook. Every pass/fail demo crashes with `NameError`. This is the highest-priority fix in the notebook.

> ⚠️ **Before adding:** First search the entire notebook for any existing definition of `check_output` (e.g., `def check_output`). If it is already defined anywhere in the notebook, skip this change. Only add the helper if it is truly missing.

If `check_output` is not defined anywhere in the notebook, insert a new **code cell** before the "Run the Full Test Suite" cell in Part 2 containing:

```python
def check_output(output: str, must_contain: list[str], must_not_contain: list[str]) -> tuple[bool, list[str]]:
    """Check if output contains all required and none of the forbidden strings."""
    output_lower = output.lower()
    failures = []

    for s in must_contain:
        if s.lower() not in output_lower:
            failures.append(f"Missing required string: {s}")

    for s in must_not_contain:
        if s.lower() in output_lower:
            failures.append(f"Contains forbidden string: {s}")

    return not failures, failures

# --------------------------------------------------------------
print("✅ check_output() helper ready")
```

---

## Change 2: Name the test-set coverage strategy in Part 1

1 of 5 reviewers (Anthropic), but the underlying issue is real and the fix is a single sentence. Students see five example test cases but no named principle for what they cover. They'll default to writing only happy-path tests.

> ⚠️ **Before adding:** Check whether the Part 1 intro cell already contains the three-category framing (happy paths, edge cases, known failure modes). If that framing is already present in the Part 1 intro, skip this change — it already exists in Key Takeaways and adding it again would be redundant.

If the three-category framing is NOT already present in the Part 1 intro, add the following sentence to the Part 1 "Building a Golden Test Set" intro cell, after the definition of what a test case is:

```
A useful test set covers three things: happy paths, edge cases (missing items, ambiguous inputs), and known failure modes you've already seen the agent get wrong.
```

---

## Change 3: Explain why the judge needs ground truth in Part 3

4 of 5 reviewers flagged this (from different angles). The judge_input construction passes the full catalog to the judge with no stated principle. The transferable insight — that the judge has no built-in knowledge of your data — is only visible in code.

Add the following sentence to the Part 3 intro cell, before the evaluation loop:

```
The judge agent has no built-in knowledge of your data — pass the ground truth into the prompt so it can verify factual accuracy instead of guessing. For larger datasets, pass only the relevant slice (e.g., the entry for the product being asked about), not the entire catalog.
```

---

## Change 4: State the hypothesis before the Part 4 comparison

1 of 5 reviewers (Anthropic), but genuinely blocks the lesson's transferable point — that you state a hypothesis before measuring. Without it, the result reads as "v2 won" with no insight.

Add the following line above the `v2_instructions` definition in Part 4:

```
# v2 trades verbosity for structure: lead with the answer, suggest alternatives proactively, cap at 3 sentences.
# The test set will tell us whether the tighter format costs us factual accuracy.
```

---

## Change 5: Clarify the pass-rate vs. avg-score decision rule in Part 4

3 of 5 reviewers flagged this. Two metrics are computed and printed but only one determines the winner — leaving students without guidance on how to use the other.

Add the following line after the winner declaration print in the Part 4 summary:

```
print("💡 Pass rate catches factual breakage; avg score catches quality drift. Don't ship a version that loses on one metric to win on the other.")
```

---

## Change 6: Fix three cell-type errors (execution blockers)

Three cells have the wrong `cell_type`, causing Parts 2–4 to fail even after Change 1 adds `check_output()`.

Apply the following cell-type corrections:

1. The cell containing `## ✅ Part 2: Pass/Fail Checks` as its content is currently a **code** cell — convert it to **markdown**.
2. The cell beginning `# Select a few test cases for rubric evaluation` is currently a **markdown** cell — convert it to **code**.
3. The cell beginning `# Reuse v1 baseline from Part 3` is currently a **markdown** cell — convert it to **code**.

> ⚠️ VERIFY: Confirm each cell's current `cell_type` before converting. The markdown→code conversions (items 2 and 3) are needed because those cells contain runnable Python that will silently not execute if left as markdown.

---

## Change 7: Add json save/load snippet after the test-set persistence note in Part 1

1 of 5 reviewers (Grok), but the fix is a code snippet that directly demonstrates a stated principle ("save and version-control your test set") that currently has no code to back it up.

Add the following **code cell** immediately after the paragraph that says test sets should be saved to a file:

```python
# Save the test set to a file for version control
import json

with open("test_cases.json", "w") as f:
    json.dump(TEST_CASES, f, indent=2)

# Load it back in a future session
# TEST_CASES = json.load(open("test_cases.json"))

print("✅ Test cases saved to test_cases.json")
```
