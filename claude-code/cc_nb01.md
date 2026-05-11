# cc_nb01.md — 01_Environment_Check.ipynb

**Notebook:** `/Users/scott/Dropbox/Notebooks/openai-agents/Week_1_Foundations/01_Environment_Check.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Acknowledge unfamiliar packages in Check 4

Three of five reviewers flagged that `fastapi`, `uvicorn`, and `pydantic` appear in the package check without ever being mentioned in the README. A student sees these alongside `openai-agents` and `chromadb` and wonders "did I miss a setup step?" One sentence under the Check 4 header dissolves the doubt for all three at once.

**Find (in the markdown cell for Check 4):**
```
Let's verify all course packages are installed.
```

**Replace with:**
```
Let's verify all course packages are installed.

These packages are used in later notebooks — you don't need to know them yet.
```

---

## Change 2: Clarify the relationship between `agents` and `openai` packages

A student knows the course is about the "OpenAI Agents SDK," which is the `agents` package. Seeing a separate `openai` import with no comment raises "are these different things?" — an easy inline comment resolves it.

**Find:**
```
import openai
```
*(in the Check 4 package verification code cell)*

**Replace with:**
```
import openai  # base library, also used by the Agents SDK
```

---

## Change 3: Explain the Gradio version pin

Every other package check is a bare import. The Gradio check has an extra version comparison that isn't explained. One inline comment answers the silent "why?"

**Find:**
```
if major >= 6:
```

**Replace with:**
```
if major >= 6:  # Week 5 features require Gradio 6+
```

---

## Change 4: Name the failure mode for not pinning the kernel

The current text tells students *what* happens if they skip pinning but not *why it's confusing* — the consequence sounds abstract. Naming the actual failure mode ("module not found" even though the package is installed) gives them a reason to do it now rather than later.

**Find:**
```
Without pinning, VS Code can silently fall back to a different Python between sessions.
```

**Replace with:**
```
Without pinning, VS Code can silently fall back to a different Python between sessions, which causes confusing 'module not found' errors even though the package is installed.
```
