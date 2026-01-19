---
description: Debug a failed Buildkite build - analyzes logs, tests, and artifacts to find the root cause
---

# Debug Build

The user wants to debug a Buildkite build. If they provided a build URL or number, use that. Otherwise, ask what build they want to debug.

## If build info provided

Parse the build information from "$ARGUMENTS". This could be:
- A full URL like `https://buildkite.com/org/pipeline/builds/123`
- Just a build number like `123`
- A pipeline and build like `my-pipeline#123`

Use the appropriate Buildkite MCP tools to:

1. **Fetch the build** using `buildkite_get_build`
2. **Identify failures** - look for jobs with failed/timed_out state
3. **Get logs** using `buildkite_read_logs` for failed jobs
4. **Check tests** using `buildkite_get_build_test_engine_runs` and `buildkite_get_failed_test_executions`
5. **Review artifacts** if relevant using `buildkite_list_artifacts_for_build`

Provide a clear analysis:
- What failed and why
- Relevant log excerpts
- Specific recommendations to fix

## If no build info provided

Ask the user which build they want to debug. They can provide:
- A build URL from Buildkite
- An organization, pipeline, and build number
- Or describe what they're looking for (e.g., "the latest failed build on main")
