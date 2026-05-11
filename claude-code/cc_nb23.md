# cc_nb23.md — 23_Prompt_Injection_And_Tool_Safety.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_4_Memory_Safety_Observability/23_Prompt_Injection_And_Tool_Safety.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Reframe the injection demo "Why This Matters" cell

2 of 5 reviewers (Anthropic, DeepSeek) flagged this as transfer-blocking. When the modern model resists the obvious injection string (likely), students read the hedge and conclude the threat isn't real — and won't apply Part 2's defenses.

**Find** (in the "Why This Matters" cell after the Part 1 naive-agent demo):
```
Modern models often resist obvious injections — but attackers get more sophisticated over time. The architectural risk is real regardless of whether this specific string succeeds.
```

**Replace with:**
```
Whether or not the model falls for this specific string, the architecture is the problem — the agent has no built-in way to tell instructions in your prompt from instructions buried in fetched content. Modern models often resist obvious injections, but attackers get more sophisticated over time and the structural vulnerability remains.
```

---

## Change 2: Define "idempotency keys"

4 of 5 reviewers (Anthropic, OpenAI, Gemini, Grok) flagged this — the strongest consensus finding in this review. "Idempotency key" appears right after "idempotent" is defined but is itself undefined; the guidance is unactionable without knowing what a key actually is.

**Find** (in Part 4, the bulleted guidance):
```
use idempotency keys where the API supports them.
```

**Replace with:**
```
use idempotency keys where the API supports them (a unique request ID — often a UUID — you send with the call so the API can recognize and ignore duplicates; Stripe, for example, uses one so a retried charge isn't billed twice).
```

---

## Change 3: Add the confirmation round-trip mechanic to Part 5 "Why This Works"

3 of 5 reviewers (Anthropic, OpenAI, Grok) flagged this as transfer-blocking. The demo stops at the preview; students never see the second turn where the user says "yes" and the tool executes — so they can't wire the pattern into a real app.

Add the following sentence to the Part 5 "Why This Works" cell:

```
In a real app the user's 'yes' arrives as a follow-up turn — call `Runner.run()` again with their response, and pair the pattern with a Session (Lesson 19) so the agent remembers the pending action across turns.
```

---

## Change 4: Tie retry danger back to agents specifically

2 of 5 reviewers (OpenAI, DeepSeek) flagged this. Part 4 reads like a generic backend rule; students don't see why it belongs in an agent safety lesson.

Add the following sentence at the close of Part 4 (Retries and Idempotency):

```
This matters for agents because they may retry tool calls automatically after a failure — so a non-idempotent tool can quietly repeat a real-world action like sending an email or charging a card.
```

---

## Change 5: Add inline comment explaining the path traversal check in Part 3

2 of 5 reviewers (Anthropic, Gemini) flagged this. The "good" read-only tool silently introduces a second security concept; students can't tell if the check is load-bearing for the lesson or whether their own file tools need it.

Add the following inline comment immediately above the path traversal check in the `read_file` definition:

```python
# Also block path traversal (e.g. "../etc/passwd") — least privilege isn't just read vs. write, it's also scoping where the tool can read from
if not safe_path.is_relative_to(workspace):
```

---

## Change 6: Add "watch for" marker before the Part 3 injection comparison

3 of 5 reviewers (Grok, DeepSeek, OpenAI) converged on this: the key comparison — same injection, different blast radius — gets buried under setup scaffolding. The DeepSeek placement (immediately before the payoff cell) is most surgical.

Add the following markdown sentence immediately before the final injection test cell in Part 3:

```
**Watch what happens when both agents receive the same injection input — one can delete, the other can't.**
```

---

## Change 7: Define "blast radius"

1 reviewer (OpenAI) flagged this. The term is used in Part 3 without definition; students can infer it but can't reuse or recognize the named principle.

Add the following gloss immediately after "Least-privilege limits the blast radius." in Part 3:

```
In security, blast radius means how much damage a failure or attack can cause.
```

---

## Change 8: Add answer hint to Exercise 2 reflection

1 reviewer (DeepSeek) flagged this. Students write a reflection answer but can't verify it — the learning point doesn't stick without confirmation.

Add the following hint after the Exercise 2 TODO 4 reflection block:

```
Answer: the agent cannot delete customer C001 because no delete tool exists. The tool surface enforces the policy.
```
