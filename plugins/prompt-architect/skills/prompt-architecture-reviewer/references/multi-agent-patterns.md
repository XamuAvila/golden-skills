# Multi-Agent Prompt Patterns for OpenAI API

Architecture patterns and prompt design principles for multi-agent systems using OpenAI models.

---

## Topology Types

| Pattern | Description | Best For |
|---------|-------------|----------|
| **Single Agent** | One model + tools + comprehensive system prompt | Early development, fast prototyping, simple workflows |
| **Sequential** | Agents in predefined linear order, output chains | Predictable, repeatable processes (data pipeline, ETL) |
| **Parallel** | Multiple agents execute simultaneously, outputs synthesized | Independent subtasks needing low latency |
| **Loop** | Repeated agent sequence until termination condition | Iterative refinement without AI orchestration |
| **Review & Critique** | Generator + critic evaluating against criteria | Tasks requiring high accuracy or strict constraints |
| **Coordinator** | Central agent decomposes and dispatches to specialists | Dynamic routing, adaptive workflows |
| **Hierarchical** | Multi-level tree of agents | Ambiguous, open-ended problems requiring decomposition |
| **Aggregate** | Parallel agents + majority voting | Mathematical reasoning (5-9 parallel agents optimal) |
| **Debate** | Adversarial exchange between agents | Factual multi-hop reasoning (1-3 rounds optimal) |

## Task-to-Architecture Mapping

| Task Type | Optimal Pattern | Rationale |
|-----------|----------------|-----------|
| Factual multi-hop QA | Debate (1-3 rounds) | Adversarial exchange surfaces truthful answers |
| Mathematical reasoning | Aggregate (5-9 agents) | Diverse reasoning path exploration |
| Code generation | Aggregate + Reflect + Executor | Iterative debugging with external validation |
| Predictable workflows | Sequential | Lower latency, lower cost |
| Dynamic routing | Coordinator | Flexible sub-task dispatch |
| High-stakes decisions | Human-in-the-Loop | Safety at cost of complexity |
| Content generation | Review & Critique | Quality assurance through structured feedback |
| Complex analysis | Hierarchical | Break ambiguous problems into manageable pieces |

## Supervisor/Coordinator Prompt Design

The supervisor prompt is the most critical prompt in a multi-agent system. It must:

### Define the agent roster
```
You coordinate the following specialist agents:
- **researcher**: Searches knowledge bases and external sources. Use for fact-finding.
- **analyst**: Performs quantitative analysis on data. Use for calculations and comparisons.
- **writer**: Produces user-facing content. Use for final output generation.
- **reviewer**: Validates outputs against criteria. Use before delivering final results.
```

### Define routing logic
```
Routing rules:
1. Assess the user's request and identify required capabilities.
2. If the request requires factual information: dispatch to researcher first.
3. If the request requires calculations: dispatch to analyst (after researcher if data is needed).
4. For final output: always route through writer, then reviewer.
5. If reviewer identifies issues: route back to the responsible agent with specific feedback.
```

### Define termination conditions
```
Termination conditions:
- The reviewer approves the output with no critical issues.
- Maximum 3 refinement cycles have been completed.
- The user explicitly accepts the output.
- A blocking error occurs that no agent can resolve (report to user).
```

### Define context sharing rules
```
Context rules:
- Each agent receives only the context relevant to its task.
- The researcher's findings are shared with analyst and writer.
- The reviewer receives the full chain of outputs for validation.
- User PII is never forwarded to external tool-calling agents.
```

## Individual Agent Prompt Design

Each agent in a multi-agent system needs:

### 1. Clear role boundary
```
You are a code review specialist. You ONLY review code for bugs, security issues, and style violations. You do NOT fix code, write new code, or make architectural decisions. When you find an issue, describe it precisely and suggest a fix category (bug, security, style) but let the implementer decide the solution.
```

### 2. Input/output contract
```
You will receive:
- `code`: The code snippet to review (string)
- `language`: Programming language (string)
- `context`: Description of what the code does (string)

You must return JSON:
{
  "issues": [
    {
      "severity": "critical|high|medium|low",
      "line": <number>,
      "category": "bug|security|style|performance",
      "description": "<what is wrong>",
      "suggestion": "<how to fix>"
    }
  ],
  "summary": "<1-2 sentence overall assessment>",
  "approval": true|false
}
```

### 3. Scope constraints
```
Constraints:
- Do not comment on architectural decisions — that is the architect agent's role.
- Do not suggest refactoring beyond the scope of the submitted code.
- If the code is too short or lacks context to review meaningfully, return {"issues": [], "summary": "Insufficient context for review", "approval": false}.
```

## Prompt Optimization in Multi-Agent Systems

Research shows a three-stage optimization approach:

1. **Block-level** (~6% improvement): Optimize individual agent prompts (instructions + exemplars jointly)
2. **Topology-level** (~3% additional): Optimize system structure (which agents, what connections)
3. **Workflow-level** (~2% additional): Optimize cross-agent interdependencies conditioned on discovered optimal topology

**Key finding**: Only a small fraction of possible topologies actually improve performance. Indiscriminate scaling (adding more agents) degrades results. Start with the simplest topology that works and add complexity only when measured performance demands it.

## Anti-Patterns

1. **No exit conditions in loops**: Causes infinite cycles and uncontrolled costs. Every loop needs a max iteration count and a success condition.
2. **All-to-all communication (swarm) without orchestration**: Agents talk past each other, waste tokens, produce contradictory outputs.
3. **Scaling agents without improving prompts**: More agents with weak prompts = worse results. Optimize prompts first.
4. **Shared mutable state**: Agents overwriting each other's context. Use immutable message passing instead.
5. **Missing role boundaries**: Agents doing each other's work, producing duplicate or conflicting outputs.
