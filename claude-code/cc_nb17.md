# cc_nb17.md — 17_Debate_And_Critique.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_3_Multi_Agent_Systems/17_Debate_And_Critique.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Add critique-vs-debate decision rule

3 reviewers (Anthropic, OpenAI, Gemini) flag this. Both Part 3 and Part 4 produce refined output through multi-agent dialogue, but students leave without a decision rule for picking between them. The delivery notes also call for a clean "critique vs debate" definition. Highest-leverage fix in the notebook.

Add the following sentence at the top of the Part 4 intro cell (immediately after the `## 🥊 Part 4: True Debate — Two Opposing Positions` header):

```
Use the critique loop to refine a single output toward a quality bar; use a true debate when the question has legitimately opposing positions and you want both surfaced before deciding.
```

---

## Change 2: Explain the REASONING_MODEL switch on the judge

2 reviewers (Anthropic, DeepSeek) flag this. Every prior agent in the notebook uses `MODEL`; the quality judge silently switches to `REASONING_MODEL` with no rationale. The delivery notes reinforce that Lesson 03 promises coverage of when to pick each model — this is the reinforcement beat.

There is no existing dedicated markdown cell introducing the `quality_judge_agent` — only the "Run the Critique Loop" header cell follows the agent definition. Add the explanation using one of these approaches:

- Add a new markdown cell immediately before the `quality_judge_agent` code cell with the following text:

```
Judging quality is a reasoning task — we use `REASONING_MODEL` here because the judge needs to evaluate the explanation carefully, while the writer can stay on the cheaper `MODEL`.
```

- OR add the following as an inline comment `# Opus-class reasoning needed for nuanced quality judgment` above the `model=REASONING_MODEL` line in the `quality_judge_agent` code cell.

> ⚠️ VERIFY: The `synthesis_agent` in Part 4 also uses `REASONING_MODEL`. If that agent has its own introductory sentence or comment, add a matching half-sentence there: "We use `REASONING_MODEL` for the same reason as the judge — synthesis requires careful weighing of both sides."

---

## Change 3: Improve the "max iterations reached" message

2 reviewers (Anthropic, Gemini) flag this. The current message names a problem and walks away. Students are left wondering: is the returned draft trustworthy? Should they re-score? The delivery notes call for making termination rules "highly visible."

**Find:**
```
print(f"\n⚠️ Max iterations reached — final draft was not re-scored")
```

**Replace with:**
```
print(f"\n⚠️ Max iterations reached — returning the most recent revision. Treat `max_iterations` as a cost cap, not a quality guarantee. Consider raising iterations or lowering the threshold.")
```

---

## Change 4: Replace `for…else` with a tracked boolean

1 reviewer (DeepSeek), but blocks transfer per reviewer. `for…else` is uncommon Python — a student replicating this loop may put the `else` in the wrong place and ship broken termination handling without knowing it. The fix removes non-obvious syntax without adding length.

**Find:**
```python
for iteration in range(1, max_iterations + 1):
```
(and the matching `else:` clause at the end of the loop)

**Replace with** the equivalent structure using a tracked boolean. The new shape should be:

```python
tripped = False
for iteration in range(1, max_iterations + 1):
    # ... existing loop body unchanged ...
    if score >= quality_threshold:
        print(f"✅ ...")
        tripped = True
        break

if not tripped:
    print(f"\n⚠️ Max iterations reached — returning the most recent revision. Treat `max_iterations` as a cost cap, not a quality guarantee. Consider raising iterations or lowering the threshold.")
```

> ⚠️ VERIFY: Read the full `critique_loop` function body before applying to ensure the `break` statement and `else` clause are exactly where the review describes. The `if not tripped:` block replaces the `else:` clause; the Change 3 message text should be used here (not the old message text).

---

## Change 5: Frame the parallelism in Part 4 as an optimization, not the pattern

2 reviewers (OpenAI, Gemini) flag this. Students are processing a new debate pattern and parallel execution simultaneously, and can't tell whether `asyncio.gather` is essential to the debate or just an optimization from Lesson 16.

**Find** the Part 4 Why This Works cell. Append the following sentence:

```
Parallelism here is an optimization carried over from Lesson 16 — the advocate and skeptic argue independently, so we run them with `gather`. The debate pattern itself is about opposing roles and synthesis, not parallelism.
```
