# cc_nb07.md — 07_Structured_Outputs.ipynb

**Notebook:** `/Users/scott/Dropbox/Notebooks/openai-agents/Week_1_Foundations/07_Structured_Outputs.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Fix the failure message in the Problem cell

One reviewer flagged this. The printed message says "format varies every run" — but what was actually shown is a `TypeError` from dict-indexing a string. These are two different failure modes. The message should match what was demonstrated rather than claiming something the student didn't see.

**Find:**
```
print("❌ Can't access fields from a string — format varies every run")
```

**Replace with:**
```
print("❌ result.final_output is a string — there's no field to access")
```

---

## Change 2: Clarify "silently passing through" in the Part 1 intro

One reviewer flagged this. A new agent builder doesn't yet think in pipeline failure modes and won't grasp what's being avoided with the current abstract phrasing. The replacement is concrete.

**Find:**
```
invalid outputs surface as errors instead of silently passing through.
```

**Replace with:**
```
invalid outputs surface as errors instead of reaching your Python code in the wrong shape and breaking later.
```

---

## Change 3: Add field-design guidance to the Part 1 intro

One reviewer flagged this. Students can copy the `SentimentResult` demo but can't answer "what should go in my schema?" The natural mistake is to include everything the agent could say rather than what the next step in their code needs. One sentence installs the right heuristic.

**Find (in the Part 1 intro markdown cell):**
```
Define a Pydantic model, pass it to the agent via `output_type=`,
```

Add the following sentence at the end of the introductory paragraph:

```
In your own projects, define fields based on what the next step in your code needs — not everything the agent could say.
```

---

## Change 4: Add field names as prompt — to the Part 1 "Why This Works" cell

One reviewer (Gemini) flagged this, and it's a genuine quality issue. Students don't realize that field names are instructions to the model. They may write terse names like `rsn` instead of `reasoning` and get degraded output without understanding why.

**Find (in the Part 1 "### 💡 Why This Works" cell):**
```
`result.final_output` is now an instance of `SentimentResult`, not a string.
```

**Replace with:**
```
`result.final_output` is now an instance of `SentimentResult`, not a string. The model also uses your field names as instructions, so descriptive names like `reasoning` produce better results than short names like `rsn`.
```

---

## Change 5: Add validation failure behavior to the Part 1 "Why This Works" cell

One reviewer flagged this. Students learned `Field(ge=0.0, le=1.0)` in NB06 and will naturally wonder: what if the model returns `confidence=2.5`? The happy path is all they see. A one-liner on what happens at validation failure prevents a "why is my code exploding?" moment later.

> ⚠️ **Verify actual SDK behavior before applying** — if constraint violations surface as a retry/repair loop rather than a `ValidationError`, adjust wording accordingly. The instruction below assumes direct `ValidationError` raising; confirm this against the actual SDK behavior before committing.

Add one sentence to the Part 1 "Why This Works" cell (after the field-names sentence added in Change 4):

```
If the model returns data that violates a `Field` constraint, the SDK raises a validation error instead of passing through invalid output.
```

---

## Change 6: Add "structured outputs guarantee shape, not truth" — from delivery notes

The delivery notes flag this as a highest-priority edit that the NBR didn't catch. Students may think that because the output matches the schema, the *facts* in it are reliable. That's a fundamental misunderstanding of what structured outputs do. Add a clarifying sentence to the Part 1 "Why This Works" cell.

Add one sentence to the Part 1 "Why This Works" cell:

```
Note: structured outputs guarantee *shape*, not truth — the model still generates the values, so validate facts separately if correctness matters.
```

---

## Change 7: Add "when to use" guidance — landing the promised Topics topic

One reviewer (Anthropic) flagged this, backed by the Topics list: "When structured outputs help most." The notebook currently has a clear *how* but no answer to *should I use this everywhere?* A new student's instinct is to put `output_type=` on every agent — including chat-style ones where free text is correct. Add one bullet to Key Takeaways.

> ⚠️ NOTE: The Find text for the Key Takeaways header may have drifted. Apply semantically — locate the Key Takeaways markdown cell at the end of the notebook, then add the new bullet to its list.

**Find (in the Key Takeaways cell):**
```
## 🔑 Key Takeaways
```

Add a new bullet to the Key Takeaways list:

```
- **Use structured outputs whenever the response feeds into code** — free-form text is the right choice for human-facing chat replies
```

---

## Change 8: Add real-world grounding to the Optional fields "Why This Works" cell

One reviewer flagged this. Students see fields can be `None` but don't know when that's the right design choice. "Real inputs are messy" is the practical answer, and one sentence makes optional fields feel like a tool for realistic data rather than an edge case.

**Find (in the Part 2 "### 💡 Why This Works" cell):**
```
`Optional` fields make missing data explicit — downstream code can safely distinguish "absent" from "wrong" without string parsing.
```

**Replace with:**
```
`Optional` fields make missing data explicit — downstream code can safely distinguish "absent" from "wrong" without string parsing. Use this pattern when real input is messy — resumes, emails, notes, and forms often contain some fields but not others.
```
