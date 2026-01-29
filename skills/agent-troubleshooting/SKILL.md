---
name: agent-troubleshooting
description: "Diagnoses Buildkite agent issues like stuck jobs and queue mismatches."
---

# Agent Troubleshooting

Diagnose why jobs aren't being picked up by agents.

## When to use

- "My build is stuck waiting for an agent"
- "Jobs aren't being picked up"
- "Why is my build stuck in scheduled?"
- "Agent not running my job"
- "Queue issues"
- "No agents available"

## Available MCP Tools

| Tool | Purpose |
|------|---------|
| `get_build` | Get job details including agent requirements |
| `list_clusters` | List available clusters |
| `get_cluster` | Get detailed cluster information |
| `list_cluster_queues` | List queues in a cluster |
| `get_cluster_queue` | Get queue stats (agent count, jobs waiting) |

## Input Parsing

User typically describes a symptom:

| Input | Likely Issue |
|-------|--------------|
| "build stuck" | Job in scheduled state |
| "waiting for agent" | No matching agents |
| "job not starting" | Agent configuration mismatch |
| "queue problem" | Queue doesn't exist or no agents |

Get the build number/URL to investigate.

## Approach

1. **Get the build** with `buildkite_get_build`
   - Find the stuck job
   - Note its state (scheduled, assigned, etc.)
   - Extract agent query rules (queue, tags)

2. **Check cluster/queue configuration**
   - List clusters with `buildkite_list_clusters`
   - List queues with `buildkite_list_cluster_queues`
   - Get queue stats with `buildkite_get_cluster_queue`

3. **Compare requirements vs availability**
   - What does the job require?
   - What agents/queues exist?
   - Where's the mismatch?

4. **Provide diagnosis and fix**

## Job States for Agent Issues

| State | Meaning | Indicates |
|-------|---------|-----------|
| `scheduled` | Waiting for agent | No matching agent available |
| `assigned` | Agent accepted | Agent has it but not starting |
| `accepted` | Agent starting | Should run soon |

Jobs stuck in `scheduled` = agent matching problem.

## Common Issues

### 1. Queue Mismatch

**Symptom**: Job stuck in scheduled
**Cause**: Job requires queue that doesn't exist or has no agents

```yaml
# Pipeline requires:
agents:
  queue: "deploy"

# But no agents are in the "deploy" queue
```

**Diagnosis**:
```text
Job requires: queue=deploy
Available queues: default (5 agents), build (10 agents)
❌ No "deploy" queue exists
```

**Fix**: Add agents to the deploy queue, or change pipeline to use existing queue.

### 2. Tag Mismatch

**Symptom**: Job stuck in scheduled
**Cause**: Job requires tags no agent has

```yaml
# Pipeline requires:
agents:
  queue: "default"
  docker: "true"
  os: "linux"

# Agents have docker=true but os=macos
```

**Diagnosis**:
```text
Job requires: queue=default, docker=true, os=linux
Available agents in default:
  - agent-1: docker=true, os=macos
  - agent-2: docker=true, os=macos
❌ No agent matches os=linux
```

**Fix**: Add Linux agents, or remove the os requirement.

### 3. No Agents Running

**Symptom**: Job stuck in scheduled
**Cause**: Queue exists but no agents connected

**Diagnosis**:
```text
Job requires: queue=deploy
Queue "deploy" exists but has 0 connected agents
```

**Fix**: Start agents, check agent host health, verify network connectivity.

### 4. All Agents Busy

**Symptom**: Job stuck in scheduled longer than usual
**Cause**: Agents exist but at capacity

**Diagnosis**:
```text
Job requires: queue=default
Queue "default": 3 agents, 15 jobs waiting
Average wait time: 12 minutes
```

**Fix**: Scale up agents, reduce parallelism, or wait.

### 5. Agent Assigned But Not Starting

**Symptom**: Job stuck in assigned state
**Cause**: Agent accepted job but can't start it

**Possible causes**:
- Agent hooks failing (environment, pre-command)
- Plugin installation failing
- Disk space issues
- Agent process problems

**Fix**: Check agent logs on the host machine.

## Response Format

````text
## Agent Issue Diagnosed

**Build**: #456
**Stuck Job**: "Run Tests"
**State**: scheduled (waiting for agent)

### Job Requirements
- Queue: `deploy`
- Tags: `docker=true`

### Available Resources
- Queue `deploy`: ❌ Does not exist
- Queue `default`: 5 agents (none match)

### Root Cause
The job requires `queue=deploy` but no such queue exists in your cluster.

### Fix
**Immediate**: Change the pipeline to use `queue=default`:
```yaml
agents:
  queue: "default"
  docker: "true"
```

**Long-term**: Create a `deploy` queue and add dedicated agents for deployments.
````

## Diagnostic Commands

When explaining fixes, reference these Buildkite agent commands:

```bash
# Check agent status
buildkite-agent status

# See what queues/tags an agent has
buildkite-agent start --tags "queue=deploy,docker=true"

# Check agent logs
journalctl -u buildkite-agent
```

## Example Interaction

```text
User: My build is stuck waiting for an agent

1. Ask for build URL/number
2. Fetch build, find stuck job in "scheduled" state
3. Extract agent requirements: queue=special, gpu=true
4. List queues - "special" exists with 2 agents
5. Check queue details - agents have gpu=false
6. Explain: "Job needs gpu=true but queue agents don't have GPU tag"
7. Suggest: Add GPU agents or modify job requirements
```
