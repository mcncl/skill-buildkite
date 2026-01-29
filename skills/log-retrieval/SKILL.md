---
name: log-retrieval
description: |
  Retrieves and displays job logs from Buildkite. Use when user asks:
  - "Show me the logs"
  - "What did the build output?"
  - "Get the logs from job X"
  - "Search the logs for Y"
  - "What happened in the test step?"
  - /buildkite:logs

  Note: For debugging failures, use build-debugging instead. Use this skill for viewing/searching logs without full failure analysis.
---

# Log Retrieval

Retrieve and display job logs from Buildkite builds.

## Available MCP Tools

| Tool | Purpose |
|------|---------|
| `buildkite_get_build` | Get build details to find job IDs |
| `buildkite_read_logs` | Get full log output for a job |
| `buildkite_search_logs` | Search for patterns within logs |

## Input Parsing

Parse from `$ARGUMENTS` or user's message:

| Input Format | Example |
|--------------|---------|
| Job URL | `https://buildkite.com/org/pipe/builds/123#job-uuid` |
| Build + job name | `build 123, test step` |
| Build number | `123` (then ask which job) |
| Description | "the failed job" or "the deploy step" |

## Approach

1. **Identify the job**
   - If just build given, fetch build and list jobs
   - Match job by name/label if specified
   - Default to failed jobs if user asks about errors

2. **Fetch logs** with `buildkite_read_logs`
   - Get full output for the identified job

3. **Present logs appropriately**
   - For short logs: show entirely
   - For long logs: show relevant section (usually end)
   - Always use code blocks

4. **Search if needed** with `buildkite_search_logs`
   - When user is looking for specific content
   - Return matching lines with context

## Output Format

```
## Logs: "Run Tests" (job abc-123)

​```
[timestamp] Installing dependencies...
[timestamp] Running test suite...
[timestamp] Error: Connection refused
[timestamp] Test failed: api.test.js
​```

Showing last 50 lines. Full log is 2,847 lines.
```

## Search Output

When searching:

```
## Search: "error" in build 456

Found 3 matches:

**Line 234** (test step):
> Error: Cannot connect to database

**Line 567** (deploy step):
> Error: Permission denied

**Line 890** (cleanup step):
> Error: File not found (non-fatal)
```

## When to Use This vs build-debugging

| Scenario | Skill |
|----------|-------|
| "Why did build fail?" | build-debugging |
| "Show me the test logs" | log-retrieval |
| "Debug this error" | build-debugging |
| "Search for 'timeout' in logs" | log-retrieval |
| "What went wrong?" | build-debugging |
| "What did the deploy output?" | log-retrieval |

## Example Interaction

```
User: Show me the logs from the failed test

1. Get build (from context or ask)
2. Find job with "test" in name that failed
3. Fetch logs with buildkite_read_logs
4. Display relevant portion in code block
```
