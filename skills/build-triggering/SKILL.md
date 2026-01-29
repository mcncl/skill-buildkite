---
name: build-triggering
description: |
  Triggers new Buildkite builds. Use when user asks:
  - "Trigger a build"
  - "Run CI"
  - "Start a build for X"
  - "Rebuild this"
  - "Deploy to staging"
  - "Kick off the pipeline"
  - /buildkite:trigger
---

# Build Triggering

Trigger new Buildkite builds with appropriate safety checks.

## Available MCP Tools

| Tool | Purpose |
|------|---------|
| `buildkite_get_pipeline` | Verify pipeline exists and get details |
| `buildkite_create_build` | Trigger a new build |
| `buildkite_list_builds` | Check recent builds (optional) |

## Input Parsing

Parse from `$ARGUMENTS` or user's message:

| Input Format | Example |
|--------------|---------|
| Pipeline name | `my-pipeline` |
| Pipeline + branch | `my-pipeline on main` |
| Description | "the deploy pipeline" |
| Rebuild request | "rebuild build 123" |

## Approach

1. **Identify the pipeline**
   - Parse from input
   - Use `buildkite_get_pipeline` to verify
   - If unclear, ask the user

2. **Determine parameters**
   - **Branch**: From input, current git branch, or ask
   - **Commit**: Usually HEAD/latest
   - **Message**: Optional custom message
   - **Environment**: Any custom env vars

3. **Confirm before triggering**
   - Show exactly what will be triggered
   - Require explicit confirmation
   - Be extra careful with deploy/production pipelines

4. **Create the build** with `buildkite_create_build`

5. **Report result**
   - Provide build URL
   - Show initial status

## Safety Requirements

### Always Confirm
```text
I'll trigger a build for **my-org/my-pipeline**:
- Branch: `main`
- Commit: `abc123` (latest)

Proceed? (yes/no)
```

### Extra Caution for Sensitive Pipelines

If pipeline name contains: `deploy`, `prod`, `production`, `release`

```text
⚠️ This appears to be a deployment pipeline.

I'll trigger **my-org/deploy-production**:
- Branch: `main`
- Commit: `abc123`

This may deploy to production. Please confirm by typing "deploy" to proceed.
```

### Environment Variables

If setting env vars, show them clearly:

```text
Build will include these environment variables:
- DEPLOY_ENV=staging
- SKIP_TESTS=true

Proceed?
```

## Build Parameters

| Parameter | Source | Default |
|-----------|--------|---------|
| branch | User input, git context | Ask user |
| commit | Usually "HEAD" | Latest on branch |
| message | Optional | Auto-generated |
| env | User-specified | None |

## Response Format

After triggering:

```text
✅ Build triggered successfully!

**Build #457**: https://buildkite.com/my-org/my-pipeline/builds/457
- Branch: main
- Commit: abc123
- Status: scheduled

The build is now queued and will start when an agent is available.
```

## Example Interaction

```text
User: Run CI on my branch

1. Identify pipeline (from context or ask)
2. Get current git branch
3. Confirm: "Trigger my-pipeline on feature-branch?"
4. On confirmation, create build
5. Return build URL and status
```
