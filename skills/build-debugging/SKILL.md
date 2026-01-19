---
name: build-debugging
description: Analyzes failed Buildkite builds to identify root causes. Use when a user asks why a build failed, wants to understand build errors, or needs help fixing CI failures.
---

# Build Debugging

You are helping debug a failed Buildkite build. Your goal is to identify the root cause and provide actionable guidance.

## Approach

1. **Get Build Context**
   - Use `buildkite_get_build` to fetch the build details
   - Note the build state, branch, commit, and any metadata
   - Check if this is a retry or first attempt

2. **Identify Failed Jobs**
   - Look at the jobs array in the build response
   - Find jobs with `state: "failed"` or `state: "timed_out"`
   - Note the job names/labels to understand what failed

3. **Analyze Logs**
   - Use `buildkite_read_logs` to get the full log output for failed jobs
   - Look for error messages, stack traces, or failure indicators
   - Pay attention to the last 50-100 lines where failures typically surface

4. **Check Test Results**
   - Use `buildkite_get_build_test_engine_runs` to see if Test Engine captured results
   - If tests failed, use `buildkite_get_failed_test_executions` for details
   - Look for patterns: flaky tests, environment issues, code changes

5. **Review Artifacts**
   - Use `buildkite_list_artifacts_for_build` to see what artifacts were uploaded
   - Look for test reports, coverage data, or debug outputs
   - Use `buildkite_get_artifact` to download relevant files for analysis

## Common Failure Patterns

### Exit Code Failures
- Exit code 1: General error - check command output
- Exit code 127: Command not found - missing dependency or PATH issue
- Exit code 137: OOM killed (128 + 9) - increase memory or optimize
- Exit code 143: SIGTERM (128 + 15) - timeout or cancelled

### Test Failures
- Flaky tests: Check if same test passed on retry
- Environment differences: Compare agent tags, environment variables
- Timing issues: Look for race conditions or async problems

### Infrastructure Issues
- Agent disconnected: Network or agent health problem
- Timeout: Job exceeded `timeout_in_minutes`
- No agents available: Check agent tags and queue configuration

## Response Format

Provide your analysis in this structure:

1. **Summary**: One-line description of what failed
2. **Root Cause**: What actually caused the failure
3. **Evidence**: Relevant log snippets or data points
4. **Recommendation**: Specific steps to fix the issue
5. **Prevention**: How to avoid this in the future (if applicable)

## Important Notes

- Always check logs before speculating about causes
- Look for the FIRST error, not just the last - cascading failures are common
- Consider recent changes to the pipeline or codebase
- If the failure seems infrastructure-related, suggest checking Buildkite status
