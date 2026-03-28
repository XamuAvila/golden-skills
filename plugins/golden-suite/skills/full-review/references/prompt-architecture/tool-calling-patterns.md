# Tool Calling Patterns for OpenAI API

Patterns and best practices for designing prompts that interact with OpenAI function calling / tool use.

---

## Core Pattern: ReAct Loop

The dominant pattern for agentic tool use:
1. **Reason** about the current state and what information is needed
2. **Act** by selecting and invoking a tool with appropriate parameters
3. **Observe** the tool output and incorporate into reasoning
4. **Loop** until the task is complete or a termination condition is met

## Tool Description Quality

Tool/function descriptions ARE part of the prompt. The model uses them as its only documentation.

### Required elements in every tool description:
- **Purpose**: One sentence explaining what the tool does and when to use it
- **Parameters**: Each with type, constraints, valid ranges, and examples
- **Expected output**: What the response looks like (schema or example)
- **Edge cases**: What happens on empty input, invalid data, or errors
- **When NOT to use**: Explicit exclusions prevent misuse

### Anti-pattern:
```json
{
  "name": "search",
  "description": "Searches for stuff"
}
```

### Pattern:
```json
{
  "name": "search_knowledge_base",
  "description": "Searches the internal knowledge base for documents matching a semantic query. Use when the user asks about company policies, procedures, or product documentation. Do NOT use for general knowledge questions or web searches.",
  "parameters": {
    "query": {
      "type": "string",
      "description": "Natural language search query. Be specific — 'vacation policy for contractors' not 'policy'."
    },
    "max_results": {
      "type": "integer",
      "description": "Number of results to return. Default 5. Range: 1-20."
    },
    "filter_category": {
      "type": "string",
      "enum": ["hr", "engineering", "product", "legal"],
      "description": "Optional. Restrict results to a specific category."
    }
  }
}
```

## Parallel vs. Sequential Tool Calls

- **Parallel**: Independent lookups that don't depend on each other. Explicitly tell the model: "You may call multiple tools simultaneously when their inputs are independent."
- **Sequential**: Output of tool A feeds into tool B. Make the dependency explicit: "First retrieve the user profile, then use the user_id from the profile to fetch their orders."

## Error Handling Guidance

Include in the system prompt what the agent should do when a tool fails:

```
When a tool call returns an error:
1. If the error is retryable (timeout, rate limit): retry once with the same parameters.
2. If the error indicates invalid input: fix the parameters and retry.
3. If the error is permanent (not found, unauthorized): report the failure to the user with the specific error. Do NOT fabricate a response.
4. Never silently ignore tool errors.
```

## Tool Selection Guidance

When multiple tools could serve the same purpose, add explicit routing logic to the system prompt:

```
Tool selection rules:
- For questions about current data (prices, availability, status): use `query_database`
- For questions about historical trends: use `analytics_api`
- For questions about policies or procedures: use `search_knowledge_base`
- If unsure which tool to use: ask the user to clarify before calling any tool
```

## Persistence and Planning

Research shows that adding these elements to agentic prompts increases task completion by ~20%:

1. **Persistence instructions**: "Continue working toward the goal even if individual steps fail. Try alternative approaches before giving up."
2. **Planning requirements**: "Before executing, outline your plan in 2-3 steps. After each tool call, assess progress against the plan."
3. **Progress tracking**: "After each step, state: what you did, what you learned, and what you'll do next."

## Token Efficiency

- Keep tool descriptions precise but complete — every unnecessary word costs tokens on every API call
- Use `enum` constraints to limit parameter values instead of describing valid options in text
- Group related tools logically so the model can reason about which category to use first
