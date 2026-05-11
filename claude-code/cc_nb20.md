# cc_nb20.md — 20_Persistent_Memory.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_4_Memory_Safety_Observability/20_Persistent_Memory.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Fix cell ordering bug in Part 3 (NameError blocker)

All 5 reviewers flagged this. The first code cell in Part 3 calls `await noisy_session.get_items()` before `noisy_session` is defined, causing a `NameError` that breaks the lesson for any student running cells in order.

Move the "Build a Noisy Session" code cell (and its markdown header) to appear **before** the `# First: inspect what's in a session` inspection cell. The inspection cell becomes redundant once the noisy session is built — consider merging it with the existing `items_before = await noisy_session.get_items()` call three cells later rather than keeping it as a standalone cell.

---

## Change 2: Bridge from demo to "what I'd write in my own code" in Part 3

3 of 5 reviewers (Grok, DeepSeek, OpenAI) flagged this. The Why This Works cell tells students to "set a turn count threshold" but never says where that logic lives in their own code, and doesn't explain why a separate summarizer agent is used.

**Find** (in the Part 3 "Why This Works" cell, near the end):
```
Apply this pattern proactively — set a turn count threshold (e.g. every 20 turns) rather than waiting for quality to degrade.
```

**Replace with:**
```
Apply this pattern proactively — set a turn count threshold (e.g. every 20 turns) rather than waiting for quality to degrade. In your own code, maintain a counter on the session (or check `len(await session.get_items())`) and trigger the summarize → clear → store sequence when it crosses your threshold. A separate summarizer agent keeps concerns clean — the assistant focuses on the user, the summarizer compresses history.
```

---

## Change 3: Add motivation for why even a clean session needs summarizing

2 of 5 reviewers (Anthropic, Gemini) flagged this. The demo shows 5 coherent turns getting pruned; students wonder why they'd compress a session that looks fine.

Add the following sentence between the Part 3 intro and the "Build a Noisy Session" demo:

```
Even a clean session like the one below grows unbounded — every turn costs tokens, and once the history exceeds the model's context window, older facts start dropping silently. Compress proactively, not reactively.
```

---

## Change 4: Explain why the Part 4 correction actually works

3 of 5 reviewers (Gemini, OpenAI, Grok) flagged this. Students may attribute the successful correction to model intelligence rather than the specific instruction line, and don't know what to do when conflicts repeat over time.

Add the following two sentences to the Part 4 "Why This Works" cell:

```
The correction works because of the instruction line `If a user corrects an earlier fact, treat the most recent statement as the current truth.` — without it, the agent sees two conflicting facts and may get confused. Include this once in your agent's instructions and the persistent session keeps the latest version automatically; for repeated conflicts, fall back to the summarize → clear → store pattern from Part 3.
```

---

## Change 5: Acknowledge the "user-controlled forgetting" gap

1 reviewer (Anthropic) flagged this as transfer-blocking. The Topics list promises "user-controlled forgetting" but the lesson only delivers correction. Students building real assistants will hit "forget that" requests.

Add the following sentence to the Part 4 section on "Conflicting Memory and Forgetting":

```
The SDK doesn't support per-fact deletion — for true forgetting, the practical pattern is to summarize the session, drop the unwanted fact during summarization, then clear and restore.
```

---

## Change 6: Tell students how to prevent sensitive data from being persisted

1 reviewer (DeepSeek) flagged this as transfer-blocking. The security note says "never store sensitive data" but sessions auto-store everything — students leave not knowing how to prevent it.

Add the following sentence immediately after the security note in Part 4 ("What NOT to Store"):

```
In practice, filter sensitive input before passing it to the agent, or use output guardrails (Lesson 22) to detect and block sensitive data from being persisted.
```
