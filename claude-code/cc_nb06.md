# cc_nb06.md — 06_Pydantic_Basics.ipynb

**Notebook:** `/Users/scott/Dropbox/Notebooks/openai-agents/Week_1_Foundations/06_Pydantic_Basics.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Explain what `Annotated` is doing

All five reviewers flagged this — the only unanimous finding for this notebook. A student comfortable with `name: str` type hints sees `Annotated[str, Field(max_length=50)]` with no introduction. They can copy the pattern but can't adapt it (e.g., add a second constraint). One sentence before the code cell closes the gap.

**Find (in the Part 3 intro markdown cell):**
```
For tighter rules (a rating between 1 and 5, a name no longer than 50 characters), use `Field` with `Annotated`.
```

**Replace with:**
```
For tighter rules (a rating between 1 and 5, a name no longer than 50 characters), use `Field` with `Annotated`. `Annotated` lets you attach extra metadata — here, the `Field` constraints — to a type hint without changing the type itself. `ge` and `le` mean *greater-or-equal* and *less-or-equal*.
```

---

## Change 2: Add "why" and decision rule to the Part 4 serialization section

Two reviewers flagged these together. The intro says "Pydantic models also serialize cleanly" but never says *when you'd need this*. The demo shows two methods but never gives a rule for which to choose. Both gaps are fixed in a 2-sentence addition — combine them so the student gets the "why" and the "which" in one place.

**Find (in the Part 4 intro markdown cell):**
```
Pydantic models also serialize cleanly to dictionaries and JSON.
```

**Replace with:**
```
Pydantic models also serialize cleanly to dictionaries and JSON. You'll use this any time validated output needs to leave the model — logging an agent result, sending it to another API, or saving it to disk. Use `model_dump()` when passing data to other Python code; use `model_dump_json()` when sending output to an API or saving to a file.
```

---

## Change 3: Reframe "Why Pydantic Instead of Raw JSON Schemas"

Two reviewers flagged this. Students at this point have never encountered "raw JSON schemas" as a noun for a validation format — the comparison feels like one side of a conversation they missed. Grok's framing is stronger: ground the comparison in the SDK and reframe the header.

> ⚠️ NOTE: The Find text below may have drifted. Apply semantically — locate the markdown cell that explains why the course uses Pydantic (it begins with a comparative reason or a header mentioning JSON schemas or "Why Pydantic"), then apply the replacements below.

**Find (the "Why Pydantic Instead of Raw JSON Schemas" markdown cell header):**
```
## Why Pydantic Instead of Raw JSON Schemas
```

**Replace with:**
```
## Why This Course Uses Pydantic
```

**Find (the first line of that same cell):**
```
This course prefers Pydantic over raw JSON schemas for three reasons:
```

**Replace with:**
```
The Agents SDK can accept either Pydantic models *or* hand-written JSON schemas for output validation. This course uses Pydantic for three reasons:
```

---

## Change 4: Bridge `ValidationError` to the agent context

One reviewer flagged this. The ValidationError section correctly shows what happens when bad data arrives — but doesn't connect the dots to *why this matters in an agent workflow*. One sentence makes validation errors feel like a feature, not a chore.

**Find (in the Part 2 "Validation in Action" section):**
```
If the data doesn't match the model, Pydantic raises a `ValidationError` with a clear explanation of what went wrong.
```

**Replace with:**
```
If the data doesn't match the model, Pydantic raises a `ValidationError` with a clear explanation of what went wrong. In agent workflows, this is the safety net that catches malformed output before your next tool call or downstream code tries to use it.
```

---

## Change 5: Add "(and hope it listens)" to the prompt-only constraint comparison

> ⚠️ **Apply only if course tone is informal.** If the course uses a consistently informal, conversational tone, apply this change. If the course style is more neutral or professional, skip it.

One reviewer flagged this — a tiny tonal change that makes the unreliability of prompt-only constraints land viscerally rather than abstractly.

**Find (in the Part 3 "Why This Works" cell):**
```
there's no need to ask the agent to "please return a rating between 1 and 5."
```

**Replace with:**
```
there's no need to ask the agent to "please return a rating between 1 and 5" (and hope it listens).
```
