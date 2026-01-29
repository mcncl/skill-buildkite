---
name: build-status
description: "Shows recent Buildkite build status for a pipeline with pass/fail visibility."
---

# Build Status

Show recent build status for a pipeline with clear pass/fail visibility.

## When to use

- "Show me recent builds"
- "What's the status of the pipeline?"
- "Are builds passing?"
- "How's CI doing?"
- "List builds for X"
- "Is main green?"
- /buildkite:status

## Available MCP Tools

| Tool | Purpose |
|------|---------|
| `list_builds` | List recent builds with optional filters |
| `get_build` | Get detailed info for a specific build |
| `get_pipeline` | Get pipeline configuration and metadata |

## Input Parsing

Parse from `$ARGUMENTS` or user's message:

| Input Format | Example |
|--------------|---------|
| Pipeline name | `my-pipeline` |
| Pipeline + branch | `my-pipeline main` |
| Just branch | `main` (if pipeline can be inferred) |
| Question | "is the deploy pipeline healthy?" |

If no pipeline specified and in a git repo, try to match against Buildkite pipelines.

## Approach

1. **Identify the pipeline**
   - Parse from input or ask if unclear
   - Use `buildkite_get_pipeline` if need to verify it exists

2. **List recent builds** with `buildkite_list_builds`
   - Filter by pipeline
   - Filter by branch if specified
   - Limit to 5-10 most recent

3. **Present a clear summary**

## Output Format

Present as a scannable table:

```text
Pipeline: my-org/my-pipeline

| # | State | Branch | Commit | Age | Duration |
|---|-------|--------|--------|-----|----------|
| 456 | ✅ passed | main | abc123 | 2h ago | 4m 32s |
| 455 | ❌ failed | feature-x | def456 | 3h ago | 2m 15s |
| 454 | ✅ passed | main | 789ghi | 5h ago | 4m 28s |
| 453 | 🔄 running | develop | jkl012 | 10m ago | - |
| 452 | ⏸️ blocked | main | mno345 | 1d ago | - |
```

## State Icons

| State | Icon | Meaning |
|-------|------|---------|
| passed | ✅ | Build succeeded |
| failed | ❌ | Build failed |
| running | 🔄 | Currently running |
| blocked | ⏸️ | Waiting for approval |
| canceled | ⚪ | User cancelled |
| scheduled | 🕐 | Waiting to start |

## Highlight Issues

- **Failed builds**: Call out explicitly, offer to debug
- **Long-running builds**: Note if duration seems unusual
- **Blocked builds**: Mention they need approval
- **Patterns**: Note if recent builds are all failing

## Example Interaction

```text
User: Is main green?

1. Identify pipeline (from context or ask)
2. List recent builds filtered to main branch
3. Show status table
4. Summarize: "Main is green - last 5 builds passed"
```
