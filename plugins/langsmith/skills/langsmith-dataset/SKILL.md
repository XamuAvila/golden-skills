---
name: langsmith-dataset
description: "Use when creating evaluation datasets, uploading datasets to LangSmith, or managing existing datasets. Covers dataset types (final_response, single_step, trajectory, RAG), CLI management commands, SDK-based creation, and example management."
---

# langsmith-dataset

Create, manage, and upload evaluation datasets to LangSmith for testing and validation.

## Setup

```bash
LANGSMITH_API_KEY=lsv2_pt_your_api_key_here          # Required
LANGSMITH_PROJECT=your-project-name                   # Check this for relevant traces
```

Python: `pip install langsmith`
JavaScript: `npm install langsmith`
CLI: `curl -sSL https://raw.githubusercontent.com/langchain-ai/langsmith-cli/main/scripts/install.sh | sh`

## CLI Commands

```bash
langsmith dataset list
langsmith dataset get <name-or-id>
langsmith dataset create --name <name>
langsmith dataset delete <name-or-id>
langsmith dataset export <name-or-id> <output-file>
langsmith dataset upload <file> --name <name>

langsmith example list --dataset <name>
langsmith example create --dataset <name> --inputs <json>
langsmith example delete <example-id>

langsmith experiment list --dataset <name>
langsmith experiment get <name>
```

## Dataset Types

- **final_response** - Full conversation with expected output
- **single_step** - Single node inputs/outputs
- **trajectory** - Tool call sequence (ordered list of tool names)
- **rag** - Question/chunks/answer/citations

## Creating Datasets from Traces

```bash
# 1. Export traces
langsmith trace export ./traces --project my-project --limit 20 --full
```

### Python
```python
import json
from pathlib import Path
from langsmith import Client

client = Client()
examples = []
for jsonl_file in Path("./traces").glob("*.jsonl"):
    runs = [json.loads(line) for line in jsonl_file.read_text().strip().split("\n")]
    root = next((r for r in runs if r.get("parent_run_id") is None), None)
    if root and root.get("inputs") and root.get("outputs"):
        examples.append({
            "trace_id": root.get("trace_id"),
            "inputs": root["inputs"],
            "outputs": root["outputs"]
        })

with open("/tmp/dataset.json", "w") as f:
    json.dump(examples, f, indent=2)
```

### Upload
```bash
langsmith dataset upload /tmp/dataset.json --name "My Evaluation Dataset"
```

## Dataset Structures

### Final Response
```json
{"trace_id": "...", "inputs": {"query": "..."}, "outputs": {"response": "..."}}
```

### Trajectory
```json
{"trace_id": "...", "inputs": {"query": "..."}, "outputs": {"expected_trajectory": ["tool_a", "tool_b"]}}
```

### RAG
```json
{"trace_id": "...", "inputs": {"question": "..."}, "outputs": {"answer": "...", "retrieved_chunks": ["..."], "cited_chunks": ["..."]}}
```

## SDK Direct Creation

### Python
```python
from langsmith import Client
client = Client()
dataset = client.create_dataset("My Dataset", description="Evaluation dataset")
client.create_examples(
    inputs=[{"query": "What is AI?"}],
    outputs=[{"answer": "AI is..."}],
    dataset_name="My Dataset",
)
```

### TypeScript
```typescript
import { Client } from "langsmith";
const client = new Client();
const dataset = await client.createDataset("My Dataset", { description: "Evaluation dataset" });
await client.createExamples({
  inputs: [{ query: "What is AI?" }],
  outputs: [{ answer: "AI is..." }],
  datasetName: "My Dataset",
});
```
