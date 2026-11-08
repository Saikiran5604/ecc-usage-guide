# Recommended ECC Configuration

## Installation Profiles

### For personal projects (start here)

```bash
npx ecc-universal@2.2.3 setup --mode claude-plugin --scope user --hooks minimal --yes
```

- Scope: `user` — available in all your projects
- Hooks: `minimal` — only essential hooks, low latency
- Safe for daily use

### For team/work repositories

```bash
npx ecc-universal@2.2.3 setup --mode claude-plugin --scope local --hooks standard --yes
```

- Scope: `local` — project-specific, not committed to repo
- Hooks: `standard` — adds quality and safety automation
- Keeps work config separate from personal

### For sensitive repositories

```bash
npx ecc-universal@2.2.3 setup --mode claude-plugin --scope local --hooks off --yes
```

- Hooks: `off` — no automated hooks at all
- Commands and skills still work
- Maximum control, minimum risk

## Hook Profiles Compared

| Feature | off | minimal | standard | strict |
|---------|-----|---------|----------|--------|
| Slash commands | Yes | Yes | Yes | Yes |
| Skills | Yes | Yes | Yes | Yes |
| Agents | Yes | Yes | Yes | Yes |
| Session start/end | No | Yes | Yes | Yes |
| Pre-tool safety | No | No | Yes | Yes |
| Post-tool quality | No | No | Yes | Yes |
| Continuous learning | No | No | Opt-in | Yes |
| Governance capture | No | No | No | Yes |

## Environment Variables

```bash
# Disable specific hooks by name
export ECC_DISABLED_HOOKS=observe-runner,cost-tracker

# Set hook profile at runtime (overrides install choice)
export ECC_HOOK_PROFILE=minimal

# Limit hook input size (bytes)
export ECC_HOOK_INPUT_MAX_BYTES=65536
```

## MCP Servers — Only Enable What You Need

The ECC MCP catalog (`mcp-configs/mcp-servers.json`) contains 30+ server configs. **Do not enable all of them.** Here are the safe, recommended ones:

### Safe to enable

| Server | Purpose |
|--------|---------|
| `chrome-devtools` | Browser debugging (already in `.mcp.json`) |
| `github` | GitHub API access (needs `GITHUB_TOKEN`) |
| `context7` | Documentation lookup |
| `playwright` | Browser automation for E2E tests |

### Use with caution

| Server | Risk |
|--------|------|
| `filesystem` | Scope to specific project directories only |
| `supabase` | Only if you use Supabase |
| `ecc-memory-vault` | Shared memory across harnesses — review trust model |

### Avoid unless necessary

| Server | Reason |
|--------|--------|
| `memxus` | Third-party HTTP endpoint — data leaves your machine |
| `parallel-search` | Third-party HTTP endpoint |

## Recommended .gitignore additions

```gitignore
# ECC local state
.env
.env.local
~/.claude/metrics/
~/.claude/sessions/
```
