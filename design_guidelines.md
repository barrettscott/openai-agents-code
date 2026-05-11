# AI Agents with Python & the OpenAI Agents SDK

# Design Guidelines

---

## Highest-Priority Review Rules

When reviewing a notebook, apply these in order. A notebook that passes these four categories is in good shape even if it has minor style inconsistencies elsewhere in this document.

1. **Correctness and honesty.** Teaching patterns actually work. No claims the agent does something it doesn't. UI history is not presented as agent memory unless the notebook explicitly addresses the distinction. Security notes are present where required. Permission modes match the lesson's safety level.

2. **Section order.** End-of-notebook structure follows the standard order: Practice Exercises → Key Takeaways → Next Step → Troubleshooting → Course Complete (final notebook only). Course Complete, when present, always comes after Troubleshooting.

3. **Code hygiene.** Code runs cleanly. Imports match actual usage. No hardcoded model strings outside the constants block. Async handling is correct. Instructions of 2+ lines are assigned to a variable before the Agent() constructor.

4. **Teaching clarity.** Markdown is for student reading, not on-camera narration — this is the most common failure mode and the first thing to cut. One main idea per markdown cell. No wall-of-text. Benefits/problems lists kept to 3 bullets max (process flows, specs, and checklists exempt).

Below this section, rules are organized by topic. Formatting preferences (emoji usage, specific HTML wrappers, table formatting) are lower priority than the four categories above.

---

## Notebook Structure

**Opening Template:** Every notebook starts with:
- `# [Title]` (single-hash h1, title only — no lesson number in the title)
- `*Notebook [two-digit number]*` on its own line below the title (e.g., `*Notebook 04*`)
- Short intro paragraph (2–3 sentences max)
- **Topics:** bullet list — use this consistently across all notebooks. When a notebook intentionally narrows from a larger toolset or topic area, a scope-setting bullet is acceptable (e.g., "This notebook focuses on structured output and validation"). Rephrase as a topic rather than meta-commentary where possible.
- `---` divider immediately after the opening section

Example:

```
# Building Tools for Agents

*Notebook 04*

In this notebook, we'll turn Python functions into tools your agent can call...

**Topics:**
- Turning any Python function into a tool with `@function_tool`
- ...

---
```

**Setup rule (canonical — referenced from other sections):** Lesson 01 uses a full Environment Check with the header `## ✅ Step 1: Environment Check (Diagnostic Only)`. All other notebooks use `## 🔧 Setup` by default. Use `## 🔧 Step 1: Setup` only when the notebook has a genuinely sequenced setup phase where numbering adds clarity.

**The Framing:** A short 🎯 The Problem framing cell to contextualize the lesson. Omit if an opening demo already provides sufficient context — the cell exists to motivate the lesson, not to satisfy a structural requirement. Aim for two short sentences — one to set up the problem, one to land the key insight. When the lesson requires a scenario to make the stakes concrete (e.g., walking through a specific failure mode step by step), the cell may run longer — use judgment. If it reads like a lecture, cut; if it earns its length, keep it.

**Key Takeaways:** Use `## 🎯 Key Takeaways` near the end.

**Troubleshooting:** All notebooks use a single link to the course TROUBLESHOOTING.md file, using a `#####` heading to keep it visually unobtrusive:

```markdown
##### 🔧 [Troubleshooting Guide](https://github.com/barrettscott/openai-agents/blob/main/TROUBLESHOOTING.md#lesson-XX-title)
```

Lesson-specific troubleshooting content lives in TROUBLESHOOTING.md and is covered on camera by pointing students to that file. The link requires only a browser — students can access GitHub even if their Python environment isn't working yet.

---

## Section Order (End of Notebook)

The standard end-of-notebook section order is:

1. Practice Exercises
2. `---` divider
3. `## 🎯 Key Takeaways`
4. `---` divider
5. `## 📍 Next Step`
6. `---` divider
7. `##### 🔧 [Troubleshooting Guide](https://github.com/barrettscott/openai-agents/blob/main/TROUBLESHOOTING.md#lesson-XX-title)` (link-only, no section header)
8. `---` divider
9. Course Complete cell (only in the final notebook — Lesson 32)
10. `---` divider

**Lesson 01 exception:** Lesson 01 (Environment Check) omits Practice Exercises entirely (see Practice Exercises section). Its end-of-notebook order is Key Takeaways → Next Step → Troubleshooting.

**Course Complete placement:** When present, the Course Complete cell appears after Troubleshooting, never before it. Placing it between Key Takeaways and Next Step, or between Next Step and Troubleshooting, is a section-order violation.

**When to use Next Step vs Course Complete:**
- Every notebook except the final one uses a standard Next Step cell pointing to the next notebook by ID and title
- Only the final notebook in the course (Lesson 32) replaces Next Step with a Course Complete cell — this is the only case where Next Step is omitted entirely
- Week boundaries do not require special completion cells — students see week structure through the Udemy section interface; internal Week Complete cells would be redundant

**Example:** Lesson 18 is the last notebook in Week 3 but points forward to Lesson 19 — it uses a standard Next Step cell, not any form of completion cell. Lesson 32 is the last notebook in the course — it correctly uses Course Complete in place of Next Step.

**Next Step cell format:** The lesson title goes on its own bold line followed by two spaces (forcing a line break). A blank line separates it from the description sentence.

Example:

```
## 📍 Next Step

**Notebook 06: Pydantic Basics**  

Learn how to define typed models with Pydantic — the foundation for structured agent output in Lesson 07.
```

**Capstone notebooks:** Capstone Next Step cells may use the short identifier form (`**Notebook 13: Capstone #1**`) without appending the full subtitle. The `#N` identifier is already self-describing in a capstone context. Adding the full subtitle to the bold line is acceptable but not required.

---

## Template and Reference Notebooks

Some notebooks use Python string templates printed to output rather than runnable agent code. These are called template or reference notebooks (Lesson 30 — Project Structure & CLI — is the primary example, where students see reusable patterns for `config.py`, `tools.py`, and agent module structure).

**For template notebooks:**
- Standard runtime checks (Runner.run() spacing, Agent() constructor rules, import hygiene) apply only to the runnable setup cell and any directly executable code cells
- String template content is exempt from runtime-style checks — it will not be executed in the notebook
- Template content should still model the preferred patterns students are expected to copy: instructions assigned to variables before Agent(), no unused imports, and — when the template is meant to model a reusable project structure — constants for model names rather than hardcoded strings

The rationale: students copy templates directly. A template that demonstrates a weaker pattern teaches the weaker pattern.

The triple-quoted instructions rule (2+ lines must be assigned to a variable first) applies to runnable Agent() constructors in notebook code. String template content inside Python string literals (`CONFIG_TEMPLATE`, `AGENT_TEMPLATE`, etc.) is exempt from this check, though templates meant to be copied into real projects should still model the variable-assignment pattern.

---

## Practice Exercises

**Lesson 01 exemption:** Lesson 01 (Environment Check) does not require a Practice Exercises section. It is a pre-flight diagnostic lesson; students have no API key, no agent, and nothing meaningful to practice beyond the environment check itself. Do not flag Lesson 01 for a missing Practice Exercises section.

**Standard format — markdown cell:**

When there is one exercise:
```
## 💪 Practice Exercise

### [Title]

*Covers: [concept] — [brief description]*

[One sentence description]
```

When there are multiple exercises:
```
## 💪 Practice Exercises

### Exercise 1: [Title]

*Covers: [concept] — [brief description]*

[One sentence description]

### Exercise 2: [Title]

*Covers: [concept] — [brief description]*

[One sentence description]
```

**Rule:** Every exercise must include a `*Covers:*` italic line immediately after the title, on its own line, followed by a blank line. This ties the exercise back to a specific concept or part from the notebook — essential for instructors reading on camera. Format: `*Covers: [concept or Part N] — [brief description]*`

**Rule:** Do not add a description sentence after `*Covers:*` unless it provides scenario context or constraints the `*Covers:*` line can't capture. A sentence that merely restates the `*Covers:*` line in different words should be cut.
```
❌ Cut — restates *Covers:*:
*Covers: handoffs — routing between agents*
Route a question to the right specialist using handoffs.
✅ Keep — adds scenario context:
*Covers: handoffs — routing between agents*
You're building a triage agent for a customer service team with billing and technical specialists.
```

**Rule:** Do not add a separate intro sentence under the Practice Exercises header unless it adds essential context. In most notebooks, go directly into the first exercise.

**Standard format — code cell:**

When there is one exercise:
```python
# --------------------------------------------------------------
# 💪 [Title]
# --------------------------------------------------------------
# Objective: [One line description]

# TODO 1: [First task]
# hint or template

# TODO 2: [Second task]
# hint or template

# --- Write your code below this line ---
```

When there are multiple exercises:
```python
# --------------------------------------------------------------
# 💪 Exercise 1: [Title]
# --------------------------------------------------------------
# Objective: [One line description]

# TODO 1: [First task]
# hint or template

# TODO 2: [Second task]
# hint or template

# --- Write your code below this line ---
```

**Exercise scaffolding:** Provide hints, not solutions. Use TODO comments with brief hints.

**Capstone exercise pattern:** For capstone exercises, provide a partial scaffold — implement all phases except the final one, then make the TODO the meaningful extension. This keeps the exercise focused on the new concept rather than rote repetition. It is also acceptable to instruct students to copy classes or tools from previous notebooks — this emphasizes code reuse over rote typing.

**Design exercises for concept notebooks:** Concept notebooks and decision-framework notebooks — notebooks whose primary content is guidance, decision trees, or reference tables rather than runnable demos — may use structured design exercises instead of executable code. A design exercise is a code cell containing only comments that ask the student to reason through a decision or plan a project architecture in writing. No runnable code, no expected output.

This is an accepted pattern, not a workaround. For a notebook like Lesson 31 (Architecture Decisions), a design exercise in comments achieves the goal of active application more honestly than forcing a runnable code exercise onto content that is inherently about judgment rather than execution.

Design exercise cells must still follow standard cell formatting: comment header, objective line, TODO items, and a separator line before the student's workspace.

---

## Markdown Pacing Rules

- One main idea per markdown cell. Don't wall-of-text.
- No "on-camera narration." Keep markdown for student reading, not presenter scripts.
- **Benefits/Problems lists:** 3 bullets max (exceptions: process flows, technical specs, checklists). This is the canonical home for the bullet-limit rule.
- **Dividers:** Use `---` as a visual "scene cut" between major sections.
- **Prose before bullets:** When a section combines a short prose statement with a bullet list, put the prose first as a single sentence that sets up the list. Do not follow the bullets with additional prose — anything after the bullets will be spoken on camera and doesn't need to appear in the cell.
- Key Takeaways and Troubleshooting are student-read reference cells, not presented on camera — detail level is appropriate, length rules do not apply.
- **Short sentences and line breaks in prose cells:** Do not write long prose sentences that wrap on camera. Break intro paragraphs and explanatory text into short sentences, each on its own line with a blank line between them. This applies to: the opening cell, section intro cells (Part headers with explanatory text below them), Why This Works cells, and any other markdown cell with explanatory prose. A good test: if a prose line exceeds ~80 characters, it will likely wrap on camera and should be split.

  Instead of:

  ```
  Learn how agents decide which tools to call and why the instructions you write
  have a bigger impact than any parameter you set. In this notebook you'll see
  exactly how Agent, Runner.run(), and tool definitions fit together — and how
  a single change to the instructions completely changes what an agent does.
  ```

  Use:

  ```
  This notebook shows how three pieces fit together:

  `Agent`, `Runner.run()`, and tool definitions.

  Change one instruction and the agent's entire behavior changes.
  ```

- **No semicolons in presented cells:** Avoid semicolons in any cell presented on camera — split into two sentences instead. A semicolon almost always signals a sentence that should be broken.
  ```
  ❌ Before:
  `Runner.run()` returns a coroutine; await it to get the result.
  ✅ After:
  `Runner.run()` returns a coroutine.
  Await it to get the result.
  ```

- **Bullet list spacing:** Always add a blank line between bullet items in markdown cells. VS Code notebook rendering compresses bullet lists without blank lines, which looks cramped on camera. This applies to all bullet lists — Topics, Key Takeaways, and any other markdown cell with a list. Exception: Troubleshooting sections keep tight spacing (no blank lines between bullets) because they are dense reference content, not presented on camera.

- **Spacing before Topics header:** In the opening cell, add two `<br>` tags on separate lines between the intro paragraph and the **Topics:** header to create visual separation. The `<br>` tags must immediately follow the last line of text — no blank line before them, or VS Code renders them as literal text.

- **Key Takeaways formatting:** Each bold sub-header group in Key Takeaways gets a blank line before its bullets and two `<br>` tags after the last bullet in the group, before the next bold sub-header. The `<br>` tags must immediately follow the last bullet — no blank line before them.

---

## Notebook Length Management

Long notebooks often turn into long videos that weaken pacing and lose student attention. Each notebook should teach one coherent concept. Parts that do not introduce a new teaching beat should be cut, merged, or shortened.

**When reviewing, ask:**
- Does this Part introduce a new idea, or just repeat the previous one with a different example?
- Could this be a shorter demo instead of a full section?
- Is this explanation adding insight, or just restating what the code already shows?
- Is this abstraction necessary for teaching, or is it adding complexity without enough benefit?

**Signals that a notebook may be getting bloated:**
- Too many Parts for a single teaching goal
- Multiple Parts showing the same core concept with only small variations
- Explanatory cells that mostly restate visible code
- Supporting code or abstractions that add complexity without improving student understanding

**Core test:** If a Part can be removed and the notebook still teaches the intended concept clearly, it should usually be cut, merged, or condensed.

---

## Headers

**Step headers:** See the canonical setup rule in Notebook Structure.

**Part headers:** Use `## [emoji] Part N: [Title]` for major sections in longer notebooks.

**Sub-headers:** No trailing colons: `### 💡 Key Observation` (not `### 💡 Key Observation:`)

**Title headers with separators:** Colons OK: `### Challenge 1: Handoff Routing`

**Technique explanations:** Use `### 💡 [Short Descriptor]` for explaining the logic behind agent design decisions. The default descriptor is "Why This Works" — use this for demos that show something working. When the demo is intentionally showing failure (e.g., free-form text breaking downstream code), use a descriptor that matches the demo's intent: `### 💡 Why This Breaks`, `### 💡 Why This Fails`, or similar. The 💡 emoji is the consistency anchor; the descriptor matches what the demo actually shows.

Default to one clear sentence explaining the core insight. Bullets are acceptable when the insight has 3 or fewer parallel, discrete items that benefit from visual separation on camera — avoid bullets when the idea flows naturally as prose.

**When to cut the explanation cell:** Cut it when the insight is already visible in the code above and will be spoken on camera. Keep it only when the design decision is non-obvious or when naming the principle helps students apply it elsewhere.

---

## Code Cell Structure

Function definitions and test code must be separated with visual dividers.

**For cells with function + test:**

```python
def my_function(param):
    """One-line docstring."""
    return result

# --------------------------------------------------------------
# Test the function
# --------------------------------------------------------------
print("🧪 FUNCTION NAME TEST")
print("="*60)

result = my_function("test input")
print(f"Result: {result}")

print("="*60)
```

**For cells that only define functions (no test):**

```python
def my_function(param):
    """One-line docstring."""
    return result

# --------------------------------------------------------------
print("✅ my_function() ready")
```

**Test cases:** When a function definition is followed by separate test code cells, use `####` headers to introduce each test (e.g., `#### Test with Simple Query`). This provides visual separation between consecutive code cells.

**No consecutive code cells:** Every code cell must be followed by a markdown cell before the next code cell. A `###` header-only markdown cell counts as a valid separator. A markdown cell containing only a sub-header (like `#### tools.py`) also counts. Never place two code cells back-to-back with nothing between them.

**Sequential agent demo pattern:** When showing multiple agents running on the same input, use `#### Run [Agent Name]` headers as separators between code cells. Each code cell should define and run the agent together as one logical unit.

---

## Docstrings & Comments

**Docstrings:** One-line docstrings only for helper functions. Remove verbose Args/Returns blocks.

**Inline comments:** Remove verbose explanatory comments. Keep structural comments (section headers, TODOs). Keep brief clarifying comments only when logic is non-obvious.

---

## Code Style Constraints

**Readable names:** Use `message` / `messages` over `request` / `requests` unless the content is literally a support request. In demos, use `messages` (list) and `message` (loop variable).

**Loop variables:** `i` is fine for short loops and inside functions where the loop body is a few lines. Use readable names for top-level step loops where the body is substantial: `index`, `turn_number`, `message_index`, etc.
- Allowed in functions: `for i, sub_question in enumerate(sub_questions, 1)`
- Preferred at top level: `for index, message in enumerate(messages)`
- If `enumerate(..., 1)` is used for display, prefer `item_number` over `i`.

**Quote style:** Use straight ASCII quotes (`"` or `'`) in code cells. Avoid smart/curly quotes.

**Temperature:** Strictly excluded. Do not use the `temperature` parameter in any API call. Unsupported in gpt-5 series models.

**Trailing commas:** Do not use trailing commas in dictionaries or function calls.

**Classification labels:** Use lowercase for single-word labels (`positive`, `negative`, `neutral`, `low`, `medium`, `high`, `true`, `false`, `uncertain`).

**Variable names:** In multi-step agent pipelines (capstones), use distinct result variable names (`research_result`, `analysis_result`, `synthesis_result`) instead of reusing `result`.

**Truncation helper:** Use the canonical format: `f"...\n\n💡 (Truncated from {len(text)} chars)"`

---

## Import Pattern

These are default patterns. Notebook-specific imports should follow actual usage — add what you use, omit what you don't. Do not mechanically flag a notebook for including an import that the notebook actually uses, or for omitting one it doesn't need.

**Lesson 01 (environment check only — no agent):**

```python
import os
import sys
from pathlib import Path
from dotenv import load_dotenv
```

**All runnable agent notebooks (including Lesson 01 after the environment check):**

```python
from pathlib import Path
from dotenv import load_dotenv
from agents import Agent, Runner
```

- Only add `import os` when the notebook explicitly uses `os` (e.g., `os.getenv()`). Do not include it by default.
- `asyncio` is not required in notebook imports — JupyterLab supports top-level `await` without it. Only import `asyncio` when explicitly using `asyncio.gather()` or similar.
- When a notebook requires additional imports beyond the standard SDK set (e.g., `import asyncio`, `import random`, `from datetime import datetime`), group them below the SDK imports with a `# Notebook-specific imports` comment. This helps students quickly identify which imports are course-standard vs lesson-specific.
- Add additional imports (`function_tool`, `handoff`, etc.) only when introduced in that notebook.

---

## Kernel Metadata

All notebooks should use:

```json
"kernelspec": {
   "display_name": "openai-agents",
   "language": "python",
   "name": "openai-agents"
}
```

---

## Standard Helper Functions

Add `truncate_response` only when the notebook's agent demos can realistically produce long free-form text responses — for example, web search results, document summarization, or multi-step reasoning outputs. Do NOT add it by default. Do not add it for notebooks whose demos return short structured responses, structured output models, or other bounded outputs.

```python
def truncate_response(text, max_length=1200):
    """Truncate text for readability."""
    if len(text) <= max_length:
        return text
    return text[:max_length] + f"...\n\n💡 (Truncated from {len(text)} chars)"
```

---

## Environment Check Pattern (Canonical)

Lesson 01 uses this full diagnostic pattern. See the canonical setup rule in Notebook Structure for when this pattern applies vs. when to use the simpler `## 🔧 Setup` cell.

```python
import os
import sys
from pathlib import Path
from dotenv import load_dotenv

print("Checking environment...\n")

# Python version
py_ok = sys.version_info >= (3, 10)
py_version = sys.version.split()[0]
if py_ok:
    print(f"✅ Python {py_version}")
else:
    print(f"❌ Python {py_version} (3.10+ required)")

# openai-agents package
try:
    import agents
    agents_ok = True
    print(f"✅ openai-agents package: {agents.__version__}")
except ImportError:
    agents_ok = False
    print("❌ openai-agents package missing — run: pip install openai-agents")

# python-dotenv package
try:
    import dotenv
    dotenv_ok = True
    print(f"✅ python-dotenv package installed")
except ImportError:
    dotenv_ok = False
    print("❌ python-dotenv missing — run: pip install python-dotenv")

# API key
env_path = Path("..") / ".env"
load_dotenv(dotenv_path=env_path)
api_key = os.getenv("OPENAI_API_KEY")
key_ok = bool(api_key)
print(f"{'✅' if key_ok else '❌'} API key {'loaded' if key_ok else 'not found'}")

# Summary
print()
if py_ok and agents_ok and dotenv_ok and key_ok:
    print("✅ All checks passed!\n")
else:
    print("❌ Please fix issues above.")
    print("   Review 01_Environment_Check.ipynb\n")
```

---

## Async Handling Pattern

`Runner.run()` is a coroutine and must be awaited. JupyterLab supports `await` at the top level, so notebooks use `await` directly — no `asyncio.run()` wrapper needed.

Always place a blank line before and after `Runner.run()` calls. The Runner call is the main event in any agent cell — visual separation makes it stand out:

```python
agent = Agent(
    name="Assistant",
    instructions="You are a helpful assistant.",
    model=MODEL
)

result = await Runner.run(agent, input=message)

print(result.final_output)
```

---

## Agents SDK Core Patterns

**Teaching flow:** Show the minimal working agent first (raw Agent + Runner.run()), then build toward class abstractions and multi-tool patterns.

**Minimal agent:**

```python
from agents import Agent, Runner

agent = Agent(
    name="Assistant",
    instructions="You are a helpful assistant.",
    model=MODEL
)

result = await Runner.run(agent, input=message)
print(result.final_output)
```

**Model constants:**

```python
MODEL = "gpt-5-mini"           # Course default — all standard agents
REASONING_MODEL = "gpt-5"     # Orchestrators and complex reasoning tasks
```

Include `REASONING_MODEL` in the setup cell when:
- The notebook uses it in runnable code
- The notebook includes templates meant to model a reusable project structure that would use it
- The notebook explicitly teaches the MODEL vs REASONING_MODEL decision (Lesson 03 introduces the two constants; Lesson 31 covers the full framework)

Remove `REASONING_MODEL` from the setup cell when it is unused and the notebook has no teaching purpose that requires naming it.

**Tool definition (decorator pattern):**

```python
from agents import function_tool

@function_tool
def get_weather(location: str) -> str:
    """Get current weather for a location."""
    # implementation
    return f"Weather in {location}: 72°F and sunny"
```

**Instructions length rule:**
- 1 line — pass inline as a string directly in the Agent constructor
- 2+ lines — assign to a variable first, then pass `instructions=instructions`

```python
# 1 line — inline is fine
agent = Agent(
    name="WeatherAgent",
    instructions="Help users with weather questions.",
    model=MODEL,
    tools=[get_weather]
)

# 2+ lines — assign first to avoid triple-quote indentation inconsistency on camera
instructions = (
    "Help users with weather questions.\n"
    "Always use the get_weather tool for current conditions.\n"
    "Be concise — one paragraph maximum."
)

agent = Agent(
    name="WeatherAgent",
    instructions=instructions,
    model=MODEL,
    tools=[get_weather]
)
```

**Instructions variable naming:** When a single agent is defined in a notebook, use the bare `instructions` variable name. When multiple agents are defined in the same notebook, use descriptive names to avoid collisions: `judge_instructions`, `v2_instructions`, `triage_instructions`, etc.

**Agent with tools:**

```python
agent = Agent(
    name="WeatherAgent",
    instructions="Help users with weather questions.",
    model=MODEL,
    tools=[get_weather]
)
```

**Handoffs:**

```python
from agents import Agent, handoff

specialist = Agent(name="Specialist", instructions="...", model=MODEL)

triage = Agent(
    name="Triage",
    instructions="Route to the right specialist.",
    model=MODEL,
    handoffs=[specialist]
)
```

**Result access:**

```python
result = await Runner.run(agent, input=message)
result.final_output          # Final string response
result.final_output.strip()  # Cleaned response
```

---

## Pydantic Models for Structured Output

Use `output_type=` with a Pydantic `BaseModel` subclass for structured agent output.

Use bounded fields where appropriate — constraints belong in the model, not in the prompt:
- Numeric scores: `Annotated[int, Field(ge=1, le=5)]`
- String length: `Field(max_length=200)`

Import pattern: `from pydantic import BaseModel, Field` and `from typing import Annotated`

Remove inline comments from Pydantic field definitions — field names should be self-documenting. Exception: in capstone notebooks, brief inline comments that enumerate valid values (e.g., `# e.g. "bug", "feature", "question"`) are acceptable when they aid student understanding of expected inputs.

---

## Variable Naming

- `agent` = an Agent instance
- `result` = the object returned by `Runner.run()` — type is `RunResult`
- `output` = `result.final_output` when stored separately
- `message` = a single string input to the agent
- **Multi-step exception:** In capstone pipelines, use specific names: `research_result`, `analysis_result`, `synthesis_result`

---

## Model References

**In code:** Use `MODEL` or `REASONING_MODEL` constants — never hardcode model strings outside the constants block in runnable notebook code.

**In prose:**
- **Week 1:** Explicitly name `gpt-5-mini` and `gpt-5` when introducing the two constants and explaining when to use each.
- **Other weeks:** Prefer generic terms ("the model", "the reasoning model") so concepts age well.

---

## Content Guidelines

**Capstone terminology:** For capstone notebooks (Lessons 13, 18, 26, 29), use `## [emoji] Phase N: [Title]` headers for pipeline steps. Phase is the default — use Component or Challenge only when the structure genuinely doesn't fit a sequential pipeline. Reserve `Step 1` / `Step 2` for setup cells only.

**Capstone identities:** Each capstone has a specific pedagogical identity that must be preserved:
- Capstone 1 (Lesson 13, Research Agent): Build a useful single-agent tool
- Capstone 2 (Lesson 18, Research Team): Coordinate multiple agents
- Capstone 3 (Lesson 26, Customer Service): Ship a reliable production agent — owns the safety theme
- Capstone 4 (Lesson 29, MCP Assistant): Extend the agent beyond its local environment — headline is external integration, not safety

**Capstone internal dividers:** Inside multi-phase functions, use section dividers with a comment label to separate phases clearly:

```python
# -------------------------------------------------------
# Phase 1: Decompose question
# -------------------------------------------------------
```

**Model constants section:** A dedicated Model Constants section is not required if the setup cell comments already introduce `MODEL` and `REASONING_MODEL`. Omit the section to avoid redundancy — the constants are self-documenting in the setup cell.

**Neutral topics:** Use neutral topics for demos and exercises (weather, product lookups, scheduling). Avoid politically charged or controversial topics.

**Cleanup cell placement (canonical):** There are two types of cleanup, and they go in different places:

- **Demo cleanup** (removing temporary files or resetting demo state) — place before Practice Exercises so the workspace is clear before students start their work
- **API resource cleanup** (deleting vector stores, uploaded files, or any API objects that incur ongoing cost) — place after Troubleshooting so students can complete Practice Exercises before teardown

A markdown divider must separate any Cleanup code cell from whatever follows it.

**Tracing cells:** Teach tracing at the concept level. Tell students that runs are automatically recorded and can be inspected to see input, instructions, and output together. Avoid step-by-step UI navigation instructions — interface locations change, create support noise, and distract from the lesson. One or two concept-level sentences is the right weight for any notebook that is not dedicated to tracing (Lesson 25 is the exception).

---

## Security Notes

Security notes should appear the **first time** a given risk pattern appears in the course. In subsequent notebooks, omit the security note — students have already seen it. The dedicated security lesson is the full treatment.

- **Web fetch tools:** The first notebook where any tool or MCP server retrieves content from external URLs — fetched content is untrusted input and can contain adversarial text designed to override agent instructions
- **Filesystem write or delete access:** The first notebook where any tool or MCP server gives the agent the ability to create, modify, move, or delete files

**Exceptions where a security note should reappear:**
- The dedicated security and safety lesson — full treatment, security notes throughout
- Any lesson where a new and distinct security risk appears that hasn't been covered before

Additionally, reviewers should strongly consider a security note when user-supplied input is passed into agent instructions or tool parameters without clear constraints or validation — but only on the first notebook where this pattern appears.

**Security note format:** A standalone markdown cell with a ⚠️ prefix, placed immediately after the relevant demo cell or its adjacent Why This Works cell:

```
⚠️ **Security note:** [One sentence describing the risk and how to treat the input.]
```

The note should:
- Name the specific risk (prompt injection, filesystem write access, etc.)
- Reference the relevant earlier notebook where the risk was first taught, when applicable
- Be one to two sentences — a reminder, not a lecture

---

## UI History vs Agent Memory

When a notebook introduces a UI framework (Gradio, Streamlit, or similar) that displays a visible conversation history, the notebook must explicitly distinguish between UI state and agent memory.

- UI history (the visible chat log) is managed by the framework and does not reach the agent unless the notebook explicitly passes it into the agent input or reconstructs it into the prompt
- The agent is stateless across turns unless sessions or persistent memory are used, or unless history is manually reconstructed into the input prompt
- A notebook that shows a chat UI without addressing this distinction can mis-teach students that "the UI has history" means "the agent remembers" — this is a correctness issue, not just a style issue

The fix is a one-sentence warning near the first UI demo and a reinforcing bullet in Key Takeaways.

---

## Troubleshooting Sections

All notebooks use a single link to TROUBLESHOOTING.md. Lesson-specific content is maintained there.

---

## Presentation Optimization

- Split multi-section markdown cells for pacing.
- Cut explanatory text that will be spoken on camera.
- **Dollar signs in markdown:** Jupyter interprets text between two `$` signs on the same line as LaTeX math, which breaks rendering. When a cell contains multiple dollar amounts, put each on its own line to prevent this. Do not use backslash escaping (`\$`) — it does not fix the issue and may cause its own rendering problems.

(Benefits/Problems bullet-limit rule lives in Markdown Pacing Rules — canonical home.)

---

## Export / Rendering Formatting

These rules address how notebooks render in exports and downstream platforms, not teaching quality. Treat them as low-priority review items — they don't affect whether a notebook teaches well.

- **Markdown tables:** Wrap all markdown tables in a left-justify div for consistent rendering:

```html
<div style="text-align: left; display: inline-block;"> ... </div>
```

---

## Emojis

- **Keep:** Section headers (e.g. `## 🎯 Key Takeaways`, `## 📍 Next Step`), semantic emojis in code output (✅/❌, 🔬, 🤝, 🛠️).
- **Keep:** The `## 🎯 Key Takeaways` section header emoji.
- **Remove:** Emojis from bold sub-headers inside Key Takeaways — use plain bold text.
  - ✅ Correct: `**Sessions add context across turns:**`
  - ❌ Incorrect: `**💾 Sessions add context across turns:**`
  - This distinction applies only to Key Takeaways sub-headers. Other section headers throughout the notebook may use emoji as usual.
- **Remove:** Decorative emojis on plain bullet lists.
- **Remove:** Trailing emojis at end of sentences/sections.

---

## Tightening Prose — Before/After Examples

When tightening a cell, the goal is: one idea per sentence, cut restatements, end on the sharpest point. If the last sentence restates what the first two already said, cut it.

**Example 1 — Cut restatements, end sharp:**

Before:
```
When an agent has tools, it doesn't always use them. The model decides based on
your prompt. If the answer doesn't need a tool, it answers from training data.
If it does, the model picks the tool that fits. Your prompt and instructions
shape the choice. Specific prompts steer the agent toward specific tools. Vague
prompts lead to vague tool use or none at all.
```

After:
```
When an agent has tools, it doesn't always use them.

The model decides per prompt — specific questions trigger tools, vague ones don't.

**Your prompt and instructions shape the choice.**
```

---

**Example 2 — Cut the trailing clause that restates the point:**

Before:
```
The model never gets to override the decision — it's Python enforcing, not a prompt suggestion.
```

After:
```
The model can't override it — it's Python, not a prompt suggestion.
```

*Pattern:* "never gets to [verb]" → "can't [verb]". Trailing restatement ("it's Python enforcing") → cut or compress. End on the contrast, not the elaboration.

---

**Example 3 — Merge two consecutive sentences that express one idea:**

Before:
```
The second shape is almost always better. Design tools so the agent can safely retry them.
```

After:
```
The second shape is almost always better — design tools the agent can safely retry.
```

*Pattern:* When sentence B is the direct implication of sentence A, join them with an em dash. Two sentences signal two ideas; one idea gets one sentence.

---

**Example 4 — Cut meta-commentary labels; break remaining ideas into camera lines:**

Before:
```
When you add a new tool and mark it as requiring human approval, the guardrail
automatically catches it — no guardrail rewrite needed. That's the win: the
policy stays stable while your tool inventory grows.
```

After:
```
Mark a tool as requiring approval and the guardrail catches it — no rewrite needed.

The policy stays stable as your tool inventory grows.
```

*Pattern:* "That's the win:", "That's the point:", "This is why it matters:" — cut the label and show the win directly. Break the remaining ideas into separate lines, one per camera beat.

---

**Example 5 — Cut throat-clearing openers:**

Before:
```
Simple setup. The user tells the agent: "Summarize the doc at malicious_doc.md."
The doc contains what looks like product strategy, but halfway through there's
an injection payload telling the agent to ignore its instructions.
```

After:
```
The user tells the agent to summarize malicious_doc.md. The doc looks like
product strategy, but halfway through there's an injection payload telling
the agent to ignore its instructions.
```

*Pattern:* "Simple setup.", "What's happening here:", "The pattern generalizes:", and similar throat-clearing openers — cut them. The content that follows does the work.

---

**Example 6 — Let the list do the work; cut the windup:**

Before:
```
The attack surface is enormous: every file, every web page, every tool
result is a potential injection vector.
```

After:
```
Every file, every web page, every tool result is a potential injection vector.
```

*Pattern:* "X is enormous: [the thing that makes it enormous]" → just say the thing. The label ("enormous") becomes redundant once the list lands.

---

**Example 7 — Compress the negative form:**

Before:
```
The output type returns a structured object — it doesn't block bad output, it just validates it.
```

After:
```
The output type returns a structured object — it only validates.
```

*Pattern:* "doesn't X, it just Y" → "only Y". The negation adds a beat without adding meaning.

---

**Example 8 — Cut redundant restatement across consecutive lines:**

Before:
```
The model does not inspect the Python function body. It sees the tool name, description, and schema.

The model picks the tool that best matches the question — or calls none.

The decision is based on name, description, and schema.

When the model picks wrong, rewrite the tool — not the agent logic.
```

After:
```
The model does not inspect the Python function body — it sees only the name, description, and schema.

The description drives tool choice.

When the model picks wrong, rewrite the description, not the agent logic.
```

*Pattern:* Two sentences that express one idea get merged with an em dash. Consecutive lines that restate the same fact get cut to one. Each remaining sentence gets its own line with a blank line between — one beat per line on camera.

---

**Example 9 — Break a dense Why This Works paragraph into camera-ready lines:**

Before:
```
`WebSearchTool()` is a built-in hosted tool — OpenAI runs the search on their side, no third-party API required. This is called **grounding**: the agent's answer is anchored to retrieved content rather than generated from training data alone. Grounded responses are more accurate for time-sensitive questions and less likely to confabulate facts.
```

After:
```
`WebSearchTool()` is a built-in tool — OpenAI runs the search, no API key needed.

This **grounds** the agent's answer in real retrieved content instead of training data.

The result: better accuracy on time-sensitive questions and less hallucination.
```

*Pattern:* A dense multi-sentence paragraph in a Why This Works cell becomes one sentence per idea, each on its own line with a blank line between. Cut technical jargon ("confabulate", "anchored to retrieved content") in favor of plain language. The last line lands the student benefit. This applies to all explanatory prose cells — opening cells, Part intro cells, Why This Works cells — anywhere prose will be read on camera.

---

**Example 10 — Strip a decision-guide cell to camera-ready bullets:**

Before:
```
## 🧭 Part 4: Picking the Right Approach

You now have three layers: modes, rules, callbacks. Here's how to pick:

- **Permission mode only** — when you want a broad behavioral default. "This agent is a prototyping helper, auto-approve edits." → `acceptEdits`.

- **Mode + `allowed_tools`/`disallowed_tools`** — when you need a fixed tool surface. "This CI agent uses Read, Grep, and Bash, nothing else." → `default` + narrow `allowed_tools`.

- **Mode + rules + `can_use_tool`** — when you need content-aware decisions. "Allow Bash but block destructive commands; rewrite Write calls to a sandbox directory." → `default` + callback. (Don't add Bash to `allowed_tools` — that would bypass the callback.)

A few quick guidelines:

- **Start with `default`** — the extra prompts teach you what the agent actually reaches for. You can loosen later.

- **`bypassPermissions` + `disallowed_tools`** is the right pattern for trusted environments that need specific hard blocks. Never use `bypassPermissions` alone outside a sandbox.
```

After:
```
## 🧭 Part 4: Picking the Right Approach

Three layers, three questions:

- **Mode only** — what's the broad default? Use `acceptEdits` for edit-heavy agents, `default` for everything else.

- **Mode + rules** — is the tool surface fixed? Use `allowed_tools`/`disallowed_tools` to lock it down.

- **Mode + rules + callback** — do decisions depend on content? Use `can_use_tool` to inspect and rewrite arguments at runtime.
```

*Pattern:* Decision-guide cells with quoted examples and trailing guidelines sections are reference docs disguised as narrated content. Strip each bullet to one line: the condition and the answer. Cut quoted examples — the demos already showed them. Cut the guidelines section entirely if KT already covers the same points. Three bullets max; each must fit on one camera line.

---

**Example 11 — Cut a Why This Works cell to its core insight:**

Before:
```
### 💡 Why This Works

When the callback returns `PermissionResultDeny`, the tool call never executes.

Claude sees the denial message and adjusts — typically it explains the command was blocked and asks what to do instead.

This is deterministic safety: your check runs before the shell does, regardless of how the agent frames the command.

Substring matching is illustrative — real-world attackers can evade it; Hooks in Lesson 22 are the stronger production pattern.
```

After:
```
### 💡 Why This Works

When the callback returns `PermissionResultDeny`, the tool call never executes.

Claude sees the denial message and adjusts — usually explaining the block and asking what to do next.
```

*Pattern:* Why This Works cells default to one clear sentence — two at most. Cut lines that restate the demo result ("your check runs before the shell" is implied by "the tool call never executes"). Cut forward references and caveats ("Hooks in Lesson 22 are stronger") — they belong in KT or a security note, not in the Why This Works explanation. End on what the student needs to carry forward.

---

**Example 12 — Cut lines the demo code already shows:**

Before:
```
## 🎛️ Part 3: The `can_use_tool` Callback

Modes and rules are static — set when you build `ClaudeAgentOptions`.

For runtime decisions, you need the `can_use_tool` callback.

The callback fires for tool calls not already resolved by the mode or deny rules.

It receives the tool name and Claude's intended input, and returns one of:

- `PermissionResultAllow()` — approve as-is

- `PermissionResultAllow(updated_input=...)` — approve with rewritten arguments

- `PermissionResultDeny(message=...)` — block with a reason Claude sees
```

After:
```
## 🎛️ Part 3: The `can_use_tool` Callback

Modes and rules are static — set when you build `ClaudeAgentOptions`.

For runtime decisions, you need the `can_use_tool` callback.

It returns one of three decisions:

- `PermissionResultAllow()` — approve as-is

- `PermissionResultAllow(updated_input=...)` — approve with rewritten arguments

- `PermissionResultDeny(message=...)` — block with a reason Claude sees
```

*Pattern:* Cut prose lines that describe what the code already shows. "The callback fires for tool calls not already resolved" is visible in the evaluation order — don't narrate the plumbing. "It receives the tool name and Claude's intended input" is visible in the function signature — don't narrate the signature. Keep only what the code can't show: why this design choice exists and what the student should take away.

---

## Version Progression Pattern

For complex class-based structures: Provide the clean final version with a markdown explanation of changes. Diff-style comments make class code unreadable — avoid them.
