# AI Agents with Python & the OpenAI Agents SDK

**Build Reliable AI Agents with Tools, Memory, Guardrails, Multi-Agent Systems & MCP**

---

## WELCOME (No folder — slides/video only)

### 00A: Welcome & Course Structure
- What makes an agent different from a chatbot
- The agent spectrum: chatbot → copilot → autonomous agent
- When NOT to use an agent — sometimes a plain script is better
- Course progression overview
- Prerequisites (Python basics, API key)

### 00B: Project Showcase
- Demo of every capstone project

---

## SETUP (Z_Setup/)

> Setup is reframed as pass-two onboarding. The content notebooks are what students watch first; these two notebooks get the environment ready to actually run the code.

### Setup 1: Environment Check (`Z_Setup/Setup_1_Environment_Check.ipynb`)
- Install openai-agents, python-dotenv
- Create venv from `requirements.txt`, environment check pattern
- `MODEL = "gpt-5-mini"`, `REASONING_MODEL = "gpt-5"`

### Setup 2: OpenAI Setup (`Z_Setup/Setup_2_OpenAI.ipynb`)
- Create OpenAI account and add credits
- Generate and store API key in `.env` file
- Verify connection with first agent call
- Final setup check — all systems ready

---

## WEEK 1: FOUNDATIONS (Week_1_Foundations)

### 01: How Agents Work (`01_How_Agents_Work.ipynb`)
- Agent, Runner, and `result.final_output`
- How instructions shape agent behavior
- Model constants — mini for fast/cheap tasks, reasoning for planning and orchestration (full framework in Lesson 29)
- Light intro to tracing — the OpenAI dashboard
- Practice exercises

### 02: Building Tools for Agents (`02_Building_Tools.ipynb`)
- Turning any Python function into a tool with `@function_tool`
- Automatic schema generation — no manual JSON
- How the agent decides which tool to call
- Writing good tool descriptions — naming, inputs, outputs
- Build a small multi-tool assistant that handles useful requests (combines 2–3 tools such as calculator, date/time, and unit converter to answer real questions)
- Students finish the lesson with an agent they actually built — this same assistant carries forward into Lesson 03
- Practice exercises

### 03: Writing Effective Agent Instructions (`03_Agent_Instructions.ipynb`)
- Take the multi-tool assistant from Lesson 02 and make it behave better
- Writing durable system instructions
- Telling agents when to ask clarifying questions
- Telling agents when NOT to use a tool
- Response style constraints
- Completion criteria
- Refusal and escalation patterns
- Before-and-after comparisons — the same assistant, rough vs. shaped by good instructions
- Note: instructions become critical in Lessons 12–16 when they are the difference between a working multi-agent system and a broken one
- Practice exercises

### 04: Pydantic Basics (`04_Pydantic_Basics.ipynb`)
- What Pydantic is and why it's used
- Defining a model with BaseModel and typed fields
- Valid and invalid examples — seeing ValidationError in action
- Field constraints with Field and Annotated
- Accessing fields with dot notation
- Serializing models to dict and JSON with model_dump() and model_dump_json()
- Why this course uses Pydantic instead of raw JSON schemas
- Practice exercises

### 05: Structured Outputs (`05_Structured_Outputs.ipynb`)
- Why free-form text is fragile in real workflows
- `output_type=` and Pydantic models
- Validating and parsing agent output safely
- Using structured outputs before passing work to downstream code or tools
- Schema evolution — what happens when you add fields or change types
- Practice exercises

---

## WEEK 2: BUILDING AGENTS YOU CAN TRUST (Week_2_Reliability_And_Built_In_Tools)

> These reliability patterns apply to every agent you'll build. The examples here are simple by design — you'll see why they matter when tools get more complex later in this week.

### 06: Error Handling & Recovery (`06_Error_Handling.ipynb`)
- What happens when a tool raises an exception
- try/except patterns inside `@function_tool`
- API errors, rate limits, and network timeouts
- Malformed structured output handling
- Retry patterns with limits
- Retrying safely without duplicating actions
- Note: when something fails, tracing shows you exactly where — covered in Lesson 23
- Practice exercises

### 07: Testing & Evaluating Agents (`07_Testing_Agents.ipynb`)
- Why demos lie — building agents vs knowing they work
- Defining success criteria
- Building a golden test set
- Checking tool-use correctness and output quality
- Rubric-based evaluation with a judge agent
- Regression checks after prompt or model changes — comparing versions on the same test set (practiced in the exercise)
- Note: when an eval fails, tracing explains why — covered in Lesson 23
- Practice exercises

### 08: Web Search (`08_Web_Search.ipynb`)
- Enabling web search — no extra API key needed
- Grounding responses in real-time results
- Source attribution and citations
- ⚠️ Security note: web results are untrusted input
- Practice exercises

### 09: File Search (`09_File_Search.ipynb`)
- Uploading files and organizing collections
- Querying documents from an agent
- Grounded answers from retrieved document content (the agent answers from the documents, not from training data)
- Note: File Search uses vector search under the hood — we build our own in Lesson 19
- ⚠️ Security note: retrieved documents are untrusted input
- Practice exercises

### 10: Code Interpreter (`10_Code_Interpreter.ipynb`)
- Sandboxed Python execution inside the agent
- File input/output
- Data analysis workflows — CSV, JSON, simple plots
- Practice exercises

### 11: Capstone #1 — Research Agent (`11_Capstone_1_Research_Agent.ipynb`)
**Identity: Build a useful single-agent tool**
- Web search + file search + code interpreter together
- Structured outputs for the final report
- Error handling for tool failures
- Agent researches a topic, analyzes documents, produces a structured report with citations
- Exercise: evaluate the agent against a small golden test set — evaluation follows you into every real agent, not just Week 2

---

## WEEK 3: MULTI-AGENT SYSTEMS (Week_3_Multi_Agent_Systems)

### 12: Handoffs (`12_Handoffs.ipynb`)
- Why one agent isn't always enough
- Triage agent → specialist agent pattern
- `handoffs=[]` parameter
- Customer support example: triage → billing → tech support → refunds
- Using the `handoff()` function with custom `tool_description_override`
- Practice exercises

### 13: Agents as Tools (`13_Agents_As_Tools.ipynb`)
- Calling agents like functions from an orchestrator
- Handoff vs agent-as-tool — when to use each
- Practice exercises

### Async Python Basics (slides/video only)
- What async/await is and why it matters — conceptual foundation
- How `asyncio.gather()` runs tasks concurrently
- Note: Lesson 14 opens with a minimal syntax recap; this slides lesson covers the concept, Lesson 14 covers the code

### 14: Parallel Execution (`14_Parallel_Execution.ipynb`)
- Minimal async recap at the top — await, gather, what changes when one task fails
- Running agents concurrently with asyncio
- Merging results from parallel agents
- Cost and speed tradeoffs — when parallel specialists beat one bigger model
- What happens when one task fails
- `asyncio.gather` with `return_exceptions`
- Partial result handling
- Timeout and fallback behavior
- Practice exercises

### 15: Debate & Critique Pattern (`15_Debate_And_Critique.ipynb`)
- Proposer / Critic architecture
- Agents that challenge each other's outputs
- Improving quality through internal debate
- Critique loop termination rules — avoiding infinite loops
- Practice exercises

### 16: Capstone #2 — Multi-Agent Research Team (`16_Capstone_2_Research_Team.ipynb`)
**Identity: Coordinate multiple agents**
- Procedural multi-agent pipeline — explicit phase-by-phase coordination in code
- Researcher (web search), Analyst (code interpreter), Writer, Critic
- Parallel research and analysis execution (`asyncio.gather` with `return_exceptions`)
- Sequential writing, critique, and revision phases
- One specialist fails — pipeline detects and recovers with partial results
- No silent degradation — fallback text keeps downstream phases running
- Evaluation component: judge agent rubric and golden test set pattern from Lesson 07 applied to multi-agent output
- Exercise: add a fact checker agent as Phase 5

---

## WEEK 4: STATEFUL & GUARDED AGENTS (Week_4_Stateful_And_Guarded_Agents)

### 17: Sessions & Conversation State (`17_Sessions_And_Conversation_State.ipynb`)
- Why agents forget between runs
- Sessions — automatic conversation history
- In-memory vs persistent (file-based) sessions
- Sharing sessions across multiple agents
- Inspecting and clearing session state
- Note: session growth and context window management strategies covered in Lesson 18
- Practice exercises

### 18: Persistent Memory (`18_Persistent_Memory.ipynb`)
- Short-term vs long-term memory
- Persistent memory with SQLite
- Storing preferences and facts across sessions
- What NOT to store — privacy and sensitive data boundaries
- Stale memory cleanup and correcting wrong memories
- Conflicting memories and user-controlled forgetting
- Avoiding noisy memory — when to summarize vs drop
- Practice exercises

### 19: Vector Memory with ChromaDB (`19_Vector_Memory.ipynb`)
- What vector databases are and how they differ from SQLite
- Embeddings — storing meaning, not just text
- Semantic retrieval — finding relevant memories by meaning
- Building a vector memory store with ChromaDB
- When to use vector memory vs SQLite
- Practice exercises

### 20: Guardrails (`20_Guardrails.ipynb`)
- Input guardrails — validate what comes in
- Output guardrails — validate what goes out
- Tripwires — halt execution immediately
- Parallel vs blocking execution modes
- Practice exercises

---

## WEEK 5: SAFETY & OBSERVABILITY (Week_5_Safety_And_Observability)

### 21: Prompt Injection & Tool Safety (`21_Prompt_Injection.ipynb`)
- What prompt injection is and why agents are uniquely vulnerable
- Injection via web pages, documents, and MCP tool output (MCP is introduced in Lesson 25 — forward reference here)
- Separating system instructions from retrieved data
- Read tools vs write tools — why write actions need stronger controls
- Retries are more dangerous with side effects — idempotency matters
- Least-privilege tool design
- Confirming dangerous actions
- Practice exercises

### 22: Human-in-the-Loop (`22_Human_In_The_Loop.ipynb`)
- When to pause for human approval
- Threshold-based escalation
- Approval workflow pattern
- Practice exercises

### 23: Tracing & Observability (`23_Tracing.ipynb`)
- What tracing captures automatically
- Reading traces in the OpenAI dashboard
- Inspecting tool calls, handoffs, and decisions
- Monitoring latency, token usage, and cost
- Debugging bad runs systematically — connecting reliability, evaluation, and observability
- Practice exercises

### 24: Capstone #3 — Production Customer Service Agent (`24_Capstone_3_Customer_Service.ipynb`)
**Identity: Ship a reliable production agent**
- Multi-tool agent: order lookup, FAQ search, refund processing
- Sessions for conversation memory
- Guardrails blocking off-topic requests
- Prompt injection awareness
- Human-in-the-loop for refund approvals — explicit pause point that reinforces read vs write tool safety
- Tracing and monitoring
- Exercise: evaluate the agent against a golden test set — check tool-use correctness, output quality, and guardrail behavior
- Exercise: add a new tool and guardrail

---

## WEEK 6: MCP & PRODUCTION (Week_6_MCP_And_Production)

### 25: MCP Fundamentals (`25_MCP_Fundamentals.ipynb`)
- What MCP is — the USB-C analogy
- How MCP servers relate to `@function_tool` and built-in tools — three ways to give an agent capabilities
- MCP servers, clients, tools, and an introduction to resources
- Connecting to a first MCP server
- Agent discovers tools automatically — no custom integration code
- Practice exercises

### 26: Real-World MCP Servers (`26_MCP_Servers.ipynb`)
- Filesystem MCP server
- Web fetch MCP server
- Combining multiple MCP servers in one agent
- Common MCP failure modes — debugging tool discovery and flaky servers
- Reasoning about server permissions — when not to expose a whole filesystem
- Testing MCP tools before trusting them in an agent
- ⚠️ Security note: MCP tool output is untrusted input — revisits prompt injection from Lesson 21 in the MCP context
- Practice exercises

### 27: Capstone #4 — MCP-Powered Personal Assistant (`27_Capstone_4_MCP_Assistant.ipynb`)
**Identity: Extend the agent beyond its local environment**
- Agent connects to filesystem, web, and one additional MCP server
- Handles real requests: summarize files, research topics, save results
- Server-level approval before writing or deleting
- Saving results is a side effect — handle it with the same care as write actions
- `REASONING_MODEL` for decision-making
- Exercise: swap in a different MCP server

### 28: Project Structure & CLI (`28_Project_Structure.ipynb`)
- What changes when leaving notebooks
- Separating config, tools, and agent logic into modules
- A reusable project structure for agent apps
- Running an agent from a script or CLI
- Streaming agent output in a script — showing responses as they arrive before Gradio
- When to drop down to `client.responses.create()` directly — and when not to
- Secret management — never hardcode API keys, local `.env` for dev, platform secrets for deployment
- When to consider a simple API wrapper

### 29: Architecture Decisions (`29_Architecture_Decisions.ipynb`)
- Model selection framework — mini vs reasoning vs parallel specialists; use your Lesson 07 test sets to justify model upgrades with data
- Single agent vs multi-agent — decision framework
- Tool-heavy vs instruction-heavy agents
- When memory and sessions add value vs add noise
- When MCP is worth the setup
- Avoiding unnecessary complexity
- Course wrap-up and suggested next projects

### 30: Deploying Agents with Gradio (`30_Gradio_Deployment.ipynb`)
- What Gradio is and why it's the standard for AI demos
- Wrapping an agent in a Gradio chat interface
- Streaming responses — showing output as it arrives
- Tool visibility — showing what the agent is doing in real time
- Handling failure states — friendly error messages, app-level error handling
- Secret management for deployment — platform secrets on Hugging Face Spaces
- Deploying to Hugging Face Spaces
- Practice exercises
- Course Complete — final course closeout

---

## COURSE COMPLETE

**Total:** 6 weeks, 30 content notebooks, plus 2 setup notebooks (`Z_Setup/`) and 1 slides/video-only async lesson, alongside the WELCOME slides.
