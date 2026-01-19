---
description: Check the status of recent builds for a pipeline
---

# Build Status

The user wants to check build status. Parse "$ARGUMENTS" for pipeline information.

## Approach

1. **Identify the pipeline**
   - If arguments provided, use them to identify the pipeline
   - If in a git repo, try to match against Buildkite pipelines

2. **List recent builds** using `buildkite_list_builds`
   - Filter by pipeline if specified
   - Show the most recent builds (limit to ~5-10)

3. **Present a summary** showing:
   - Build number and state (passed/failed/running/etc.)
   - Branch and commit
   - When it was created
   - Duration if completed

4. **Highlight any issues**
   - Failed builds should be called out
   - Long-running builds might need attention
   - Blocked builds waiting for approval

Format the output as a clear table or list that's easy to scan.

## Examples

- `/buildkite:status` - List recent builds (user should specify pipeline or be in relevant repo)
- `/buildkite:status my-pipeline` - Recent builds for my-pipeline
- `/buildkite:status my-pipeline main` - Recent builds for my-pipeline on main branch
