# cc_nb08.md — 08_Error_Handling_And_Recovery.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_2_Reliability_And_Built_In_Tools/08_Error_Handling_And_Recovery.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Explain the default SDK behavior before Part 1 demo

4 of 5 reviewers flagged this gap. The Part 1 intro introduces `failure_error_function=None` without explaining what the SDK's default behavior is, causing Part 2 to feel like a reversal. The student needs to know the default *before* seeing it disabled.

**Find** (in the Part 1 intro markdown cell, the sentence about disabling the error handler):
```
We disable the SDK's default error handler with `failure_error_function=None` so tool exceptions crash the run — this makes the failure mode visible.
```

**Replace with:**
```
By default, the SDK catches tool exceptions and returns a generic error message to the model. We disable that default here with `failure_error_function=None` so the raw exception crashes the run — making the failure mode visible. Part 2 will write proper handling that gives the agent something more useful than the generic default.
```

---

## Change 2: Add inline comment to `failure_error_function=None` decorator

Same 4-reviewer finding (Finding 1). Students copying Part 1 code will propagate the demo-only flag into their own tools without knowing it disables real error handling.

**Find** (the decorator line in the Part 1 code cell):
```
@function_tool(failure_error_function=None)
```

**Replace with:**
```
@function_tool(failure_error_function=None)  # Demo-only: surfaces the raw exception. Don't set this on real tools.
```

---

## Change 3: Define "exponential backoff with jitter" in Part 3

3 of 5 reviewers flagged this. The Why This Works cell tells students the demo's fixed-wait pattern isn't production-ready but doesn't explain the alternative. Students are left knowing what *not* to ship without knowing what to do instead.

**Find** (in the Part 3 "Why This Works" cell):
```
In production, use exponential backoff with jitter instead of fixed waits.
```

**Replace with:**
```
In production, use exponential backoff (longer wait after each failure) with jitter (a small random offset, so many clients don't all retry at once) instead of fixed waits. In a real application, also remove the print statements — the pattern inside the loop is what matters.
```

---

## Change 5: Define "idempotent" in the Part 3 security note

3 of 5 reviewers flagged this. The security note uses a technical term that gates the entire safety principle on first read. Students unfamiliar with it may skip or misread the warning.

**Find** (in the Part 3 security note cell):
```
⚠️ **Security note:** Only retry read-only or idempotent actions.
```

**Replace with:**
```
⚠️ **Security note:** Only retry read-only or idempotent actions (operations that produce the same result whether run once or multiple times).
```

---

## Change 6: Name the fallback mechanism explicitly in Part 4

Unanimous finding (5 of 5 reviewers). The fallback demo works but the mechanism — that the agent reads the tool's returned error string and acts on it per instructions — is never stated. Students may think fallback is automatic at the SDK level.

> ⚠️ **Note:** Part 4 currently has no existing "Why This Works" cell (cells 25–28 cover Part 4 with no Why-This-Works). Add a **new** markdown "Why This Works" cell after the last Part 4 code cell (cell 28) rather than appending to an existing cell.

Add a new `### 💡 Why This Works` markdown cell after the last Part 4 code cell (cell 28):

```markdown
### 💡 Why This Works

The convention from Part 2 holds here: a failing tool returns a clear error string instead of raising. The agent reads that string and follows the instructions to call the fallback — there is no automatic SDK mechanism. Putting recovery logic in the instructions (rather than a Python try/except around the tool calls) means you can change the strategy by editing the prompt as you add or swap tools.
```

---

## Change 7: Name what `run_safely()` actually catches in Part 5

3 of 5 reviewers flagged that the demo only exercises the happy path, leaving students without a mental model of when the wrapper triggers vs. tool-level try/except.

Add the following sentence after the printed response in the Part 5 demo cell or in the cell immediately below it:

```
This wrapper catches errors that happen outside any tool — an invalid API key, a rate-limit response, or a network timeout while the agent is calling the model. In a real app, it usually sits at your app boundary, where every user request passes through. If the API were unavailable, you'd see the friendly fallback message instead of the success above.
```
