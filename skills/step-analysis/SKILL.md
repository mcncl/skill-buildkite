---
name: step-analysis
description: Analyzes why Buildkite pipeline steps were skipped, didn't run, or behaved unexpectedly. Use when a user asks why a step didn't execute, was skipped, or wants to understand step dependencies.
---

# Step Analysis

You are helping a user understand why a pipeline step didn't run as expected, was skipped, or behaved unexpectedly.

## Approach

1. **Get Build Information**
   - Use `buildkite_get_build` to fetch the build with all job details
   - Look at each job's `state` field to understand what happened
   - Check for `skipped`, `blocked`, `waiting`, `waiting_failed` states

2. **Analyze Step States**

   | State | Meaning |
   |-------|---------|
   | `scheduled` | Waiting to be assigned to an agent |
   | `assigned` | Assigned to agent, waiting to start |
   | `accepted` | Agent accepted, starting soon |
   | `running` | Currently executing |
   | `passed` | Completed successfully |
   | `failed` | Command returned non-zero exit |
   | `blocked` | Waiting for manual unblock |
   | `canceled` | User or system cancelled |
   | `canceling` | In process of cancelling |
   | `skipped` | Skipped due to condition |
   | `not_run` | Dependency failed, step not run |
   | `waiting` | Waiting for dependency |
   | `waiting_failed` | Waiting but will not run |
   | `timed_out` | Exceeded timeout |

3. **Check Pipeline Configuration**
   - If the user has a `pipeline.yml`, read it to understand the intended flow
   - Look for these step-control fields:
     - `if`: Conditional expression that must evaluate to true
     - `depends_on`: Steps that must complete first
     - `allow_dependency_failure`: Whether to run if dependency failed
     - `branches`: Branch filter patterns
     - `skip`: Explicit skip condition

4. **Investigate Specific Scenarios**

### Step Skipped
- Check the `if` condition - what variables were evaluated?
- Look at branch filters - does this branch match?
- Check if `skip: true` or a skip message was set
- Review build metadata that might affect conditions

### Step Didn't Run (not_run state)
- Identify which dependency failed using `depends_on`
- Check if `allow_dependency_failure: true` was set
- Look at the dependency chain - find the root failure

### Step Stuck in Waiting
- Check what it's waiting for: dependency, agent, or block
- If waiting for agents: check queue and agent tags
- If blocked: check who can unblock and current state

### Step Timed Out
- Check `timeout_in_minutes` setting
- Look at logs to see what was happening when timeout hit
- Consider if the step is legitimately slow or stuck

## Understanding Conditionals

Buildkite `if` conditions can reference:
- `build.branch` - Branch name
- `build.tag` - Tag name (if tag build)
- `build.message` - Commit message
- `build.source` - What triggered the build (webhook, ui, api, schedule)
- `build.env.*` - Environment variables
- `build.meta_data.*` - Build metadata
- `build.pull_request.*` - PR information

Example conditions:
```yaml
# Only on main
if: build.branch == "main"

# Skip for PRs from forks
if: build.pull_request.repository.fork != true

# Only when specific files changed
if: build.env.BUILDKITE_PLUGINS =~ /docker/
```

## Response Format

1. **What Happened**: Clear explanation of the step's state
2. **Why**: The specific condition or dependency that caused this
3. **Evidence**: Point to the exact configuration or build data
4. **Resolution**: How to change behavior if desired

## Important Notes

- Always fetch the actual build data - don't guess about step states
- If the pipeline YAML is available, cross-reference with actual execution
- Consider that dynamic pipelines may modify steps at runtime
- Group steps can affect how dependencies are interpreted
