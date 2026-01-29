# Buildkite Claude Code Plugin

A [Claude Code plugin](https://code.claude.com/docs/en/plugins) for working with [Buildkite](https://buildkite.com) CI/CD pipelines. Debug failed builds, understand why steps didn't run, troubleshoot agent issues, and author pipelines.

## Features

### Skills

Claude automatically uses these capabilities when relevant. You can also invoke them directly:

| Skill | Slash Command | Description |
|-------|---------------|-------------|
| **build-debugging** | `/buildkite:debug` | Analyzes failed builds - reads logs, checks test results, identifies root causes |
| **build-status** | `/buildkite:status` | Shows recent build status for a pipeline with pass/fail state |
| **log-retrieval** | `/buildkite:logs` | Retrieves and displays job logs with search capability |
| **build-triggering** | `/buildkite:trigger` | Triggers new builds with branch/commit control |
| **build-unblocking** | `/buildkite:unblock` | Unblocks builds waiting for manual approval |
| **step-analysis** | - | Explains why steps were skipped, didn't run, or behaved unexpectedly |
| **agent-troubleshooting** | - | Diagnoses agent issues - jobs stuck waiting, queue mismatches, connectivity problems |
| **pipeline-authoring** | - | Helps write and improve pipeline configurations with best practices |

### MCP Server Integration

The plugin connects to Buildkite's [remote MCP server](https://buildkite.com/docs/apis/mcp-server) which provides tools for:

- Listing and managing pipelines
- Fetching builds, jobs, and logs
- Triggering builds and unblocking jobs
- Accessing test results and artifacts
- Managing clusters and queues

## Installation

### Prerequisites

- [Claude Code](https://claude.com/claude-code) CLI installed
- A Buildkite account

### Setup

1. Clone this repository:

   ```bash
   git clone https://github.com/mcncl/skill-buildkite.git
   ```

2. Run Claude Code with the plugin:

   ```bash
   claude --plugin-dir /path/to/buildkite-claude-plugin
   ```

3. Run `/mcp` to authenticate with Buildkite via OAuth in your browser.

### Authentication

This plugin uses Buildkite's [remote MCP server](https://buildkite.com/docs/apis/mcp-server/remote/configuring-ai-tools) with OAuth authentication. No API tokens or Docker required.

- OAuth tokens are valid for 12 hours
- Refresh tokens are valid for 7 days
- Provides read and write access based on your Buildkite permissions

## Usage Examples

### Debug a failed build

```text
> /buildkite:debug https://buildkite.com/my-org/my-pipeline/builds/123

or just ask:

> Why did build 123 fail?
```

### Check build status

```text
> /buildkite:status my-pipeline

or:

> Show me recent builds for the deploy pipeline
```

### View job logs

```text
> /buildkite:logs build-123

or:

> Show me the logs from the failed test job
```

### Trigger a build

```text
> /buildkite:trigger my-pipeline

or:

> Run CI on the main branch
```

### Unblock a deployment

```text
> /buildkite:unblock build-456

or:

> Approve the staging deployment
```

### Understand why a step was skipped

```text
> Why didn't the deploy step run in build 456?
```

### Troubleshoot agent issues

```text
> My build is stuck waiting for an agent, what's wrong?
```

### Get help writing a pipeline

```text
> Help me add a matrix build for testing on multiple Node versions
```

## Project Structure

```text
buildkite-claude-plugin/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── .mcp.json                 # MCP server configuration
├── skills/                   # AI capabilities
│   ├── build-debugging/
│   │   └── SKILL.md
│   ├── build-status/
│   │   └── SKILL.md
│   ├── log-retrieval/
│   │   └── SKILL.md
│   ├── build-triggering/
│   │   └── SKILL.md
│   ├── build-unblocking/
│   │   └── SKILL.md
│   ├── step-analysis/
│   │   └── SKILL.md
│   ├── agent-troubleshooting/
│   │   └── SKILL.md
│   └── pipeline-authoring/
│       └── SKILL.md
└── README.md
```

## Contributing

Contributions are welcome! This plugin is designed to be modular and extensible.

### Adding a New Skill

1. Create a new directory under `skills/`:

   ```bash
   mkdir skills/my-new-skill
   ```

2. Add a `SKILL.md` file with frontmatter:

   ```markdown
   ---
   name: my-new-skill
   description: What this skill does and when Claude should use it.
   ---

   # My New Skill

   Instructions for Claude on how to use this skill...
   ```

## License

MIT
