---
name: build-debugging
description: |
  Analyzes failed Buildkite builds to identify root causes. Use when user asks:
  - "Why did build X fail?"
  - "Debug this build"
  - "What's wrong with my CI?"
  - "Fix this build failure"
  - "Help me understand this error"
  - /buildkite:debug
---

# Build Debugging

Analyze failed Buildkite builds to identify root causes and provide actionable fixes.

## Available MCP Tools

| Tool | Purpose |
|------|---------|
| `buildkite_get_build` | Fetch build details including all jobs and their states |
| `buildkite_read_logs` | Get full log output for a specific job |
| `buildkite_search_logs` | Search for patterns within job logs |
| `buildkite_get_build_test_engine_runs` | Get Test Engine results for the build |
| `buildkite_get_failed_test_executions` | Get details of failed tests |
| `buildkite_list_artifacts_for_build` | List uploaded artifacts |
| `buildkite_get_artifact` | Download a specific artifact |

## Input Parsing

Parse build information from `$ARGUMENTS` or the user's message:

| Input Format | Example |
|--------------|---------|
| Full URL | `https://buildkite.com/org/pipeline/builds/123` |
| Build number | `123` |
| Pipeline + build | `my-pipeline#123` or `my-pipeline 123` |
| Description | "the latest failed build on main" |

If no build specified, ask the user which build to debug.

## Approach

1. **Fetch the build** with `buildkite_get_build`
   - Note the overall state, branch, commit, and message
   - Check if this is a retry or first attempt

2. **Identify failed jobs** in the jobs array
   - Look for `state: "failed"` or `state: "timed_out"`
   - Note job names/labels to understand what failed
   - Check job exit codes

3. **Read logs** with `buildkite_read_logs` for failed jobs
   - Focus on the last 50-100 lines where failures surface
   - Look for the FIRST error, not just the last (cascading failures are common)

4. **Check test results** if applicable
   - Use `buildkite_get_build_test_engine_runs` for Test Engine data
   - Use `buildkite_get_failed_test_executions` for failure details

5. **Review artifacts** for additional context
   - Test reports, coverage data, debug outputs

## Common Failure Patterns

### Exit Codes
| Code | Meaning | Action |
|------|---------|--------|
| 1 | General error | Check command output |
| 127 | Command not found | Missing dependency or PATH issue |
| 137 | OOM killed (128+9) | Increase memory or optimize |
| 143 | SIGTERM (128+15) | Timeout or cancelled |

### Test Failures
- **Flaky tests**: Check if same test passed on retry
- **Environment differences**: Compare agent tags, env vars
- **Timing issues**: Race conditions or async problems

### Infrastructure Issues
- **Agent disconnected**: Network or agent health
- **Timeout**: Job exceeded `timeout_in_minutes`
- **No agents**: Check queue and agent tags

## Response Format

1. **Summary**: One-line description of what failed
2. **Root Cause**: What actually caused the failure
3. **Evidence**: Relevant log snippets (use code blocks)
4. **Recommendation**: Specific steps to fix
5. **Prevention**: How to avoid this in future (if applicable)

## Example Interaction

```text
User: Why did build 456 fail?

1. Fetch build 456 with buildkite_get_build
2. Find failed job: "Run Tests" with exit code 1
3. Read logs, find: "Error: Cannot find module 'lodash'"
4. Respond with root cause (missing dependency) and fix (add to package.json)
```
