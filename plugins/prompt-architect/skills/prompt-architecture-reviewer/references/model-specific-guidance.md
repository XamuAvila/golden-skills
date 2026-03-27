# Model-Specific Guidance for OpenAI Stack

Prompt design guidance specific to each model in the OpenAI ecosystem. Models behave differently — a prompt optimized for GPT-4o may underperform on o1 and vice versa.

---

## GPT-4o / GPT-4o-mini

**Type**: General-purpose chat models

### Prompt characteristics:
- Respond well to detailed system prompts with structured sections
- Benefit from Chain-of-Thought reasoning ("Let's think step by step")
- Follow few-shot examples reliably (3-5 diverse examples recommended)
- Support native function calling and structured outputs (json_schema)
- Handle long context well (128K tokens) but exhibit recency bias

### System prompt structure:
```
[Role and persona definition]
[Task description and scope]
[Step-by-step instructions or rules]
[Constraints and boundaries]
[Few-shot examples if needed]
[Output format specification]
```

### Best practices:
- Use XML tags or markdown headers to separate sections
- Place reference material before instructions
- Be explicit about what NOT to do
- Use `response_format: { type: "json_schema" }` for structured output
- Temperature 0.0-0.3 for deterministic tasks, 0.7-1.0 for creative tasks

### GPT-4o-mini differences:
- More cost-efficient but less capable on nuanced reasoning
- Benefits more from explicit few-shot examples
- Keep prompts shorter and more direct — less tolerance for ambiguity
- Better suited for classification, extraction, and simple generation
- Avoid complex multi-step reasoning without decomposition

---

## o1 / o3 (Reasoning Models)

**Type**: Extended reasoning models with internal chain-of-thought

### Critical differences from GPT-4o:
- These models reason INTERNALLY before responding
- Adding "think step by step" provides only +2.9% improvement while increasing processing time 20-80%
- Benefit from CONCISE, DIRECT instructions — not verbose prompts
- System prompt role is more limited — use `developer` message instead of `system`

### Prompt characteristics:
- **Do**: State the goal clearly and concisely
- **Do**: Provide constraints and edge cases
- **Do**: Include evaluation criteria for the answer
- **Don't**: Add step-by-step reasoning instructions (the model does this internally)
- **Don't**: Use verbose role-playing setups
- **Don't**: Add Chain-of-Thought prompting

### System prompt structure:
```
[Goal: one clear sentence]
[Context: only what's necessary]
[Constraints: explicit boundaries]
[Output format: what you expect back]
```

### Best practices:
- Keep system prompts under 200 words
- Use `reasoning_effort` parameter: "low" for simple tasks, "high" for complex analysis
- Let the model decide HOW to reason — just tell it WHAT you need
- Structured outputs work but keep schemas simple
- Higher latency is expected — batch non-urgent requests

---

## GPT-4.1 / GPT-4.1-mini / GPT-4.1-nano

**Type**: Latest generation instruction-following models

### Prompt characteristics:
- Excellent at following complex, multi-part instructions
- Strong agentic capabilities with improved tool use
- Better at long-context tasks with reduced recency bias
- Support for parallel tool calls natively
- More reliable structured output adherence

### Best practices:
- Can handle longer, more detailed system prompts than previous generations
- Agentic prompts benefit from explicit planning steps
- Tool descriptions should be comprehensive — these models use them effectively
- Well-suited for multi-agent coordinator/supervisor roles

---

## Fine-Tuned Models

**Type**: Custom models trained on your data

### Prompt characteristics:
- System prompts should be SHORTER — the fine-tuning already encodes behavior
- Don't repeat instructions that are baked into the training data
- Focus the prompt on dynamic context and specific input
- Few-shot examples are usually unnecessary (the model already learned the pattern)

### System prompt structure:
```
[Minimal role reminder — 1 sentence]
[Dynamic context for this specific request]
[Input data]
```

### Best practices:
- Test with minimal prompts first — add instructions only if behavior drifts
- Keep the system prompt consistent with what was used during fine-tuning
- Changing the prompt style post-fine-tuning can degrade performance
- Use for high-volume, narrow tasks where prompt tokens cost matters

---

## Model Selection Guide

| Scenario | Recommended Model | Why |
|----------|------------------|-----|
| General API backend | GPT-4o | Best balance of capability, speed, and cost |
| High-volume classification | GPT-4o-mini | Cost-efficient for simple tasks |
| Complex reasoning / math | o1 or o3 | Internal reasoning produces better analytical results |
| Multi-agent coordinator | GPT-4.1 | Best agentic capabilities |
| Specialized narrow task | Fine-tuned GPT-4o-mini | Lowest per-token cost with trained behavior |
| Code generation | GPT-4o or o3 | Strong code capabilities with reasoning |
| Real-time user-facing chat | GPT-4o-mini | Low latency, good enough quality |
| Document analysis | GPT-4o (128K) | Long context with good instruction following |

---

## Cross-Model Anti-Patterns

1. **Using the same prompt across all models**: Each model family has different strengths. Test and adapt.
2. **Adding CoT to reasoning models**: Wastes tokens and adds latency with minimal benefit.
3. **Verbose prompts for fine-tuned models**: The model already knows the task — over-prompting confuses it.
4. **Testing on GPT-4o, deploying on mini**: Performance characteristics differ. Always test on the production model.
5. **Ignoring reasoning_effort parameter**: Using "high" for simple tasks wastes compute; "low" for complex tasks produces shallow answers.
