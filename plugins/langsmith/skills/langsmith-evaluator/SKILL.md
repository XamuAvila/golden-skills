---
name: langsmith-evaluator
description: "Use when building evaluation pipelines for LangSmith. Covers creating evaluators (LLM-as-Judge, custom code), defining run functions to capture outputs/trajectories, and running evaluations locally or via LangSmith auto-run."
---

# langsmith-evaluator

Three core components: **(1) Creating Evaluators** - LLM-as-Judge, custom code; **(2) Defining Run Functions** - capture agent outputs/trajectories; **(3) Running Evaluations** - locally with `evaluate()` or auto-run via uploaded evaluators.

## Setup

```bash
LANGSMITH_API_KEY=lsv2_pt_your_api_key_here
LANGSMITH_PROJECT=your-project-name
OPENAI_API_KEY=your_openai_key                        # For LLM as Judge
```

Python: `pip install langsmith langchain-openai python-dotenv`
CLI: `curl -sSL https://raw.githubusercontent.com/langchain-ai/langsmith-cli/main/scripts/install.sh | sh`

## Golden Rule: Inspect Before You Implement

Before writing ANY evaluator:
1. Run your agent on sample inputs and capture the actual output
2. Inspect the output — print it, query LangSmith traces
3. Only then write code that processes that output

## Evaluator Types

- **Offline** (attached to datasets): `(run, example)` — compares to expected values
- **Online** (attached to projects): `(run)` — real-time quality checks

Each evaluator returns **ONE metric only**. For multiple metrics, create separate functions.

## LLM as Judge (Python)

```python
from typing import TypedDict, Annotated
from langchain_openai import ChatOpenAI

class Grade(TypedDict):
    reasoning: Annotated[str, ..., "Explain your reasoning"]
    is_accurate: Annotated[bool, ..., "True if response is accurate"]

judge = ChatOpenAI(model="gpt-4o-mini", temperature=0).with_structured_output(Grade, method="json_schema", strict=True)

async def accuracy_evaluator(run, example):
    run_outputs = run.outputs if hasattr(run, "outputs") else run.get("outputs", {}) or {}
    example_outputs = example.outputs if hasattr(example, "outputs") else example.get("outputs", {}) or {}
    grade = await judge.ainvoke([{"role": "user", "content": f"Expected: {example_outputs}\nActual: {run_outputs}\nIs this accurate?"}])
    return {"score": 1 if grade["is_accurate"] else 0, "comment": grade["reasoning"]}
```

## Custom Code Evaluator (Python)

```python
def trajectory_evaluator(run, example):
    run_outputs = run.outputs if hasattr(run, "outputs") else run.get("outputs", {}) or {}
    example_outputs = example.outputs if hasattr(example, "outputs") else example.get("outputs", {}) or {}
    actual = run_outputs.get("YOUR_TRAJECTORY_FIELD", [])
    expected = example_outputs.get("YOUR_EXPECTED_FIELD", [])
    return {"score": 1 if actual == expected else 0, "comment": f"Expected {expected}, got {actual}"}
```

## Run Functions

```python
def run_agent(inputs: dict) -> dict:
    result = your_agent.run(inputs)
    print(f"DEBUG - type: {type(result)}, keys: {result.keys() if hasattr(result, 'keys') else 'N/A'}")
    return {"output": result}
```

## Uploading Evaluators

```bash
langsmith evaluator list
langsmith evaluator upload my_evaluators.py \
  --name "Trajectory Match" --function trajectory_evaluator \
  --dataset "My Dataset" --replace
langsmith evaluator upload my_evaluators.py \
  --name "Quality Check" --function quality_check \
  --project "Production Agent" --replace
langsmith evaluator delete "Trajectory Match"
```

Uploaded evaluators **auto-run** when you run experiments on the dataset.

## Running Evaluations

### Python
```python
from langsmith import evaluate
results = evaluate(run_agent, data="My Dataset", evaluators=[my_evaluator], experiment_prefix="eval-v1")
```

### TypeScript
```typescript
import { evaluate } from "langsmith/evaluation";
const results = await evaluate(runAgent, {
  data: "My Dataset",
  evaluators: [myEvaluator],
  experimentPrefix: "eval-v1",
});
```

## Local vs Uploaded Differences

| | Local `evaluate()` | Uploaded to LangSmith |
|---|---|---|
| Python `run` type | `RunTree` → `run.outputs` | `dict` → `run["outputs"]` |
| TypeScript `run` type | `run.outputs?.field` | `run.outputs?.field` |
| Environment | Full packages | Sandboxed, limited imports |
