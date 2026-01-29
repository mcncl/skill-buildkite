---
name: step-analysis
description: |
  Analyzes why Buildkite pipeline steps behaved unexpectedly. Use when user asks:
  - "Why was this step skipped?"
  - "Why didn't the deploy run?"
  - "Why is this step waiting?"
  - "What's blocking this step?"
  - "Why did this step not run?"
  - "Explain the step dependencies"
---

# Step Analysis

Analyze why pipeline steps were skipped, didn't run, or behaved unexpectedly.

## Available MCP Tools

| Tool | Purpose |
|------|---------|
| `buildkite_get_build` | Get build with all job states |
| `buildkite_get_pipeline` | Get pipeline YAML configuration |
| `buildkite_read_logs` | Check logs for clues about conditions |

## Input Parsing

User typically asks about a specific step:

| Input Format | Example |
|--------------|---------|
| Step name | "the deploy step" |
| Build + step | "build 123, the test job" |
| Description | "why didn't prod deploy run?" |

## Approach

1. **Get build information**
   - Fetch with `buildkite_get_build`
   - Find the step in question by name/label
   - Note its current state

2. **Analyze the state**
   - Check state field on the job
   - Cross-reference with pipeline config if available

3. **Investigate the cause**
   - For skipped: check `if` conditions
   - For not_run: find failed dependency
   - For waiting: identify what it's waiting on

4. **Explain clearly**
   - What happened
   - Why it happened
   - How to change the behavior if desired

## Job States Reference

| State | Meaning | Common Cause |
|-------|---------|--------------|
| `scheduled` | Waiting for agent | No matching agent available |
| `assigned` | Agent accepted | Starting soon |
| `running` | Executing | In progress |
| `passed` | Completed successfully | - |
| `failed` | Non-zero exit | Command error |
| `blocked` | Manual approval needed | Block step |
| `canceled` | User cancelled | - |
| `skipped` | Condition false | `if` evaluated to false |
| `not_run` | Dependency failed | `depends_on` step failed |
| `waiting` | Waiting for dependency | Upstream not complete |
| `waiting_failed` | Will not run | Upstream failed |
| `timed_out` | Exceeded timeout | `timeout_in_minutes` hit |

## Conditional Evaluation

### Available Variables in `if` Conditions

```yaml
build.branch          # "main", "feature/x"
build.tag             # "v1.0.0" or null
build.message         # Commit message
build.source          # "webhook", "ui", "api", "schedule"
build.state           # Current build state
build.pull_request.id # PR number or null
build.pull_request.base_branch
build.pull_request.repository.fork  # true/false
build.env.VARNAME     # Environment variable
build.meta_data.key   # Build metadata
```

### Common Conditions

```yaml
# Only on main
if: build.branch == "main"

# Only for tags
if: build.tag != null

# Skip for fork PRs
if: build.pull_request.repository.fork != true

# Only from webhook (not manual)
if: build.source == "webhook"

# Based on env var
if: build.env.RUN_DEPLOY == "true"
```

## Dependency Analysis

### Simple Dependency
```yaml
steps:
  - command: "test"
    key: "test"
  - command: "deploy"
    depends_on: "test"  # Won't run if test fails
```

### Allow Failure
```yaml
- command: "deploy"
  depends_on:
    - step: "test"
      allow_failure: true  # Runs even if test fails
```

### Implicit Dependencies
- Steps after `wait` depend on all previous steps
- Group steps depend on all steps within the group

## Response Format

```
## Why "Deploy to Production" was skipped

**State**: skipped

**Reason**: The step has an `if` condition that evaluated to false.

**Condition**: `build.branch == "main"`
**Actual value**: `build.branch` = "feature/new-login"

**Resolution**: This step only runs on the `main` branch. To deploy this
branch, either:
1. Merge to main and let CI run
2. Modify the condition to include this branch
3. Manually trigger a build with the condition overridden
```

## Common Scenarios

### Step Skipped
1. Check for `if` condition in pipeline YAML
2. Determine what values were evaluated
3. Explain why condition was false

### Step Didn't Run (not_run)
1. Find the `depends_on` configuration
2. Identify which dependency failed
3. Trace back to root cause

### Step Stuck Waiting
1. Check if waiting for dependency or agent
2. For dependencies: show which step it's waiting on
3. For agents: suggest agent-troubleshooting skill

### Step Timed Out
1. Check `timeout_in_minutes` setting
2. Look at logs to see what was happening
3. Suggest increasing timeout or optimizing step

## Example Interaction

```
User: Why didn't the deploy step run in build 456?

1. Fetch build 456
2. Find "deploy" step - state is "not_run"
3. Check depends_on - depends on "test"
4. Find "test" step - state is "failed"
5. Explain: "Deploy didn't run because the test step it depends on failed.
   Fix the test failure and the deploy will run on the next build."
```
