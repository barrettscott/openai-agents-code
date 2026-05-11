# cc_nb02.md — 02_OpenAI_Setup.ipynb

**Notebook:** `/Users/scott/Dropbox/Notebooks/openai-agents/Week_1_Foundations/02_OpenAI_Setup.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Answer "what actually stops charges?" after the soft-limit warning

The budget alert callout tells students the limit won't automatically stop requests — but then leaves them hanging. A risk-averse student who just added a credit card needs to know the prepaid credit balance is their real cap. One sentence closes the loop.

**Find:**
```
⚠️ **Note:** This is a soft limit for alerts only — it won't automatically stop requests
```

**Replace with:**
```
⚠️ **Note:** This is a soft limit for alerts only — it won't automatically stop requests. Your prepaid credit balance is the real cap — when it runs out, requests stop until you top up. The budget alert is just an early warning.
```

---

## Change 2: Add the standard course comment to the MODEL constant

`gpt-5-mini` appears for the first time here with no context. The standard DG inline comment answers the silent "why this one?"

**Find:**
```
MODEL = "gpt-5-mini"
```
*(in the Step 6 code cell)*

**Replace with:**
```
MODEL = "gpt-5-mini"  # Course default — all standard agents
```

---

## Change 3: Orient Step 4 before the multi-path setup details

After five screens about accounts, billing, hidden-file conventions across three OSes, and two setup methods, the student has lost the thread. A single orienting sentence at the top of Step 4 re-anchors the whole section to its simple goal.

**Find (at the start of the Step 4 markdown cell):**
```
Create a `.env` file in the course root so the notebook can load your API key.
```

**Replace with:**
```
Create a `.env` file in the course root so the notebook can load your API key.

All we need is a file named `.env` in the course root containing `OPENAI_API_KEY=sk-…`. Use whichever method feels easiest.
```

---

## Change 4: Explain `load_dotenv` arguments in the verification cell

Notebook 01 used `load_dotenv` with no arguments. Now `override=True` and `dotenv_path` appear — without a comment the student doesn't know whether to copy these parameters forever. The inline comment answers the question.

**Find:**
```
load_dotenv(dotenv_path=env_path, override=True)
```

**Replace with:**
```
load_dotenv(dotenv_path=env_path, override=True)  # dotenv_path: location of .env; override=True: new key replaces any old one in memory
```

---

## Change 5: Resolve the "same pattern" sentence — softened version

> ⚠️ VERIFY FIRST: Check whether the sentence *"This same pattern (load key → create agent → call `Runner.run()`) is what every notebook in the course will use."* is currently present in Step 6 or not. Then apply the appropriate fix below.
>
> - **If the sentence IS present** (Gemini's version): the fix is to replace it with the softened form below — this resolves the conflict between "don't worry about the code" and "remember this pattern."
> - **If the sentence IS NOT present** (Grok's version): add the softened form after the success print — students need a takeaway skeleton even if they don't yet understand the code.

**Either find (if present):**
```
This same pattern (load key → create agent → call `Runner.run()`) is what every notebook in the course will use.
```

**Or add after the `print("\n✅ You're ready for the course!")` line (if absent).**

**Target text either way:**
```
You'll see this same shape — load key → create Agent → run — in every notebook from here on. Lesson 03 walks through it in detail.
```
