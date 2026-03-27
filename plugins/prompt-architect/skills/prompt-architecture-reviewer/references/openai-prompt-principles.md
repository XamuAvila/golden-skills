# OpenAI Prompt Architecture Principles

Foundational principles for designing production-grade prompts for OpenAI API. Synthesized from OpenAI official documentation, research papers, and production best practices.

---

## 1. Clarity Over Complexity

A prompt should be understandable by a human colleague with minimal context. If a person would be confused, the model will be too. Modern models often perform best with concise, well-structured prompts rather than verbose instructions.

**Test**: Read the prompt aloud. If you stumble, simplify it.

## 2. Minimum Effective Context

Well-structured short prompts (under 300 words for moderate tasks) consistently outperform longer ones. Research shows that a focused 16K-token prompt with retrieval augmentation outperformed a sprawling 128K-token prompt.

**Rule**: The smallest high-signal token set that maximizes desired outcomes.

## 3. Structure Is a Force Multiplier

Numbered lists, bullet points, delimiters (`###`, `===`, XML tags `<context>`, `<instructions>`), and section headers improve consistency by 30%+ in measured experiments. Models parse structured input more reliably than prose.

**Preferred delimiters for OpenAI models**: Markdown headers, XML tags, or triple-dash separators.

## 4. Decomposition and Aggregation

These are the two meta-principles underlying nearly all reliability techniques:
- **Decomposition**: Complex tasks should be split into smaller, more reliable subtasks.
- **Aggregation**: Multiple outputs or steps should be aggregated so the system's overall reliability exceeds any single component.

## 5. Explicit Over Implicit

State desired behaviors, constraints, format, and success criteria directly. Omitting constraints leads to drift; conflicting instructions cause the model to waste tokens reconciling them.

**Anti-pattern**: "Write a good summary" (implicit criteria).
**Pattern**: "Write a 3-sentence summary covering: main argument, key evidence, and conclusion. Do not include opinions." (explicit criteria).

## 6. Output Anchoring

Prefilling the beginning of a response steers the model toward the exact structure you want. In the API, use the `assistant` message to start the response format.

**Example**: Start with `{"analysis": {` to force JSON output structure.

## 7. Context Placement Matters

For long documents, place source material first and instructions at the end. Use anchor phrases like "Based on the entire document above..." Transformers exhibit recency bias, weighting later tokens more heavily.

**Rule**: Source material → Context → Instructions → Output format (in that order for long-context prompts).

## 8. Separation of Concerns

Every prompt should clearly separate three types of content:
- **Instructions**: What the model should do (system prompt)
- **Context**: Background information, retrieved documents, conversation history
- **Data/Input**: The specific user query or content to process

Mixing these causes ambiguity. Use delimiters, XML tags, or message roles to enforce separation.

## 9. Fail-Safe Design

Prompts should handle edge cases explicitly:
- What to do when information is missing or insufficient
- How to respond when the task is ambiguous
- When to refuse or ask for clarification instead of guessing
- How to handle tool errors or unexpected inputs

## 10. Versioning and Testability

Production prompts are code. They should be:
- **Version-controlled**: Track changes, review diffs
- **Testable**: Define expected outputs for known inputs
- **Measurable**: Define success metrics before deploying
- **Reproducible**: Same input should produce consistent quality
