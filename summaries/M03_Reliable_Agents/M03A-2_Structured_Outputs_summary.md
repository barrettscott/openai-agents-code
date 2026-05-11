

---

# M03A-2: Structured Outputs — Recording Prep

## What this notebook teaches
This notebook teaches students how to replace free-form text responses from agents with guaranteed, typed output structures using Pydantic models and the `output_type=` parameter. It matters because the moment agent output feeds into downstream code — aggregation, filtering, database writes, another agent — free-form text breaks unpredictably. Structured outputs make `result.final_output` a Pydantic model instance with dot-accessible fields instead of a raw string, eliminating all string parsing.

## Concepts to explain on camera

**Structured Output (via `output_type=`)**
- **Term:** A feature of the OpenAI Agents SDK that constrains the agent's response to match a specific Pydantic model shape. The SDK converts your model into a JSON schema, sends it with the request, and validates the response against it automatically.
- **Analogy:** It's like giving someone a form to fill out instead of asking them to write a letter. The form has specific boxes — they can't rearrange them or skip required ones.
- **What students need to understand:**
  - You pass `output_type=YourModel` on the `Agent` constructor, not on `Runner.run()`
  - `result.final_output` becomes an instance of your Pydantic model, not a string
  - You access fields with dot notation: `result.final_output.sentiment`, not dictionary brackets
  - The SDK handles schema generation and validation — you just define the model

**`Annotated[float, Field(ge=0.0, le=1.0)]`**
- **Term:** A way to add constraints to a Pydantic field beyond just its type. `Annotated` wraps the type with metadata, and `Field(ge=0.0, le=1.0)` says "greater than or equal to 0, less than or equal to 1." This constraint gets baked into the JSON schema the model sees.
- **Analogy:** It's like setting min/max on a slider in a UI — the user can only pick values within the range you allow.
- **What students need to understand:**
  - `ge` = greater than or equal, `le` = less than or equal
  - The constraint is enforced during Pydantic validation, so out-of-range values raise `ValidationError`
  - The model also sees these constraints in the schema, so it's guided to stay in range
  - Used in this notebook for `confidence` (0.0–1.0) and `rating` (1–5)

**`Optional[str] = None`**
- **Term:** A Python typing pattern that says "this field can be a string or `None`." The `= None` default means the field isn't required — if the agent doesn't have enough information, it returns `None` instead of making something up.
- **Analogy:** It's like a form field marked "(optional)" — leaving it blank is valid, but you can still fill it in when you have the information.
- **What students need to understand:**
  - Use `Optional` when the agent might not have enough context to fill a field
  - Downstream code can check `if contact.email:` to handle missing data cleanly
  - Adding an `Optional` field to an existing model is safe — existing code that doesn't reference it keeps working
  - This is different from a required field, which the agent must always populate

**Schema Evolution**
- **Term:** Changing a Pydantic model over time — adding, removing, or modifying fields — as requirements change. The key question is whether changes break existing pipeline code.
- **Analogy:** It's like adding a new column to a spreadsheet. If the column is optional, all your existing formulas still work. If it's required, every row now needs a value.
- **What students need to understand:**
  - Adding an `Optional` field is always safe — it defaults to `None`
  - Adding a required field forces the agent to populate it every time
  - Removing a field drops it silently — but any code that referenced it will break
  - This is why you start minimal and add fields only when downstream code needs them

**`ValidationError`**
- **Term:** An exception Pydantic raises when the agent's response doesn't match the schema — wrong type, missing required field, value out of range. The SDK parses the response and validates it; if validation fails, you get this error.
- **Analogy:** It's like a form being rejected because a required field was left blank or a phone number was entered where an email was expected.
- **What students need to understand:**
  - In practice this is rare because the model is guided by the schema, but it can happen
  - Wrapping `Runner.run()` in try/except `ValidationError` prevents crashes
  - The error object tells you exactly which field failed and why (`e.errors()`)
  - Full error handling patterns are covered in M03B — this is specifically for structured output failures

## Key SDK pattern

```python
class SentimentResult(BaseModel):
    sentiment: str
    confidence: Annotated[float, Field(ge=0.0, le=1.0)]
    key_phrase: str
    reasoning: str

sentiment_agent = Agent(
    name="SentimentAnalyst",
    instructions=instructions,
    model=MODEL,
    output_type=SentimentResult      # ← This is the key addition
)

result = await Runner.run(sentiment_agent, input="The product arrived on time and works great.")

analysis = result.final_output       # ← This is now a SentimentResult, not a string
print(analysis.sentiment)            # ← Dot notation, not string parsing
print(analysis.confidence)
```

**Parameter breakdown:**
- `output_type=SentimentResult` — tells the SDK to generate a JSON schema from this Pydantic model, instruct the model to conform to it, and validate the response. This goes on the `Agent` constructor, **not** on `Runner.run()`.
- `result.final_output` — without `output_type`, this is a plain string. With it, this becomes an instance of `SentimentResult` with typed, dot-accessible fields.
- The `Annotated[float, Field(ge=0.0, le=1.0)]` on `confidence` constrains the value range in both the schema (guiding the model) and validation (catching violations).
- The `instructions` string should align with the model fields — tell the agent to "classify as positive, negative, or neutral" when the field is `sentiment: str`.

## The demos and what each shows

**The Problem (unstructured agent):** This is the motivating example. An `UnstructuredAnalyst` agent analyzes sentiment but returns free-form text. The code then tries `output["sentiment"]` and gets a `TypeError` because you can't index into a string with a key. Emphasize on camera: the agent's answer is *correct* — it just isn't *usable*. Run this and let the error print. Say something like "Every time you run this, the phrasing changes. Sometimes it says 'This is positive,' sometimes 'Sentiment: positive.' All correct, none consistent, all break downstream code."

**Part 1 (SentimentResult with `output_type=`):** The fix. Same task, but now with a `SentimentResult` model and `output_type=SentimentResult`. The output comes back as a proper object — `analysis.sentiment`, `analysis.confidence`, `analysis.key_phrase`, `analysis.reasoning` all accessible via dot notation. Emphasize on camera: "This is the same agent task. The difference is one parameter: `output_type`. Now `result.final_output` is a `SentimentResult` instance, not a string. Every field is typed, every run produces the same shape."

**Part 2 (ProductReview pipeline):** Shows why structured outputs matter in real workflows. Three product reviews are processed in a loop, each returning a `ProductReview` with `rating`, `pros`, `cons`, `recommended`, and `summary`. After the loop, the code computes `avg_rating` with `sum(r.rating for r in results)` and counts recommendations — pure Python, no parsing. Emphasize on camera: "This is the payoff. Because every result is a `ProductReview` instance, the aggregate math just works. Imagine doing this with free-form text — you'd need regex, string splitting, and prayer."

**Part 3 (ContactInfo with Optional fields):** Three texts with varying amounts of contact information. "Sarah Chen" has name, email, and phone. "Contact Marcus" has only a name. The `Optional[str] = None` fields handle missing data gracefully — `contact.email or '—'` displays a dash for missing fields, and `is_complete` flags whether enough info was extracted. Emphasize on camera: "Not every input has every field. `Optional` makes missing data explicit — `None` means absent, not wrong. Downstream code can safely branch on this."

**Part 4 (Schema Evolution — ProductReviewV2):** Takes the original `ProductReview` model and adds `sentiment_score: Optional[float] = None`. The agent populates it when it can. Existing pipeline code (which never referenced `sentiment_score`) would still work unchanged. Emphasize on camera: "This is how you evolve your schemas over time. Adding an optional field is always safe. Your existing code doesn't break because it never asked for that field."

**Safe Validation Pattern:** A reusable `run_with_validation()` async function that wraps `Runner.run()` in a try/except `ValidationError`. If validation fails, it prints the error count and which field failed. In the demo it succeeds, but walk through what *would* happen if it failed. Emphasize on camera: "In practice, validation errors with structured outputs are rare because the model is guided by the schema. But when they happen, this pattern catches them gracefully instead of crashing your pipeline. We'll go much deeper on error handling in M03B."

## Gotchas worth knowing before recording

- **`output_type` goes on `Agent`, not `Runner.run()`** — the troubleshooting section calls this out, and it's the #1 mistake students will make. Mention it explicitly when you first show the parameter.
- **`result.final_output` type changes based on `output_type`** — without it, it's a `str`. With it, it's a Pydantic model instance. If you accidentally `print(result.final_output)` it will show the Pydantic repr, not pretty JSON. The demos use individual field access (`analysis.sentiment`) which looks cleaner on camera.
- **`instructions` variable reuse in Part 4** — `review_agent_v2` uses the `instructions` variable from Part 2's `ProductReview` agent. This is intentional (same review-analysis instructions), but if you run cells out of order, `instructions` might hold a different value. Be aware of this if you re-run cells during recording.
- **Confidence formatting: `{analysis.confidence:.0%}`** — the `:.0%` format spec turns `0.95` into `95%`. If a student asks why it says "95%" instead of "0.95", this is why. Worth a quick mention.
- **The `await` keyword** — every `Runner.run()` call uses `await`. Students haven't had the async deep-dive yet (that's M05C-1). For now, just say "we use `await` because the agent call is asynchronous — it needs to wait for the API response. We'll cover async in detail later in the course." Don't belabor it.
- **The unstructured demo's error might look confusing** — the `TypeError: string indices must be integers` message isn't intuitive. Explain: "We're trying to use dictionary-style access on a string. Python strings support indexing by position (`output[0]`), not by key (`output['sentiment']`). That's the error."
- **`Annotated` import** — comes from `typing`, not `pydantic`. It's imported in the setup cell. If a student copies code without the import, they'll get a `NameError`.
- **The "Marcus" contact extraction** — the agent might or might not set `is_complete` to `False` depending on interpretation. The instructions say "name and at least one contact method," and Marcus has no email or phone, so it should be `False`. But the model might interpret "Contact Marcus" as implying an implicit contact method. If this happens on camera, it's actually a great teaching moment about why instructions need to be precise.
- **`sentiment_score` in Part 4 might be `None`** — the agent may or may not populate it since the instructions don't explicitly mention it. The field name is descriptive enough that the model usually fills it, but don't be surprised if it's `None`. Either outcome is valid and teachable.

## How it connects to adjacent notebooks

**Builds on M03A-1 (Pydantic Basics):** Students just learned `BaseModel`, typed fields, `ValidationError`, and dot notation. Say on camera: "In the last notebook, we learned how to define Pydantic models and saw what happens when data doesn't fit. Now we're putting that to work — the agent's output has to match the model we defined." All the Pydantic concepts (fields, types, `Optional`, `Field` constraints) were introduced there; this notebook applies them to agent output.

**Builds on M02A (How Agents Work):** Students already know `Agent`, `Runner.run()`, and `result.final_output`. Say on camera: "You've been using `result.final_output` as a string. Adding `output_type` changes it from a string to a Pydantic model instance — same pattern, stronger output."

**Feeds into M03B (Error Handling & Recovery):** The Safe Validation Pattern at the end is a bridge. Say on camera: "We just saw the basics of catching validation errors. Next, in M03B, we'll cover the full picture — what happens when tools fail, how to retry safely, and how to build agents that recover gracefully."

**Feeds into M04D (Capstone #1):** The research agent capstone uses structured outputs for its final report. Say on camera: "Structured outputs will keep showing up. In the first capstone project, the agent produces a structured research report — same pattern, bigger model."

**Feeds into M05E (Capstone #2) and beyond:** Every capstone uses structured outputs. The pipeline pattern from Part 2 (processing multiple items, aggregating results) is the foundation for multi-agent systems where one agent's structured output feeds into the next agent's input.

---