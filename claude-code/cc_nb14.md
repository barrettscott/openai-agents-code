# cc_nb14.md — 14_Handoffs.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_3_Multi_Agent_Systems/14_Handoffs.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Explain why triage doesn't get control back

2 reviewers (Anthropic, DeepSeek) flag this as a rule stated without rationale. Students wonder if it's a feature or a limitation, and can't evaluate when to use handoffs vs. agents-as-tools (Lesson 15) without understanding the design choice.

**Find:**
```
The triage agent does not get control back.
```

**Replace with:**
```
The triage agent does not get control back. This is intentional — the specialist owns the conversation once it starts, so triage can't re-route or second-guess. If the specialist needs more information, it asks the user directly.
```

---

## Change 2: Name the agent-name → tool-name connection

1 reviewer (Anthropic), but blocks transfer and the fix is one sentence. Students see `BillingAgent` in the definition and later `transfer_to_billing_agent` in the Why This Works cell without ever learning the connection.

Add the following sentence after the markdown intro before the three specialist agent definitions (the cell introducing `billing_agent`, `tech_support_agent`, `refunds_agent`):

```
The agent `name` becomes part of the auto-generated handoff tool name (e.g., `BillingAgent` → `transfer_to_billing_agent`), so pick names that read well as routing destinations.
```

---

## Change 3: Explain that routing is prompt-driven, not enforced

2 reviewers (Anthropic, DeepSeek) flag this. Students don't know if the SDK enforces handoff or if it's just a prompt instruction — critical for understanding how reliable their own triage systems will be.

**Find** the Why This Works cell in Part 2 (the one beginning with "The SDK automatically creates `transfer_to_billing_agent`..."). Append the following sentence:

```
Routing is prompt-driven, not enforced — clear, narrow triage instructions are what keep the agent from answering directly. If `result.last_agent.name` is still the triage agent, the handoff didn't happen.
```

---

## Change 4: Explain `result.last_agent` and motivate it

2 reviewers (OpenAI, Grok) flag this from different angles: Grok notes `last_agent` is new (students only know `final_output` from prior notebooks), OpenAI notes even after understanding it, the sentence reads like trivia without motivation.

**Find:**
```
`result.last_agent.name` tells you which agent produced the final response.
```

**Replace with:**
```
The `RunResult` returned by `Runner.run` has a `last_agent` attribute — `result.last_agent.name` tells you which agent produced the final response. This is how you debug routing mistakes and verify the triage agent is sending requests to the right specialist.
```

---

## Change 5: Clarify "more control" in the Part 4 intro

1 reviewer (OpenAI), but the fix is a single word substitution and "more control" is genuinely vague — the student doesn't know if control refers to the transfer itself, the tool name, or the selection criteria.

**Find:**
```
Use the `handoff()` function when you need more control — a custom description tells the triage agent exactly when to use each specialist.
```

**Replace with:**
```
Use the `handoff()` function when you need more control over how each handoff is described to the triage agent — a custom description tells it exactly when to use each specialist.
```

---

## Change 6: Set expectation before the ambiguous-message comparison

2 reviewers (Anthropic, DeepSeek) flag that a single run may show identical routing for v1 and v2, which visibly contradicts the lesson's conclusion. Students watch identical output and can't tell whether they witnessed the lesson or not.

Add the following sentence immediately before the comparison code cell (the cell with `ambiguous_message = "I need help with my account settings."`):

```
You may need to run this a few times — on borderline messages like this one, v1 routing can vary while v2 stays consistent.
```

---

## Change 7: Add transferable rule for when/how to use custom descriptions

4 reviewers (OpenAI ×2, Gemini, Grok) converge on the same gap: no decision rule for plain `handoffs=[agent]` vs. `handoff(..., tool_description_override=...)`, and no principle for what to write. Highest-leverage fix in the notebook.

**Find** the Why This Works cell in Part 4 (the one containing "Custom `tool_description_override` text gives the triage agent more explicit routing criteria..."). Append the following two sentences:

```
Use plain `handoffs=[...]` when the agent names already make the routing obvious; reach for `handoff(..., tool_description_override=...)` when requests could plausibly fit two specialists. Write the override text in terms of user intent and example request types, not internal team names — the model uses this text to choose.
```
