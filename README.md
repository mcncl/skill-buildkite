# Buildkite Claude Code Plugin

A [Claude Code plugin](https://code.claude.com/docs/en/plugins) for working with [Buildkite](https://buildkite.com) CI/CD pipelines. Debug failed builds, understand why steps didn't run, troubleshoot agent issues, and author pipelines.

## Features

### Skills

Claude automatically uses these capabilities when relevant:

| Skill | Description |
|-------|-------------|
| **build-debugging** | Analyzes failed builds - reads logs, checks test results, identifies root causes |
| **step-analysis** | Explains why steps were skipped, didn't run, or behaved unexpectedly |
| **agent-troubleshooting** | Diagnoses agent issues - jobs stuck waiting, queue mismatches, connectivity problems |
| **pipeline-authoring** | Helps write and improve pipeline configurations with best practices |

### Slash Commands

| Command | Description |
|---------|-------------|
| `/buildkite:debug <build>` | Debug a failed build |
| `/buildkite:status [pipeline]` | Check recent build status |
| `/buildkite:logs <build> [job]` | View job logs |
| `/buildkite:trigger <pipeline>` | Trigger a new build |
| `/buildkite:unblock <build>` | Unblock a waiting build |

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
   git clone https://github.com/mcncl/buildkite-claude-plugin.git
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
├── commands/                 # Slash commands
│   ├── debug.md
│   ├── logs.md
│   ├── status.md
│   ├── trigger.md
│   └── unblock.md
├── skills/                   # AI capabilities
│   ├── build-debugging/
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

### Adding a New Command

1. Create a new `.md` file in `commands/`:

   ```markdown
   ---
   description: Short description shown in command list
   ---

   # Command Name

   Instructions for what the command should do...
   Use $ARGUMENTS to access user input.
   ```

## License

MIT
