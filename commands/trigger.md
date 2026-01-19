---
description: Trigger a new Buildkite build for a pipeline
---

# Trigger Build

The user wants to trigger a new Buildkite build. Parse "$ARGUMENTS" for pipeline and options.

## Approach

1. **Identify the pipeline**
   - Parse pipeline name/slug from arguments
   - If not provided, ask which pipeline to trigger

2. **Determine build parameters**
   - Branch: Default to current git branch if in a repo, or ask
   - Commit: Default to HEAD or latest
   - Message: Optional custom message
   - Environment: Any custom env vars needed

3. **Create the build** using `buildkite_create_build`
   - Confirm with the user before triggering (especially for production pipelines)
   - Show what will be triggered

4. **Report the result**
   - Provide the build URL
   - Show initial status
   - Offer to watch for completion using `buildkite_wait_for_build`

## Examples

- `/buildkite:trigger my-pipeline` - Trigger build on current branch
- `/buildkite:trigger my-pipeline main` - Trigger build on main
- `/buildkite:trigger deploy-pipeline --env DEPLOY_ENV=staging` - Trigger with env var

## Safety

- Ask for confirmation before triggering builds
- Be extra careful with pipelines that have "deploy" or "production" in the name
- Show what branch/commit will be built before triggering
