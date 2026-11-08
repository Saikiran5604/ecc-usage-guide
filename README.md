# ECC (Everything Claude Code) — Usage Guide & Setup

A practical, security-audited guide to setting up and using [ECC](https://github.com/affaan-m/ECC) — the agent harness performance optimization system for Claude Code.

This repo documents my workflow for using ECC's 71 agents, 314 skills, and 98 slash commands to build software faster and safer.

---

## Why ECC?

Claude Code is powerful on its own, but ECC adds **structured workflows** that make it more predictable and productive:

- **Plan before coding** — `/plan` analyzes requirements, identifies risks, and waits for your approval before writing a single line
- **Catch bugs before pushing** — `/code-review` finds security issues, logic bugs, and style violations in your uncommitted changes
- **Fix builds automatically** — `/build-fix` detects your build system and fixes errors one by one
- **Learn from sessions** — `/learn` extracts reusable patterns so Claude remembers solutions across sessions

---

## Security Audit Summary

Before using ECC, I ran a **3-layer independent security audit** covering all 4,701 files:

| Area | Verdict | Details |
|------|---------|---------|
| Install scripts | **SAFE** | Uses `--ignore-scripts` to block supply-chain RCE |
| Supply chain (npm) | **SAFE** | Only 5 runtime deps, no lifecycle hooks, zero binaries |
| Credential access | **SAFE** | GateGuard actively blocks `.env`, `id_rsa`, `credentials` |
| Agents & Skills (385 files) | **SAFE** | No prompt injection, every agent has defense baseline |
| Hooks (20+) | **MEDIUM** | Wildcard matchers on every tool call; start with `--no-hooks` |
| MCP catalog | **MEDIUM** | Third-party endpoints if configured; only enable what you need |
| Network calls | **SAFE** | No calls to external endpoints from hook scripts |

**Result: 0 critical issues. No data exfiltration, credential theft, or malicious code found.**

---

## Safe Installation

### Prerequisites

- Node.js 18+
- Git
- Claude Code 2.1+

### Step 1: Install with minimal hooks

```bash
npx ecc-universal@2.2.3 setup --mode claude-plugin --scope user --hooks minimal --yes
```

### Step 2: Reload Claude Code

```bash
# In Claude Code, run:
/reload-plugins
```

### Step 3: Verify installation

```bash
# Check plugin is registered
cat ~/.claude/plugins/installed_plugins.json
```

### Optional: Enable hooks later

```bash
# Upgrade to standard hooks after you're comfortable
npx ecc-universal@2.2.3 setup --mode claude-plugin --scope user --hooks standard --yes

# Disable the continuous learning observer if you prefer privacy
export ECC_DISABLED_HOOKS=observe-runner
```

---

## Daily Workflow

Here's how I use ECC commands in a typical workday:

```
Morning:    /plan          → plan the task before coding
Building:   /feature-dev   → guided implementation with codebase exploration
Pre-push:   /code-review   → catch bugs and security issues
CI failure: /build-fix     → auto-detect and fix build errors
End of day: /learn         → save patterns for future sessions
```

---

## Command Reference with Examples

### `/plan` — Plan before coding

Use when starting any non-trivial task. Claude analyzes requirements, identifies risks, creates a step-by-step plan, and **waits for your approval**.

```
/plan add a /health endpoint that checks database connectivity and Redis status
```

**What Claude does:**
1. Restates requirements clearly
2. Identifies risks (e.g., timeout handling per dependency)
3. Breaks down into phases
4. Waits for your explicit "go ahead"

---

### `/code-review` — Review before pushing

Reviews uncommitted changes or a GitHub PR for security issues, bugs, and style problems.

```bash
# Review local uncommitted changes
/code-review

# Review a specific PR
/code-review 142

# Review by URL
/code-review https://github.com/user/repo/pull/142
```

**What it checks:**
- **Security**: SQL injection, missing auth, exposed secrets, XSS
- **Bugs**: Missing `await`, wrong types, null dereferences
- **Style**: Unused imports, inconsistent naming

---

### `/build-fix` — Fix build errors automatically

Auto-detects your build system and fixes errors one by one, re-checking after each fix.

```
/build-fix
```

**Supported build systems:**

| Indicator | Build Command |
|-----------|---------------|
| `package.json` | `npm run build` |
| `tsconfig.json` | `npx tsc --noEmit` |
| `Cargo.toml` | `cargo build` |
| `go.mod` | `go build ./...` |
| `pyproject.toml` | `python -m compileall -q .` |
| `pom.xml` | `mvn compile` |

---

### `/feature-dev` — Full feature workflow

The most complete workflow — Claude explores your codebase first, asks questions, designs, implements, and reviews.

```
/feature-dev add user role-based access control with admin and member roles
```

**Phases:**
1. **Discovery** — reads the request, identifies requirements
2. **Codebase exploration** — traces existing code and architecture
3. **Clarifying questions** — asks targeted design questions
4. **Architecture design** — provides implementation blueprint
5. **Implementation** — builds following the approved design
6. **Quality review** — reviews the result

---

### `/learn` — Save patterns for next time

Extracts reusable patterns from the current session and saves them as skills.

```
/learn
```

**What it extracts:**
- Error resolution patterns (what broke, why, what fixed it)
- Debugging techniques (non-obvious steps that worked)
- Workarounds (library quirks, API limitations)
- Project-specific patterns (conventions, architecture decisions)

---

## Other Useful Commands

| Command | Description |
|---------|-------------|
| `/tdd` | Test-driven development — write tests first, then implementation |
| `/e2e` | Generate and run end-to-end tests |
| `/checkpoint` | Save progress mid-session |
| `/go-build` | Go-specific build error resolution |
| `/go-review` | Go-specific code review |
| `/python-review` | Python-specific code review |
| `/cost-report` | View token usage and cost metrics |

---

## Security Precautions

### Do
- Start with `--hooks minimal` and upgrade gradually
- Only enable MCP servers you actually need
- Review `git diff --staged` before every commit
- Use per-project scope (`--scope local`) for sensitive repos

### Don't
- Don't enable all MCP servers from the catalog blindly
- Don't configure the `filesystem` MCP with broad paths
- Don't commit `.env` files (ECC's GateGuard blocks this, but verify your `.gitignore`)
- Don't use `memxus` or `parallel-search` MCP servers (they send data to third-party endpoints)

---

## Project Structure

```
├── README.md              # This guide
├── security-audit/
│   └── AUDIT-REPORT.md    # Full security audit findings
├── examples/
│   ├── plan-workflow.md    # Detailed /plan example
│   ├── review-workflow.md  # Detailed /code-review example
│   └── feature-workflow.md # Detailed /feature-dev example
└── config/
    └── recommended.md      # Recommended ECC configuration
```

---

## References

- [ECC Repository](https://github.com/affaan-m/ECC)
- [ECC Security Guide](https://github.com/affaan-m/ECC/blob/main/the-security-guide.md)
- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)

---

## License

MIT
