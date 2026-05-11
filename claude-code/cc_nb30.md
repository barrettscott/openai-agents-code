# cc_nb30.md — 30_Project_Structure_And_CLI.ipynb

**Notebook:** `/Users/scott/Library/CloudStorage/Dropbox/Notebooks/openai-agents/Week_5_MCP_And_Production/30_Project_Structure_And_CLI.ipynb`

Apply each change below to the notebook. Use NotebookEdit for cell edits.

---

## Change 1: Fix Part 7 → Part 6 numbering

3 reviewers (Anthropic, Grok, DeepSeek) flagged this. Delivery notes explicitly call it out: "Fix the numbering or restore the missing Part 6 heading." Students scroll back looking for a section they think they missed.

**Find:**
```
## 🌐 Part 7: When to Consider a Simple API Wrapper
```

**Replace with:**
```
## 🌐 Part 6: When to Consider a Simple API Wrapper
```

> ⚠️ VERIFY: Check whether any other cells reference "Part 7" by number (e.g., in the streaming "Why This Works" cell or any table of contents). Update those references to "Part 6" as well.

---

## Change 2: Explain streaming event types in "Why This Works" cell

5/5 reviewers flagged this as the top finding. Three new types (`RawResponsesStreamEvent`, `ResponseTextDeltaEvent`, `data.delta`) appear with no description. Students can copy the pattern but can't adapt it.

Add the following sentences to the "Why This Works" cell for Part 4, after the existing explanation:

```
The stream yields many event types — tool calls, metadata, and text chunks. We filter for `RawResponsesStreamEvent` → `ResponseTextDeltaEvent` because `data.delta` is the next chunk of text from the model. Print it immediately to stream output as it arrives. To also surface tool calls, listen for `RunItemStreamEvent` + `ToolCallItem` in the same loop.
```

---

## Change 3: Remove unused imports from Part 4 streaming template

1 reviewer (DeepSeek), but this is a template that students will copy verbatim. Unused imports in a template teach bad habits and erode trust in the pattern. Clean templates are non-negotiable.

**Find:**
```
from config import MODEL
from tools import my_tool
from agent import agent
```

(in the Part 4 streaming template string)

**Replace with:**
```
from agent import agent  # agent is already configured in agent.py — no extra imports needed
```

> ⚠️ VERIFY: Confirm that `MODEL` and `my_tool` are genuinely unused in the streaming template (only `agent` is passed to `Runner.run_streamed()`). If the template does use them, keep the imports. Also confirm that `Runner` and stream event imports (`RawResponsesStreamEvent`, `ResponseTextDeltaEvent`) are NOT part of the Find text — those ARE used and must remain. This change should only remove the `MODEL` and `my_tool` lines.

---

## Change 4: Add SDK-vs-OpenAI-package relationship sentence in Part 5

3 reviewers (Anthropic, OpenAI, Gemini) flagged that students don't know what `from openai import OpenAI` is or how it relates to `from agents import Agent, Runner`. Without this, students copying the `RESPONSES_TEMPLATE` may hit auth errors without understanding why.

Add the following sentence to the Part 5 intro markdown cell, before the "Drop down" paragraph:

```
The Agents SDK is a higher-level library built on the core `openai` package — `client.responses.create()` is the lower-level API call underneath the agent patterns you've used throughout this course.
```

---

## Change 5: Add API key comment to `RESPONSES_TEMPLATE`

1 reviewer (Anthropic), but blocks transfer with a concrete error: a student copying this template into a new project without `config.py` imported will get a confusing auth error. One comment prevents it.

**Find:**
```
client = OpenAI()
```

(inside the `RESPONSES_TEMPLATE` string in Part 5)

**Replace with:**
```
client = OpenAI()  # reads OPENAI_API_KEY from the environment — config.py already loaded it
```

---

## Change 6: Add "why files matter" sentence to Part 3 intro

1 reviewer (OpenAI), but the delivery notes also call for connecting the structure to real work. Students may treat the four-file split as packaging convention rather than a practical shift.

**Find:**
```
These are printed templates you can copy directly into real project files — they are not executed in this notebook.
```

**Replace with:**
```
These are printed templates you can copy directly into real project files — they are not executed in this notebook. This matters because once your agent lives in files instead of notebook cells, you can run it from the terminal, import it into other apps, and change one part without touching the rest.
```

---

## Change 7: Add decision rule between Part 5's two paragraphs

1 reviewer (OpenAI), but the delivery notes explicitly call for "clarify when to stay with agents vs lower-level client calls." The section currently introduces two ideas at once with no rule of thumb.

Add the following sentence between the two paragraphs in Part 5 (between "Use it by default" and "Drop down to..."):

```
The decision is simple: if the model needs to decide what to do next, use an agent; if you already know the exact one-shot call you want, use the Responses API directly.
```

---

## Change 8: Add workflow note above Practice Exercises

1 reviewer (Gemini), but this is a real blocker: all prior work has been inside notebooks and students have no idea whether to use `pathlib` in a cell or open a terminal. The delivery notes frame this lesson as "turning notebook cells into files."

Add the following note as a markdown cell immediately above the Practice Exercises code cell:

```markdown
**Note:** For these exercises, work outside Jupyter — open a terminal, create the `my_agent/` directory, and create the `.py` files with a text editor, copying the templates from this notebook.
```

---

## Change 9: Add streaming "why" sentence anchored to MCP context

1 reviewer (Grok), but the delivery notes call for contrasting streaming with "waiting for the full run." One sentence ties streaming to the specific pain it solves.

**Find:**
```
It's a small fixed pattern you copy once. Lesson 32 extends this streaming pattern into a Gradio chat interface.
```

**Replace with:**
```
Users see progress immediately instead of waiting in silence for the entire MCP research + file-write task to finish. It's a small fixed pattern you copy once. Lesson 32 extends this streaming pattern into a Gradio chat interface.
```
