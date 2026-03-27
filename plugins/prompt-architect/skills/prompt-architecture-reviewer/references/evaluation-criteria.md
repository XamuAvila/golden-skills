# Prompt Evaluation Criteria

The 10 criteria used by the prompt-architecture-reviewer to analyze and score prompts. Each criterion includes what to look for, common failures, and how to fix them.

---

## 1. Instruction Clarity

**Question**: Could a colleague understand this prompt with no additional context?

**What to check**:
- Is the core task stated in the first 1-2 sentences?
- Are instructions unambiguous — only one possible interpretation?
- Is there a clear distinction between what to do and what NOT to do?
- Are there conflicting instructions?

**Common failures**:
- Burying the main task inside paragraphs of context
- Using vague verbs: "handle", "process", "deal with" instead of "classify", "extract", "generate"
- Contradictory instructions that force the model to guess which to follow

**Fix**: Lead with the task. Use specific action verbs. Remove contradictions.

---

## 2. Sufficient Context

**Question**: Does the model have all the background it needs to perform the task correctly?

**What to check**:
- Is the domain defined? (industry, product, audience)
- Is the user's situation described?
- Are acronyms, jargon, or domain-specific terms defined?
- Is there too much irrelevant context diluting the signal?

**Common failures**:
- Assuming the model knows your product, company, or internal terms
- Dumping entire documents when only a section is relevant
- Providing no context at all — "summarize this"

**Fix**: Include minimum necessary context. Define domain terms. Remove noise.

---

## 3. Well-Defined Task

**Question**: Is there exactly one clear task, or is it a bundle of vaguely related requests?

**What to check**:
- Is there a single primary task?
- If multiple subtasks exist, are they decomposed into ordered steps?
- Is the scope bounded — does the model know when it's "done"?
- Are success criteria stated?

**Common failures**:
- "Analyze this data and also create a report and suggest improvements" (3 tasks in 1)
- Open-ended tasks with no completion criteria
- Mixing analysis with generation with evaluation

**Fix**: One task per prompt, or explicit decomposition into numbered steps.

---

## 4. Explicit Constraints

**Question**: Are the boundaries clearly stated?

**What to check**:
- Length limits (word count, token count, number of items)
- Scope exclusions ("do not discuss X", "only consider Y")
- Tone and style requirements
- Language requirements
- Compliance or policy constraints

**Common failures**:
- No length guidance — model writes 2000 words when you needed 200
- Implicit assumptions about tone or formality
- Missing safety constraints for user-facing applications

**Fix**: State every constraint explicitly. "Maximum 3 sentences." "Formal tone." "English only."

---

## 5. Defined Output Format

**Question**: Does the model know exactly what the response should look like?

**What to check**:
- Is the format specified? (JSON, markdown, bullet list, table, prose)
- Is a JSON schema provided for structured outputs?
- Are field names, types, and valid values defined?
- Is there an example of the expected output?

**Common failures**:
- "Return a structured response" without defining the structure
- Specifying JSON but not the schema
- Using `response_format: json_object` without a schema (allows any valid JSON)

**Fix**: Provide explicit format specification. Use `json_schema` with `strict: true` when possible. Include one example.

---

## 6. Appropriate Use of Examples

**Question**: Are few-shot examples used effectively?

**What to check**:
- Are 2-5 diverse examples provided for novel or ambiguous tasks?
- Do examples cover normal case, edge case, and ambiguous case?
- Are examples consistent with the stated instructions?
- Are examples REMOVED when using structured output mode (often unnecessary)?

**Common failures**:
- No examples for a task the model hasn't seen before
- All examples show the same type of input (no diversity)
- Examples that contradict the instructions
- Too many examples that waste context window

**Fix**: 3 diverse examples covering different scenarios. Ensure consistency with instructions.

---

## 7. Separation of Instruction, Context, and Data

**Question**: Are the three types of content clearly delimited?

**What to check**:
- Are instructions (what to do) separated from context (background)?
- Is user input/data clearly delimited from the prompt itself?
- Are delimiters used consistently (XML tags, markdown, separators)?
- Could injected user content be confused with instructions?

**Common failures**:
- Mixing instructions into the middle of context paragraphs
- No delimiters between system instructions and user-provided content
- Prompt injection vulnerability — user input can override instructions

**Fix**: Use message roles (system, user, assistant). Add delimiters. Isolate user content in tags like `<user_input>`.

---

## 8. Task Decomposition

**Question**: Is the task appropriately decomposed for the model's capabilities?

**What to check**:
- Is a complex task broken into sequential steps?
- Are steps ordered logically (dependencies respected)?
- Could the task be split into multiple API calls for better reliability?
- Is each step simple enough for reliable execution?

**Common failures**:
- Single monolithic prompt for a 5-step workflow
- No step ordering — model chooses random execution order
- Steps that are too fine-grained (micromanaging the model)

**Fix**: Decompose into 3-7 numbered steps. Consider chaining multiple API calls for complex workflows.

---

## 9. Appropriate Use of Tools/Retrieval

**Question**: Are tool-calling and RAG patterns used correctly?

**What to check**:
- Are tool descriptions precise, typed, and edge-case-aware?
- Is there routing logic for choosing between tools?
- Are error handling instructions included?
- Is retrieval used instead of relying on model knowledge for facts?
- Are retrieved documents properly delimited and referenced?

**Common failures**:
- Vague tool descriptions that cause misuse
- No error handling — model hallucinates when tool fails
- Relying on model knowledge for current/factual information
- No source citation instructions for RAG

**Fix**: Write tool descriptions as documentation. Add error handling. Use RAG for factual accuracy.

---

## 10. Testability and Versioning

**Question**: Can this prompt be tested, measured, and maintained over time?

**What to check**:
- Are there defined test cases (input → expected output)?
- Are success metrics specified?
- Is the prompt version-controlled?
- Are prompt variables clearly separated from static instructions?
- Can the prompt be A/B tested against alternatives?

**Common failures**:
- Hardcoded values that should be variables
- No way to measure if a prompt change improved or degraded quality
- Prompt lives in application code instead of being managed separately
- No regression testing after prompt changes

**Fix**: Parameterize dynamic values. Define evaluation criteria. Version-control prompts as configuration.

---

## Scoring Guide

When reviewing a prompt, rate each criterion:

| Score | Meaning |
|-------|---------|
| **0 — Missing** | Criterion not addressed at all |
| **1 — Weak** | Partially addressed but has significant gaps |
| **2 — Adequate** | Addressed but could be improved |
| **3 — Strong** | Well-addressed, follows best practices |

**Overall prompt quality**:
- **24-30**: Production-ready
- **18-23**: Needs minor improvements
- **12-17**: Needs significant rework
- **0-11**: Fundamental redesign required
