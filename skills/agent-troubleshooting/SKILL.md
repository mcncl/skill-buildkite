---
name: agent-troubleshooting
description: Troubleshoots Buildkite agent issues including jobs stuck waiting for agents, agent connectivity problems, and queue configuration. Use when jobs aren't being picked up or there are agent-related problems.
---

# Agent Troubleshooting

You are helping troubleshoot Buildkite agent issues - jobs not being picked up, agent connectivity, or queue configuration problems.

## Approach

1. **Understand the Symptom**
   - Jobs stuck in "scheduled" state = no agent picking them up
   - Jobs stuck in "assigned" state = agent accepted but not starting
   - Jobs failing immediately = agent issue or configuration problem

2. **Check Cluster and Queue Configuration**
   - Use `buildkite_list_clusters` to see available clusters
   - Use `buildkite_list_cluster_queues` to see queues in each cluster
   - Compare job's required queue/tags with available queues

3. **Analyze Job Agent Requirements**
   - Use `buildkite_get_build` to see job details
   - Check the `agent_query_rules` or agent tags required by the job
   - Look for queue targeting: `queue=deploy`, `queue=default`

4. **Check for Common Issues**

### Jobs Not Being Picked Up

**Queue Mismatch**
- Job requires `queue=special` but no agents are in that queue
- Check pipeline YAML for `agents:` configuration
- Verify queue exists with `buildkite_list_cluster_queues`

**Tag Mismatch**
- Job requires specific tags like `docker=true` or `os=linux`
- No agents match ALL required tags (tags are AND-ed)
- Check what tags are configured on available agents

**No Agents Running**
- Queue exists but no agents are connected
- Check Buildkite Agents page or use cluster APIs
- Verify agent is running and can reach Buildkite

**Agent Concurrency Exhausted**
- Agents exist but are all busy
- Check `spawn` setting on agents
- Consider scaling up or adjusting parallelism

### Agent Connectivity Issues

**Agent Disconnecting**
- Network instability between agent and Buildkite
- Agent process crashing or being killed
- Resource exhaustion on agent host (memory, disk)

**Agent Not Starting Jobs**
- Agent hooks failing (environment, pre-command)
- Plugin installation failures
- File system permissions

### Configuration Issues

**Pipeline Agent Rules**
```yaml
# Global agent rules (apply to all steps)
agents:
  queue: "deploy"

# Per-step agent rules
steps:
  - command: "deploy.sh"
    agents:
      queue: "deploy"
      cloud: "aws"
```

**Agent Tag Format**
- Tags are key=value or just key (boolean)
- Multiple tags are AND conditions
- Use `*` wildcard for partial matching: `instance-type=m5.*`

## Diagnostic Steps

1. **List available queues**
   ```
   Use buildkite_list_cluster_queues to see all queues
   ```

2. **Check queue details**
   ```
   Use buildkite_get_cluster_queue for specific queue stats
   ```

3. **Compare with job requirements**
   - Get the build details
   - Look at failed/waiting job's agent configuration
   - Match against available queues

## Response Format

1. **Issue Identified**: What's preventing job execution
2. **Agent Configuration**: What the job requires vs what's available
3. **Root Cause**: Specific mismatch or problem
4. **Resolution**: Steps to fix
   - Immediate: How to get this build working
   - Long-term: How to prevent recurrence

## Common Resolutions

- **Wrong queue**: Update pipeline YAML or add agents to queue
- **Missing tags**: Add tags to agents or remove requirement from job
- **No agents**: Start agents or check agent host health
- **All agents busy**: Scale up agents or reduce parallelism
- **Agent crashing**: Check agent logs, hooks, and host resources

## Important Notes

- Agent issues often manifest as jobs stuck in "scheduled" state
- Always verify the queue/tags in the pipeline match available agents
- Consider that dynamic pipelines might change agent requirements
- Cluster configuration affects which queues are available
