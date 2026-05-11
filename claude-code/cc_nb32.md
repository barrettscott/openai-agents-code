# cc_nb32.md — 32_Deploying_With_Gradio.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_5_MCP_And_Production/32_Deploying_With_Gradio.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Add concrete HF Spaces deployment steps to Part 6

3 reviewers (Anthropic, OpenAI, Gemini) flagged this as the top finding. The deployed agent is the headline outcome of the entire course, and the actual workflow is skipped. Students cannot complete Exercise 2 without these steps.

**Find:**
```
A Hugging Face Space is a free hosted environment for Gradio apps. Create a Space, push your files, and your agent gets a public URL.
```

**Replace with:**
```
Hugging Face is a central platform for the AI community; a Space is a free hosted environment for Gradio apps. To deploy:

1. Sign in at [huggingface.co](https://huggingface.co)
2. Click **New Space**, give it a name, and choose **Gradio** as the SDK
3. Upload your files via the web UI drag-and-drop, or `git push` to the Space's repo
4. Add your `OPENAI_API_KEY` under **Settings → Repository secrets**
5. The Space builds automatically and gets a public URL

Your agent gets a shareable link as soon as the build completes.
```

---

## Change 2: Add tool visibility note above `APP_PY` deployment template

1 reviewer (Anthropic), but high blast radius — affects any student deploying a tool-using agent (Capstones 1, 3, or 4). The `APP_PY` template combines streaming + failure handling but silently drops the `RunItemStreamEvent` block from Part 4. Students won't know to add it back.

Add the following sentence immediately above the `APP_PY` template:

```
This template combines streaming and failure handling. If your agent uses tools, copy the `RunItemStreamEvent` block from Part 4 into this `chat()` function so tool activity is visible in the UI.
```

---

## Change 3: Add `history` object shape one-liner in Part 2

2 reviewers (Gemini, Grok) flagged that the history snippet uses `h['role']` and `h['content']` without ever stating what Gradio passes. Students can copy but can't adapt or debug.

Add the following sentence immediately before the `context = "\n".join(...)` snippet in the history-vs-memory cell:

```
Gradio passes `history` as a list of `{'role': 'user'|'assistant', 'content': text}` dicts (most recent last).
```

---

## Change 4: Add history-vs-sessions decision rule

1 reviewer (OpenAI), but this is a genuine transfer gap: the notebook shows the mechanics of building context from history but never says when to use this pattern vs. sessions. Directly affects students building their own chat apps.

Add the following sentence immediately before the `context = "\n".join(...)` snippet (before the shape one-liner from Change 3):

```
Use this pattern for lightweight short-term context in a simple chat app; use sessions when you want the SDK to manage conversation state for you.
```

---

## Change 5: Add progressive-build orienting sentence at start of Part 2

2 reviewers (OpenAI, Grok) flagged that by Part 5, students lose track of which version is "the real one" to start from. One orienting sentence at the entry point prevents four sections of accumulating confusion.

Add the following sentence to the Part 2 intro markdown cell:

```
These next four parts build the same app step by step — start with Part 2, then layer on streaming, status updates, and error handling. Part 6's deployment template combines them all.
```

---

## Change 6: Replace "Bridge" with plain-English action in Part 3

1 reviewer (OpenAI), trivial wording change. "Bridge" reads like a known technical pattern rather than a plain action.

**Find:**
```
Bridge `Runner.run_streamed()` from Lesson 30 into this pattern.
```

**Replace with:**
```
Use `Runner.run_streamed()` and pass each text chunk into Gradio with `yield`.
```

---

## Change 7: Cut the Part 4 forward-reference to Part 5's `try/except`

1 reviewer (Anthropic). The sentence references code that isn't in Part 4, causing students to scroll back looking for something that isn't there. Removing it is cleaner than any other fix.

**Find:**
```
For failure handling, the outer `try/except` in Part 5 catches tool errors and shows a friendly message rather than crashing the app.
```

**Replace with:**
*(delete the sentence entirely)*

---

## Change 8: Add tool-visibility "why it matters" sentence in Part 4

1 reviewer (OpenAI), but delivery notes also call for making the user-experience point explicit. The current phrasing makes tool visibility sound cosmetic.

**Find:**
```
Adding a brief status update makes the experience feel responsive and shows that something is happening.
```

**Replace with:**
```
Adding a brief status update makes the experience feel responsive and shows that something is happening. This matters most when tools are slow — without a status update, users often assume the app froze or the request failed.
```

---

## Change 9: Reframe the `config.py` / `os.getenv()` deployment note

1 reviewer (Anthropic). The current phrasing "there is no `config.py`" reads as a platform constraint, leaving students uncertain whether HF Spaces forbids the Lesson 30 project structure. Delivery notes call for clarifying local `.env` vs hosted secrets.

**Find:**
```
On Hugging Face Spaces, there is no `config.py` — constants come from environment variables set in the Space settings.
```

**Replace with:**
```
We're packaging this as a single `app.py` for deployment simplicity — you can still use the Lesson 30 module structure on Spaces by uploading those files alongside `app.py`. The change here is reading `MODEL` from `os.getenv()` so it can be set via Space secrets rather than a local `.env` file.
```

---

## Change 10: Add streaming event type conceptual frame in Part 3

2 reviewers (Gemini, Grok) flagged this. DeepSeek explicitly disagreed, arguing Lesson 30 already introduced these classes. `cc_nb30 Change 2` is expected to add a conceptual explanation of `RawResponsesStreamEvent` and `ResponseTextDeltaEvent` in Notebook 30's "Why This Works" cell — if that change was applied, **skip this change entirely** to avoid duplicating the same explanation verbatim in both lessons.

> ⚠️ VERIFY before applying: Read the Part 4 "Why This Works" cell in the updated Notebook 30. If it already names and explains `RawResponsesStreamEvent` and `ResponseTextDeltaEvent` with a conceptual frame (not just copy-paste code), skip this change and keep Lesson 32 UI-focused. Only apply if Notebook 30 still lacks the conceptual explanation.

If applying, add the following sentence to the Part 3 intro paragraph, after the `Runner.run_streamed()` reference:

```
`Runner.run_streamed()` emits different event types — `RawResponsesStreamEvent` carries text deltas, `RunItemStreamEvent` + `ToolCallItem` signal tool activity — so you can react as the run unfolds rather than waiting for the full result.
```
