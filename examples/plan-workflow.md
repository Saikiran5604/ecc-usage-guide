# /plan — Detailed Workflow Example

## When to Use

- Starting any non-trivial feature
- Making architectural changes
- Complex refactoring across multiple files
- Requirements are unclear or ambiguous

## Example: Adding a Health Check Endpoint

### You type:

```
/plan add a /health endpoint that checks database connectivity, 
Redis cache status, and returns service version from package.json
```

### Claude responds with:

**Requirements Restatement:**
1. New `GET /health` endpoint
2. Check database connection (ping with timeout)
3. Check Redis connection (ping with timeout)
4. Return service version from `package.json`
5. Return aggregate status (healthy/degraded/unhealthy)

**Risks Identified:**
- Health check must timeout per dependency (don't hang if Redis is down)
- Should not expose internal error details in production
- Consider caching the response briefly to avoid hammering dependencies

**Implementation Plan:**

| Phase | Steps | Complexity |
|-------|-------|------------|
| 1. Health controller | Create controller with timeout wrapper utility | Low |
| 2. Dependency checks | Add DB ping and Redis ping with individual timeouts | Medium |
| 3. Response format | Aggregate results with per-component status | Low |
| 4. Route registration | Register `GET /health` with no auth middleware | Low |
| 5. Tests | Unit tests for each check + integration test | Medium |

**Estimated complexity:** Medium

*Shall I proceed with this plan?*

### You respond:

```
yes, but add a 3-second timeout per check and include 
the uptime in the response
```

### Then Claude implements

Only after your explicit approval does Claude start writing code — following the plan you agreed on, with your modifications included.

## Key Behavior

- Claude **never writes code** before you confirm
- The plan is grounded in your actual codebase (if a repo is available)
- You can modify the plan before approving
- Risks and dependencies are called out upfront

## Variants

| Command | Description |
|---------|-------------|
| `/plan` | Single-model inline plan (default) |
| `/plan path/to/feature.prd.md` | Plan from a PRD document |
