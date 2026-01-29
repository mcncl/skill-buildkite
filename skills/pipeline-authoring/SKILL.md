---
name: pipeline-authoring
description: |
  Helps write and improve Buildkite pipeline configurations. Use when user asks:
  - "Help me write a pipeline"
  - "Create a CI pipeline for X"
  - "Add a step to my pipeline"
  - "How do I do X in Buildkite?"
  - "Optimize my pipeline"
  - "Add matrix builds"
  - "Set up parallel testing"
---

# Pipeline Authoring

Help users write and improve Buildkite pipeline configurations.

## Available MCP Tools

| Tool | Purpose |
|------|---------|
| `buildkite_get_pipeline` | Get existing pipeline configuration |

Also use file reading to examine existing `pipeline.yml` or `.buildkite/` files in the user's codebase.

## Approach

1. **Understand the goal**
   - What workflow are they building?
   - What language/framework?
   - What's the deployment target?

2. **Check existing configuration**
   - Look for `.buildkite/pipeline.yml` or `pipeline.yml`
   - Use `buildkite_get_pipeline` if they have an existing pipeline

3. **Provide appropriate help**
   - New pipeline: scaffold complete config
   - Modification: suggest specific changes
   - Question: explain with examples

4. **Validate the result**
   - Check dependencies make sense
   - Verify conditions are correct
   - Consider edge cases

## Pipeline Structure

```yaml
# Environment for all steps
env:
  NODE_ENV: "test"

# Default agent configuration
agents:
  queue: "default"

# Pipeline steps
steps:
  - command: "npm test"
    label: ":npm: Tests"
```

## Step Types

### Command Step
```yaml
- label: ":test_tube: Tests"
  command: "npm test"
  # Multiple commands
  commands:
    - "npm ci"
    - "npm test"
  # Artifacts to upload
  artifact_paths:
    - "coverage/**/*"
  # Step-specific env
  env:
    CI: "true"
  # Timeout
  timeout_in_minutes: 10
  # Retry configuration
  retry:
    automatic:
      - exit_status: -1
        limit: 2
```

### Wait Step
```yaml
- wait

# Continue even if previous failed
- wait:
    continue_on_failure: true
```

### Block Step
```yaml
- block: ":rocket: Deploy"
  prompt: "Ready to deploy?"
  branches: "main"
  fields:
    - text: "Reason"
      key: "reason"
      required: true
    - select: "Environment"
      key: "env"
      options:
        - label: "Staging"
          value: "staging"
        - label: "Production"
          value: "production"
```

### Trigger Step
```yaml
- trigger: "deploy-pipeline"
  label: ":rocket: Deploy"
  build:
    branch: "${BUILDKITE_BRANCH}"
    commit: "${BUILDKITE_COMMIT}"
    env:
      DEPLOY_ENV: "production"
```

### Group Step
```yaml
- group: ":test_tube: Tests"
  steps:
    - command: "npm run test:unit"
      label: "Unit"
    - command: "npm run test:e2e"
      label: "E2E"
```

## Dependencies

### Explicit with Keys
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

### Allow Dependency Failure
```yaml
- command: "npm run report"
  depends_on:
    - step: "test"
      allow_failure: true
```

## Conditionals

### Branch Filtering
```yaml
- command: "deploy.sh"
  branches: "main"

# Multiple branches
- command: "deploy.sh"
  branches:
    - "main"
    - "release/*"
```

### If Conditions
```yaml
# Only on main
- command: "deploy.sh"
  if: build.branch == "main"

# Only for tags
- command: "release.sh"
  if: build.tag =~ /^v[0-9]+/

# Skip draft PRs
- command: "full-tests.sh"
  if: build.pull_request.draft != true

# Based on metadata
- command: "deploy.sh"
  if: build.meta_data.deploy == "true"
```

## Parallelism and Matrix

### Parallel Jobs
```yaml
- command: "npm test"
  parallelism: 4
  # Uses BUILDKITE_PARALLEL_JOB (0-3)
  # and BUILDKITE_PARALLEL_JOB_COUNT (4)
```

### Matrix Builds
```yaml
- command: "test.sh"
  matrix:
    setup:
      node: ["16", "18", "20"]
      os: ["linux", "macos"]
  # Creates 6 jobs (all combinations)
  agents:
    os: "{{matrix.os}}"
  env:
    NODE_VERSION: "{{matrix.node}}"
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
        soft_fail: true
```

## Best Practices

### Use Keys for Dependencies
```yaml
# Good - explicit
- command: "build"
  key: "build"
- command: "test"
  depends_on: "build"

# Avoid - implicit ordering is fragile
```

### Soft Fail for Non-Critical
```yaml
- command: "npm run lint"
  soft_fail: true

# Or specific exit codes
- command: "npm audit"
  soft_fail:
    - exit_status: 1
```

### Retry Flaky Operations
```yaml
- command: "npm test"
  retry:
    automatic:
      - exit_status: -1  # Agent lost
        limit: 2
      - exit_status: 255 # Network
        limit: 2
```

### Always Set Timeouts
```yaml
- command: "npm test"
  timeout_in_minutes: 15
```

### Cache Dependencies
```yaml
- command: "npm test"
  plugins:
    - cache#v1.0.0:
        paths:
          - "node_modules"
        key: "npm-{{ checksum 'package-lock.json' }}"
```

## Common Patterns

### Basic CI
```yaml
steps:
  - label: ":npm: Install"
    command: "npm ci"
    key: "install"

  - label: ":eslint: Lint"
    command: "npm run lint"
    depends_on: "install"
    soft_fail: true

  - label: ":jest: Test"
    command: "npm test"
    depends_on: "install"

  - label: ":package: Build"
    command: "npm run build"
    depends_on:
      - "install"
      - "test"
```

### Deploy Pipeline
```yaml
steps:
  - label: ":test_tube: Test"
    command: "npm test"
    key: "test"

  - wait

  - label: ":rocket: Deploy Staging"
    command: "deploy.sh staging"
    key: "staging"
    branches: "main"

  - block: ":rocket: Deploy Production?"
    branches: "main"

  - label: ":rocket: Deploy Production"
    command: "deploy.sh production"
    branches: "main"
```

### Monorepo
```yaml
steps:
  - label: ":mag: Detect Changes"
    command: "scripts/detect-changes.sh"
    key: "detect"

  - label: ":package: Build API"
    command: "npm run build -w api"
    if: build.env.API_CHANGED == "true"
    depends_on: "detect"

  - label: ":package: Build Web"
    command: "npm run build -w web"
    if: build.env.WEB_CHANGED == "true"
    depends_on: "detect"
```

## Response Format

When helping with pipelines:

1. **Understand** what they're trying to achieve
2. **Show** the YAML configuration
3. **Explain** key parts and why they're configured that way
4. **Validate** the logic makes sense
5. **Suggest** improvements if applicable

Always provide complete, copy-pasteable YAML.

## Example Interaction

```text
User: Help me set up CI for my Node.js project

1. Ask about test framework, deploy targets, any special needs
2. Provide scaffold:

​```yaml
steps:
  - label: ":npm: Install"
    command: "npm ci"
    key: "install"

  - label: ":jest: Test"
    command: "npm test"
    depends_on: "install"
    artifact_paths:
      - "coverage/**/*"

  - label: ":package: Build"
    command: "npm run build"
    depends_on: "install"
    artifact_paths:
      - "dist/**/*"
​```

3. Explain the structure and ask if they need additions
```
