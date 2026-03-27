# Structured Output Patterns for OpenAI API

Patterns for designing prompts that produce reliable, parseable structured outputs (JSON, schemas, typed responses).

---

## 1. Use Native Structured Output Mode

When possible, use OpenAI's native structured output support rather than prompt-only instructions:

```python
response = client.chat.completions.create(
    model="gpt-4o",
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "analysis_result",
            "strict": True,
            "schema": { ... }
        }
    },
    messages=[...]
)
```

**Why**: Native mode guarantees valid JSON conforming to the schema. Prompt-only approaches produce valid JSON ~90-95% of the time.

## 2. Schema Design Principles

### Keep schemas flat when possible
Deeply nested schemas increase error rates. Flatten when the nesting doesn't carry semantic meaning.

### Use enums for constrained fields
```json
{
  "severity": { "type": "string", "enum": ["critical", "high", "medium", "low"] }
}
```

### Make fields required or provide defaults
Optional fields without defaults cause inconsistent outputs. Either require the field or specify what the default should be.

### Add descriptions to every field
```json
{
  "risk_score": {
    "type": "number",
    "description": "Risk score from 0.0 (no risk) to 1.0 (critical risk). Base this on the number and severity of issues found."
  }
}
```

## 3. Format Specification Placement

Place output format instructions at the END of the prompt, after context and task description:

```
[Context and source material]

[Task instructions]

[Output format specification — last]
```

**Why**: Recency bias means the model pays more attention to later instructions. Format specs at the end produce more consistent adherence.

## 4. Output Anchoring

Start the assistant's response with the expected structure to constrain format:

```python
messages = [
    {"role": "system", "content": "..."},
    {"role": "user", "content": "Analyze this code..."},
    {"role": "assistant", "content": '{"analysis": {'}  # prefill
]
```

**When to use**: When you can't use native structured output mode (e.g., streaming, older models, or complex conditional schemas).

## 5. Delimiter Strategy

Use consistent delimiters to separate structured sections within prompts:

| Delimiter | Best For |
|-----------|----------|
| XML tags (`<context>`, `<output>`) | Complex prompts with multiple sections |
| Markdown headers (`## Section`) | Human-readable prompts |
| Triple backticks (` ``` `) | Code blocks and examples |
| Triple dashes (`---`) | Section separators |

**Consistency matters more than choice**. Pick one style and use it throughout.

## 6. Few-Shot Examples for Structure

When teaching the model a custom output format, provide 2-3 diverse examples:

```
Example 1 (simple case):
Input: "The login button is blue"
Output: {"element": "button", "property": "color", "value": "blue", "issues": []}

Example 2 (issue found):
Input: "The modal has no close button"
Output: {"element": "modal", "property": "accessibility", "value": null, "issues": ["missing_close_button"]}

Example 3 (ambiguous case):
Input: "The page feels slow"
Output: {"element": "page", "property": "performance", "value": "subjective", "issues": ["needs_measurement"]}
```

**Diversity matters more than quantity**. Cover: normal case, edge case, ambiguous case.

## 7. Handling Complex Conditional Outputs

When the output structure varies by condition, make the conditions explicit:

```
If the input contains code:
  Return {"type": "code_review", "language": "...", "issues": [...]}

If the input is a question:
  Return {"type": "answer", "confidence": 0.0-1.0, "response": "..."}

If the input is ambiguous:
  Return {"type": "clarification_needed", "questions": [...]}
```

## 8. Validation Instructions

Include self-validation steps in the prompt:

```
Before returning your response:
1. Verify the JSON is syntactically valid
2. Confirm all required fields are present
3. Check that enum fields contain only allowed values
4. Ensure numeric fields are within specified ranges
```

## 9. Streaming with Structured Outputs

When streaming structured responses:
- Use `json_schema` response format (supported with streaming in newer API versions)
- Or use XML tags for incremental parsing: `<result>...</result>` can be parsed as chunks arrive
- Avoid relying on complete JSON parsing during streaming — buffer until complete
