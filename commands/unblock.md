---
description: Unblock a Buildkite build that's waiting for manual approval
---

# Unblock Build

The user wants to unblock a build step that's waiting for manual approval.

## Approach

1. **Identify the blocked job**
   - Parse build/job information from "$ARGUMENTS"
   - Or find blocked jobs in recent builds

2. **Show block details**
   - What step is blocked
   - What fields need to be filled (if any)
   - Who can unblock it

3. **Collect required input**
   - If the block step has required fields, ask the user for values
   - Validate the input matches expected format

4. **Unblock the job** using `buildkite_unblock_job`
   - Pass any required field values
   - Confirm success

5. **Report status**
   - Confirm the job was unblocked
   - Show next steps in the pipeline

## Examples

- `/buildkite:unblock build-123` - Find and unblock blocked step in build 123
- `/buildkite:unblock https://buildkite.com/org/pipeline/builds/123` - Unblock from URL

## Safety

- Always show what will be unblocked before doing it
- If the block is a deploy gate, get explicit confirmation
- Show any required fields that need to be filled
