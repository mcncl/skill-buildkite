# Buildkite Skills

AI skills for working with [Buildkite](https://buildkite.com) CI/CD pipelines.

## Installation

```bash
# Install all skills
amp skill add mcncl/skill-buildkite

# Or install a specific skill
amp skill add mcncl/skill-buildkite/build-debugging
```

## Skills

| Skill | Description |
|-------|-------------|
| build-debugging | Analyzes failed builds to identify root causes and provide fixes |
| build-status | Shows recent build status for a pipeline with pass/fail visibility |
| log-retrieval | Retrieves and displays job logs (for debugging, use build-debugging) |
| build-triggering | Triggers new builds with branch/commit control and safety confirmations |
| build-unblocking | Unblocks builds waiting for manual approval |
| step-analysis | Explains why steps were skipped, didn't run, or behaved unexpectedly |
| agent-troubleshooting | Diagnoses agent issues like stuck jobs and queue mismatches |
| pipeline-authoring | Helps write and improve pipeline YAML configurations |

Each skill bundles its own MCP configuration with filtered tools for optimal token efficiency.

## Requirements

- [Amp](https://ampcode.com) or another AI tool that supports skills and MCP
- A Buildkite account

These skills use Buildkite's [remote MCP server](https://buildkite.com/docs/apis/mcp-server) with OAuth authentication.

## Usage

Just ask naturally:

- "Why did build 123 fail?"
- "Show me recent builds for the deploy pipeline"
- "My build is stuck waiting for an agent"
- "Help me add a matrix build for testing on multiple Node versions"
- "Trigger a build on my branch"
- "Unblock the staging deploy"

## License

MIT
