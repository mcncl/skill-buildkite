# Buildkite Skills

AI skills for working with [Buildkite](https://buildkite.com) CI/CD pipelines.

## Skills

| Skill | Description |
|-------|-------------|
| build-debugging | Analyzes failed builds, reads logs, identifies root causes |
| build-status | Shows recent build status for a pipeline |
| log-retrieval | Retrieves and displays job logs |
| build-triggering | Triggers new builds with branch/commit control |
| build-unblocking | Unblocks builds waiting for manual approval |
| step-analysis | Explains why steps were skipped or didn't run |
| agent-troubleshooting | Diagnoses agent issues and queue mismatches |
| pipeline-authoring | Helps write and improve pipeline configurations |

## Requirements

- An AI tool that supports skills and MCP
- A Buildkite account

These skills use Buildkite's [remote MCP server](https://buildkite.com/docs/apis/mcp-server) with OAuth authentication.

## Usage

Just ask naturally:

- "Why did build 123 fail?"
- "Show me recent builds for the deploy pipeline"
- "My build is stuck waiting for an agent"
- "Help me add a matrix build for testing on multiple Node versions"

## License

MIT
