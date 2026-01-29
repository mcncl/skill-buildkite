---
name: build-unblocking
description: "Unblocks Buildkite builds waiting for manual approval."
---

# Build Unblocking

Unblock builds that are waiting for manual approval.

## When to use

- "Unblock the build"
- "Approve the deployment"
- "Continue the pipeline"
- "Unblock build X"
- "Approve the staging deploy"
- /buildkite:unblock

## Available MCP Tools

| Tool | Purpose |
|------|---------|
| `get_build` | Get build details and find blocked jobs |
| `unblock_job` | Unblock a specific job |
| `list_builds` | Find builds with blocked steps |

## Input Parsing

Parse from `$ARGUMENTS` or user's message:

| Input Format | Example |
|--------------|---------|
| Build URL | `https://buildkite.com/org/pipe/builds/123` |
| Build number | `123` |
| Description | "the blocked deploy" |
| Pipeline reference | "unblock the staging pipeline" |

## Approach

1. **Find the blocked job**
   - Fetch build with `buildkite_get_build`
   - Look for jobs with `state: "blocked"`
   - If multiple blocked jobs, ask which one

2. **Show block details**
   - What step is blocked (name/label)
   - What the block step says (prompt message)
   - Any required fields

3. **Collect required fields** if any
   - Block steps can require text input or selections
   - Ask user for each required field
   - Validate input format

4. **Confirm before unblocking**
   - Show what will be unblocked
   - Extra confirmation for deploy gates

5. **Unblock** with `buildkite_unblock_job`

6. **Report status**
   - Confirm unblocked
   - Show what runs next

## Block Step Fields

Block steps may require input:

```yaml
- block: "Deploy to Production"
  fields:
    - text: "Reason for deployment"
      key: "reason"
      required: true
    - select: "Target environment"
      key: "environment"
      options:
        - label: "Staging"
          value: "staging"
        - label: "Production"
          value: "production"
```

When fields are required, collect them:

```text
This block step requires input:

1. **Reason for deployment** (required):
   > [waiting for user input]

2. **Target environment**:
   - Staging
   - Production
```

## Safety Requirements

### Standard Block
```text
I'll unblock **"Deploy to Staging"** in build #456.

This will continue the pipeline and run the deployment steps.

Proceed? (yes/no)
```

### Production/Deploy Gates
```text
⚠️ This is a production deployment gate.

Unblocking **"Deploy to Production"** in build #456 will:
- Deploy commit abc123 to production
- This cannot be easily undone

Type "deploy" to confirm.
```

## Response Format

After unblocking:

```text
✅ Build unblocked!

**Build #456**: Continuing...
- Unblocked step: "Deploy to Staging"
- Next steps: deploy, smoke-tests, notify

The pipeline is now continuing.
```

## Finding Blocked Builds

If user doesn't specify a build:

1. Use `buildkite_list_builds` filtered to blocked state
2. Show recent blocked builds:

```text
Found 2 blocked builds:

1. **#456** - my-pipeline (main)
   Blocked at: "Deploy to Production"
   Waiting since: 2 hours ago

2. **#453** - deploy-service (release/v2)
   Blocked at: "Manual QA Approval"
   Waiting since: 1 day ago

Which build would you like to unblock?
```

## Example Interaction

```text
User: Approve the staging deployment

1. Find recent builds with blocked state
2. Identify block step matching "staging"
3. Show block details and any required fields
4. Confirm with user
5. Unblock and report status
```
