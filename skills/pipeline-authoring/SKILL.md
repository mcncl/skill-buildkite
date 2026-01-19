---
name: pipeline-authoring
description: Helps write and improve Buildkite pipeline configurations. Use when creating new pipelines, modifying existing ones, or optimizing pipeline structure.
---

# Pipeline Authoring

You are helping write or improve Buildkite pipeline configurations.

## Pipeline Structure

```yaml
# Environment variables for all steps
env:
  NODE_ENV: "test"

# Default agent requirements
agents:
  queue: "default"

# Pipeline steps
steps:
  - command: "npm test"
    label: ":npm: Tests"
```

## Step Types

### Command Step
Runs commands on an agent.

```yaml
- label: ":test_tube: Run Tests"
  command: "npm test"
  # Or multiple commands
  commands:
    - "npm ci"
    - "npm test"
  artifact_paths:
    - "coverage/**/*"
    - "test-results/*.xml"
  env:
    CI: "true"
  timeout_in_minutes: 10
  retry:
    automatic:
      - exit_status: -1  # Agent lost
        limit: 2
      - exit_status: "*"
        limit: 1
```

### Wait Step
Synchronization point between stages.

```yaml
- wait

# Or continue even if previous steps failed
- wait:
    continue_on_failure: true
```

### Block Step
Manual approval gate.

```yaml
- block: ":rocket: Deploy to Production"
  prompt: "Ready to deploy?"
  branches: "main"
  fields:
    - text: "Reason"
      key: "deploy-reason"
      required: true
    - select: "Environment"
      key: "deploy-env"
      options:
        - label: "Staging"
          value: "staging"
        - label: "Production"
          value: "production"
```

### Input Step
Collect input without blocking.

```yaml
- input: "Release Configuration"
  fields:
    - text: "Version"
      key: "version"
      default: "1.0.0"
```

### Trigger Step
Start another pipeline.

```yaml
- trigger: "deploy-pipeline"
  label: ":rocket: Trigger Deploy"
  build:
    branch: "${BUILDKITE_BRANCH}"
    commit: "${BUILDKITE_COMMIT}"
    message: "Deploy: ${BUILDKITE_MESSAGE}"
    env:
      DEPLOY_ENV: "production"
```

### Group Step
Organize related steps.

```yaml
- group: ":test_tube: Tests"
  steps:
    - command: "npm run test:unit"
      label: "Unit Tests"
    - command: "npm run test:integration"
      label: "Integration Tests"
```

## Dependencies and Conditions

### Explicit Dependencies
```yaml
steps:
  - command: "npm ci"
    key: "install"

  - command: "npm test"
    key: "test"
    depends_on: "install"

  - command: "npm run build"
    depends_on:
      - "install"
      - "test"
```

### Allow Dependency Failures
```yaml
- command: "npm run report"
  depends_on:
    - step: "test"
      allow_failure: true
```

### Conditional Steps
```yaml
# Only on main branch
- command: "deploy.sh"
  if: build.branch == "main"

# Only for tags
- command: "release.sh"
  if: build.tag =~ /^v[0-9]+/

# Skip for draft PRs
- command: "full-test-suite.sh"
  if: build.pull_request.draft != true

# Based on changed files (requires diff plugin or metadata)
- command: "build-docs.sh"
  if: build.env.DOCS_CHANGED == "true"
```

### Branch Filtering
```yaml
- command: "deploy.sh"
  branches: "main"

# Multiple branches
- command: "deploy-staging.sh"
  branches:
    - "main"
    - "develop"
    - "release/*"
```

## Parallelism and Matrix

### Parallel Jobs
```yaml
- command: "npm test"
  parallelism: 4
  # Uses BUILDKITE_PARALLEL_JOB (0-3) and BUILDKITE_PARALLEL_JOB_COUNT
```

### Matrix Builds
```yaml
- command: "test.sh"
  matrix:
    setup:
      node: ["16", "18", "20"]
      os: ["linux", "macos"]
    # Creates 6 jobs: all combinations
```

### Matrix with Adjustments
```yaml
- command: "test.sh"
  matrix:
    setup:
      node: ["16", "18", "20"]
    adjustments:
      - with:
          node: "20"
        soft_fail: true  # Node 20 failures are non-blocking
```

## Best Practices

### Use Keys for Dependencies
```yaml
# Good - explicit dependencies
- command: "build"
  key: "build"
- command: "test"
  depends_on: "build"

# Avoid - implicit ordering is fragile
```

### Fail Fast with Soft Fails
```yaml
# Non-critical steps shouldn't block
- command: "npm run lint"
  soft_fail: true

# Or specific exit codes
- command: "npm run lint"
  soft_fail:
    - exit_status: 1
```

### Use Retry for Flaky Operations
```yaml
- command: "npm test"
  retry:
    automatic:
      - exit_status: -1  # Agent issues
        limit: 2
      - exit_status: 255  # SSH/network issues
        limit: 2
```

### Cache Dependencies
```yaml
- command: "npm test"
  cache:
    paths:
      - "node_modules"
    restore_keys:
      - "v1-npm-{{ checksum 'package-lock.json' }}"
      - "v1-npm-"
    save_key: "v1-npm-{{ checksum 'package-lock.json' }}"
```

### Use Timeouts
```yaml
# Prevent stuck jobs
- command: "npm test"
  timeout_in_minutes: 15
```

### Notifications
```yaml
notify:
  - slack:
      channels:
        - "#builds"
      conditions:
        - branch: "main"
        - state: "failed"
```

## When Helping Users

1. **Understand their goal** - What workflow are they trying to achieve?
2. **Check existing pipelines** - Use `buildkite_get_pipeline` if they have one
3. **Suggest improvements** - Parallelism, caching, better error handling
4. **Validate logic** - Ensure dependencies and conditions make sense
5. **Consider edge cases** - What happens on failure? On specific branches?

## Important Notes

- YAML indentation matters - use consistent 2-space indentation
- Environment variables in strings need `${VAR}` syntax
- Boolean conditions use Buildkite's expression syntax, not YAML booleans
- Test complex pipelines with `buildkite-agent pipeline upload --dry-run`
