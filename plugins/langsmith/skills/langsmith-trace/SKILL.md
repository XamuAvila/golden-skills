---
name: langsmith-trace
description: "Use when working with LangSmith tracing OR querying traces. Covers adding tracing to applications and querying/exporting trace data. Uses the langsmith CLI tool."
---

# langsmith-trace

Two main topics: **adding tracing** to your application, and **querying traces** for debugging and analysis. Python and Javascript implementations are both supported.

## Setup

Environment Variables

```bash
LANGSMITH_API_KEY=lsv2_pt_your_api_key_here          # Required
LANGSMITH_PROJECT=your-project-name                   # Optional: default project
LANGSMITH_WORKSPACE_ID=your-workspace-id              # Optional: for org-scoped keys
```

**IMPORTANT:** Always check the environment variables or `.env` file for `LANGSMITH_PROJECT` before querying or interacting with LangSmith.

CLI Tool
```bash
curl -sSL https://raw.githubusercontent.com/langchain-ai/langsmith-cli/main/scripts/install.sh | sh
```

## Tracing LangChain/LangGraph Apps

Tracing is automatic. Just set environment variables:

```bash
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY=<your-api-key>
```

Optional variables:
- `LANGSMITH_PROJECT` - specify project name (defaults to "default")
- `LANGCHAIN_CALLBACKS_BACKGROUND=false` - use for serverless

## Tracing Other Frameworks

For non-LangChain apps with native OpenTelemetry support, use LangSmith's OpenTelemetry integration.

Otherwise, use the traceable decorator/wrapper and wrap your LLM client.

### Python
```python
from langsmith import traceable
from langsmith.wrappers import wrap_openai
from openai import OpenAI

client = wrap_openai(OpenAI())

@traceable
def my_llm_pipeline(question: str) -> str:
    resp = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": question}],
    )
    return resp.choices[0].message.content
```

### TypeScript
```typescript
import { traceable } from "langsmith/traceable";
import { wrapOpenAI } from "langsmith/wrappers";
import OpenAI from "openai";

const client = wrapOpenAI(new OpenAI());

const myLlmPipeline = traceable(async (question: string): Promise<string> => {
  const resp = await client.chat.completions.create({
    model: "gpt-4o-mini",
    messages: [{ role: "user", content: question }],
  });
  return resp.choices[0].message.content || "";
}, { name: "my_llm_pipeline" });
```

Best Practices:
- Apply traceable to all nested functions you want visible in LangSmith
- Wrapped clients auto-trace all calls
- Name your traces for easier filtering
- Add metadata for searchability

## Traces vs Runs

- **Trace** = A complete execution tree (root run + all child runs)
- **Run** = A single node in the tree (one LLM call, one tool call, etc.)

**Generally, query traces first** — they provide complete context.

## CLI Command Structure

```
langsmith
├── trace
│   ├── list    - List traces (filters apply to root run)
│   ├── get     - Get single trace with full hierarchy
│   └── export  - Export traces to JSONL files
├── run
│   ├── list    - List runs (flat, filters apply to any run)
│   ├── get     - Get single run
│   └── export  - Export runs to single JSONL file
```

## Querying Traces

```bash
langsmith trace list --limit 10 --project my-project
langsmith trace list --limit 10 --include-metadata
langsmith trace list --last-n-minutes 60
langsmith trace get <trace-id>
langsmith trace list --limit 5 --show-hierarchy
langsmith trace export ./traces --limit 20 --full
langsmith trace list --min-latency 5.0 --limit 10
langsmith trace list --error --last-n-minutes 60
langsmith run list --run-type llm --limit 20
```

## Filters

- `--trace-ids abc,def` - Filter to specific traces
- `--limit N` - Max results
- `--project NAME` - Project name
- `--last-n-minutes N` - Time filter
- `--since TIMESTAMP` - Time filter (ISO format)
- `--error / --no-error` - Error status
- `--name PATTERN` - Name contains
- `--min-latency SECONDS` - Minimum latency
- `--max-latency SECONDS` - Maximum latency
- `--min-tokens N` - Minimum total tokens
- `--tags tag1,tag2` - Has any of these tags
- `--filter QUERY` - Raw LangSmith filter query

## Tips

- Start with traces for complete context
- Use `traces export --full` for bulk data
- Always specify `--project`
- Use `/tmp` for temporary exports
- Include `--include-metadata` for performance/cost analysis
