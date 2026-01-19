---
description: Get logs from a Buildkite job - useful for investigating specific step output
---

# Get Logs

The user wants to view logs from a Buildkite job. Parse "$ARGUMENTS" for build/job information.

## Approach

1. **Parse the request**
   - Could be a job URL
   - Could be a build number and step name
   - Could be "the failed job from build X"

2. **Fetch logs** using `buildkite_read_logs`
   - If a specific job is identified, get those logs
   - If just a build is provided, ask which job or show failed job logs

3. **Present the logs**
   - For long logs, show the most relevant section (usually the end)
   - Highlight errors, warnings, or the specific section the user needs
   - Offer to show more if the logs are truncated

4. **Search capability**
   - If the user is looking for something specific, use `buildkite_search_logs`
   - Help them find relevant lines in verbose output

## Examples

- `/buildkite:logs build-123 job-456` - Logs for a specific job
- `/buildkite:logs https://buildkite.com/org/pipeline/builds/123#job-id` - Logs from URL
- `/buildkite:logs build-123` - Show logs from failed jobs in build 123
