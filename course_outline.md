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

## WEEK 1: FOUNDATIONS (Week_1_Foundations)

### 01: Environment Check (`01_Environment_Check.ipynb`)
- Install openai-agents, python-dotenv
- Create conda environment, environment check pattern
- `MODEL = "gpt-5-mini"`, `REASONING_MODEL = "gpt-5"`

### 02: OpenAI Setup (`02_OpenAI_Setup.ipynb`)
- Create OpenAI account and add credits
- Generate and store API key in `.env` file
- Verify connection with first agent call
- Final setup check — all systems ready

### 03: How Agents Work (`03_How_Agents_Work.ipynb`)
- Agent, Runner, and `result.final_output`
- How instructions shape agent behavior
- Model constants — mini for fast/cheap tasks, reasoning for planning and orchestration (full framework in Lesson 31)
- Light intro to tracing — the OpenAI dashboard
- Practice exercises

### 04: Building Tools for Agents (`04_Building_Tools_For_Agents.ipynb`)
- Turning any Python function into a tool with `@function_tool`
- Automatic schema generation — no manual JSON
- How the agent decides which tool to call
- Writing good tool descriptions — naming, inputs, outputs
- Build a small multi-tool assistant that handles useful requests (combines 2–3 tools such as calculator, date/time, and unit converter to answer real questions)
- Students finish the lesson with an agent they actually built — this same assistant carries forward into Lesson 05
- Practice exercises

### 05: Writing Effective Agent Instructions (`05_Writing_Agent_Instructions.ipynb`)
- Take the multi-tool assistant from Lesson 04 and make it behave better
- Writing durable system instructions
- Telling agents when to ask clarifying questions
- Telling agents when NOT to use a tool
- Response style constraints
- Completion criteria
- Refusal and escalation patterns
- Before-and-after comparisons — the same assistant, rough vs. shaped by good instructions
- Note: instructions become critical in Lessons 14–18 when they are the difference between a working multi-agent system and a broken one
- Practice exercises

### 06: Pydantic Basics (`06_Pydantic_Basics.ipynb`)
- What Pydantic is and why it's used
- Defining a model with BaseModel and typed fields
- Valid and invalid examples — seeing ValidationError in action
- Field constraints with Field and Annotated
- Accessing fields with dot notation
- Serializing models to dict and JSON with model_dump() and model_dump_json()
- Why this course uses Pydantic instead of raw JSON schemas
- Practice exercises

### 07: Structured Outputs (`07_Structured_Outputs.ipynb`)
- Why free-form text is fragile in real workflows
- `output_type=` and Pydantic models
- Validating and parsing agent output safely
- Using structured outputs before passing work to downstream code or tools
- Schema evolution — what happens when you add fields or change types
- Practice exercises

---

## WEEK 2: BUILDING AGENTS YOU CAN TRUST (Week_2_Reliability_And_Built_In_Tools)

> These reliability patterns apply to every agent you'll build. The examples here are simple by design — you'll see why they matter when tools get more complex later in this week.

### 08: Error Handling & Recovery (`08_Error_Handling_And_Recovery.ipynb`)
- What happens when a tool raises an exception
- try/except patterns inside `@function_tool`
- API errors, rate limits, and network timeouts
- Malformed structured output handling
- Retry patterns with limits
- Fallback strategies — alternative tool, graceful failure, ask for clarification
- Retrying safely without duplicating actions
- Note: when something fails, tracing shows you exactly where — covered in Lesson 25
- Practice exercises

### 09: Testing & Evaluating Agents (`09_Testing_And_Evaluating_Agents.ipynb`)
- Why demos lie — building agents vs knowing they work
- Defining success criteria
- Building a golden test set — saving and versioning over time
- Checking tool-use correctness and output quality
- Rubric-based evaluation with a judge agent
- Comparing two agent or prompt versions
- Regression checks after model swaps, prompt changes, or schema updates
- Note: when an eval fails, tracing explains why — covered in Lesson 25
- Practice exercises

### 10: Web Search (`10_Web_Search.ipynb`)
- Enabling web search — no extra API key needed
- Grounding responses in real-time results
- Source attribution and citations
- ⚠️ Security note: web results are untrusted input
- Practice exercises

### 11: File Search (`11_File_Search.ipynb`)
- Uploading files and organizing collections
- Querying documents from an agent
- Grounded answers from retrieved document content (the agent answers from the documents, not from training data)
- Note: File Search uses vector search under the hood — we build our own in Lesson 21
- ⚠️ Security note: retrieved documents are untrusted input
- Practice exercises

### 12: Code Interpreter (`12_Code_Interpreter.ipynb`)
- Sandboxed Python execution inside the agent
- File input/output
- Data analysis workflows — CSV, JSON, simple plots
- Practice exercises

### 13: Capstone #1 — Research Agent (`13_Capstone_1_Research_Agent.ipynb`)
- Web search + file search + code interpreter together
- Structured outputs for the final report
- Error handling for tool failures
- Agent researches a topic, analyzes documents, produces a structured report with citations
- Exercise: evaluate the agent against a small golden test set — evaluation follows you into every real agent, not just Week 2

---

## WEEK 3: MULTI-AGENT SYSTEMS (Week_3_Multi_Agent_Systems)

### 14: Handoffs (`14_Handoffs.ipynb`)
- Why one agent isn't always enough
- Triage agent → specialist agent pattern
- `handoffs=[]` parameter
- Customer support example: triage → billing → tech support → refunds
- Using the `handoff()` function with custom `tool_description_override`
- Practice exercises

### 15: Agents as Tools (`15_Agents_As_Tools.ipynb`)
- Calling agents like functions from an orchestrator
- Handoff vs agent-as-tool — when to use each
- Practice exercises

### 15A: Async Python Basics (slides/video only)
- What async/await is and why it matters — conceptual foundation
- How `asyncio.gather()` runs tasks concurrently
- Note: Lesson 16 opens with a minimal syntax recap; this slides lesson covers the concept, Lesson 16 covers the code

### 16: Parallel Execution (`16_Parallel_Execution.ipynb`)
- Minimal async recap at the top — await, gather, what changes when one task fails
- Running agents concurrently with asyncio
- Merging results from parallel agents
- Cost and speed tradeoffs — when parallel specialists beat one bigger model
- What happens when one task fails
- `asyncio.gather` with `return_exceptions`
- Partial result handling
- Timeout and fallback behavior
- Practice exercises

### 17: Debate & Critique Pattern (`17_Debate_And_Critique.ipynb`)
- Proposer / Critic architecture
- Agents that challenge each other's outputs
- Improving quality through internal debate
- Critique loop termination rules — avoiding infinite loops
- Practice exercises

### 18: Capstone #2 — Multi-Agent Research Team (`18_Capstone_2_Research_Team.ipynb`)
- Procedural multi-agent pipeline — explicit phase-by-phase coordination in code
- Researcher (web search), Analyst (code interpreter), Writer, Critic
- Parallel research and analysis execution (`asyncio.gather` with `return_exceptions`)
- Sequential writing, critique, and revision phases
- One specialist fails — pipeline detects and recovers with partial results
- No silent degradation — fallback text keeps downstream phases running
- Evaluation component: judge agent rubric and golden test set pattern from Lesson 09 applied to multi-agent output
- Exercise: add a fact checker agent as Phase 5

---

## WEEK 4: MEMORY, SAFETY & OBSERVABILITY (Week_4_Memory_Safety_Observability)

### 19: Sessions & Conversation State (`19_Sessions_And_Conversation_State.ipynb`)
- Why agents forget between runs
- Sessions — automatic conversation history
- In-memory vs persistent (file-based) sessions
- Sharing sessions across multiple agents
- Inspecting and clearing session state
- Note: session growth and context window management strategies covered in Lesson 20
- Practice exercises

### 20: Persistent Memory (`20_Persistent_Memory.ipynb`)
- Short-term vs long-term memory
- Persistent memory with SQLite
- Storing preferences and facts across sessions
- What NOT to store — privacy and sensitive data boundaries
- Stale memory cleanup and correcting wrong memories
- Conflicting memories and user-controlled forgetting
- Avoiding noisy memory — when to summarize vs drop
- Practice exercises

### 21: Vector Memory with ChromaDB (`21_Vector_Memory.ipynb`)
- What vector databases are and how they differ from SQLite
- Embeddings — storing meaning, not just text
- Semantic retrieval — finding relevant memories by meaning
- Building a vector memory store with ChromaDB
- When to use vector memory vs SQLite
- Practice exercises

### 22: Guardrails (`22_Guardrails.ipynb`)
- Input guardrails — validate what comes in
- Output guardrails — validate what goes out
- Tripwires — halt execution immediately
- Parallel vs blocking execution modes
- Practice exercises

### 23: Prompt Injection & Tool Safety (`23_Prompt_Injection_And_Tool_Safety.ipynb`)
- What prompt injection is and why agents are uniquely vulnerable
- Injection via web pages, documents, and MCP tool output (MCP is introduced in Lesson 27 — forward reference here)
- Separating system instructions from retrieved data
- Read tools vs write tools — why write actions need stronger controls
- Retries are more dangerous with side effects — idempotency matters
- Least-privilege tool design
- Confirming dangerous actions
- Practice exercises

### 24: Human-in-the-Loop (`24_Human_In_The_Loop.ipynb`)
- When to pause for human approval
- Threshold-based escalation
- Approval workflow pattern
- Practice exercises

### 25: Tracing & Observability (`25_Tracing_And_Observability.ipynb`)
- What tracing captures automatically
- Reading traces in the OpenAI dashboard
- Inspecting tool calls, handoffs, and decisions
- Monitoring latency, token usage, and cost
- Debugging bad runs systematically — connecting reliability, evaluation, and observability
- Practice exercises

### 26: Capstone #3 — Production Customer Service Agent (`26_Capstone_3_Customer_Service.ipynb`)
- Multi-tool agent: order lookup, FAQ search, refund processing
- Sessions for conversation memory
- Guardrails blocking off-topic requests
- Prompt injection awareness
- Human-in-the-loop for refund approvals — explicit pause point that reinforces read vs write tool safety
- Tracing and monitoring
- Exercise: evaluate the agent against a golden test set — check tool-use correctness, output quality, and guardrail behavior
- Exercise: add a new tool and guardrail

---

## WEEK 5: MCP & PRODUCTION (Week_5_MCP_And_Production)

### 27: MCP Fundamentals (`27_MCP_Fundamentals.ipynb`)
- What MCP is — the USB-C analogy
- How MCP servers relate to `@function_tool` and built-in tools — three ways to give an agent capabilities
- MCP servers, clients, tools, and an introduction to resources
- Connecting to a first MCP server
- Agent discovers tools automatically — no custom integration code
- Practice exercises

### 28: Real-World MCP Servers (`28_Real_World_MCP_Servers.ipynb`)
- Filesystem MCP server
- Web fetch MCP server
- Combining multiple MCP servers in one agent
- Common MCP failure modes — debugging tool discovery and flaky servers
- Reasoning about server permissions — when not to expose a whole filesystem
- Testing MCP tools before trusting them in an agent
- ⚠️ Security note: MCP tool output is untrusted input — revisits prompt injection from Lesson 23 in the MCP context
- Practice exercises

### 29: Capstone #4 — MCP-Powered Personal Assistant (`29_Capstone_4_MCP_Assistant.ipynb`)
- Agent connects to filesystem, web, and one additional MCP server
- Handles real requests: summarize files, research topics, save results
- Server-level approval before writing or deleting
- Saving results is a side effect — handle it with the same care as write actions
- `REASONING_MODEL` for decision-making
- Exercise: swap in a different MCP server

### 30: Project Structure & CLI (`30_Project_Structure_And_CLI.ipynb`)
- What changes when leaving notebooks
- Separating config, tools, and agent logic into modules
- A reusable project structure for agent apps
- Running an agent from a script or CLI
- Streaming agent output in a script — showing responses as they arrive before Gradio
- When to drop down to `client.responses.create()` directly — and when not to
- Secret management — never hardcode API keys, local `.env` for dev, platform secrets for deployment
- When to consider a simple API wrapper

### 31: Architecture Decisions (`31_Architecture_Decisions.ipynb`)
- Model selection framework — mini vs reasoning vs parallel specialists; use your Lesson 09 test sets to justify model upgrades with data
- Single agent vs multi-agent — decision framework
- Tool-heavy vs instruction-heavy agents
- When memory and sessions add value vs add noise
- When MCP is worth the setup
- Avoiding unnecessary complexity
- Course wrap-up and suggested next projects

### 32: Deploying Agents with Gradio (`32_Deploying_With_Gradio.ipynb`)
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

**Total:** 5 weeks, 32 notebooks, plus 2 slides/video-only lessons