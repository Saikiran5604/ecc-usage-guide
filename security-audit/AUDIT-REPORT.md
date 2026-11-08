# ECC Security Audit Report

**Repository:** [affaan-m/ECC](https://github.com/affaan-m/ECC) v2.2.3  
**Date:** 2026-11-08  
**Method:** Three independent parallel reviews covering all 4,701 files  

---

## Executive Summary

**Verdict: SAFE for use with recommended precautions.**

No malicious code, data exfiltration, credential theft, or prompt injection found. The project includes strong defensive features (GateGuard, AgentShield, config-protection, governance-capture).

Three medium-risk areas identified — all operational, not malicious.

---

## Audit Methodology

Three independent reviewers analyzed the full repository in parallel:

1. **Reviewer A** — Install scripts, hooks, MCP configs, Claude settings
2. **Reviewer B** — Agents, skills, rules, commands (prompt injection focus)
3. **Reviewer C** — JavaScript, Python, Rust code, package.json, Docker, binaries

---

## Detailed Findings

### 1. Install Scripts — SAFE

| File | Finding |
|------|---------|
| `install.sh` | Clean bash wrapper. Uses `npm install --ignore-scripts` to block supply-chain RCE. Delegates to Node.js installer. No hidden downloads. |
| `install.ps1` | Windows equivalent. Same `--ignore-scripts` protection. No credential access. |
| `scripts/install-apply.js` | Node.js installer. Copies config files to target directory. No network calls beyond npm. |

### 2. Supply Chain (package.json) — SAFE

- **5 runtime dependencies** (all well-known): `@iarna/toml`, `ajv`, `js-yaml`, `sql.js`, `zod`
- **No `postinstall`/`preinstall` lifecycle hooks**
- `--ignore-scripts` enforced in install.sh
- Zero compiled binaries in the repository

### 3. Credential & Secret Access — SAFE

- No scripts read `~/.ssh`, `~/.aws`, `~/.gnupg`, `~/.npmrc`, or `~/.kube`
- GateGuard (`gateguard-target-class.js:281`) actively **blocks** access to `.env`, `id_rsa`, `.netrc`, `credentials`, `secrets.*`
- Dashboard (`dashboard-web.js`) redacts env values to `••••••` before display
- `GITHUB_TOKEN` accessed only in CI scripts (`pr-burst-guard.js`)
- No access to `AWS_*`, `SSH_*`, `GPG_*`, `OPENAI_*`, or `ANTHROPIC_*` in hook scripts

### 4. Agents & Skills (385 files) — SAFE

- Every agent includes a "Prompt Defense Baseline" that blocks:
  - Identity/persona override
  - Credential leakage
  - Malicious output generation
  - Prompt injection via unicode, homoglyphs, or embedded commands
- No instructions to send data externally
- No instructions to bypass permission checks
- No base64-encoded content in agent/skill files
- No obfuscated code
- `skill-comply` tests explicitly block malicious install patterns

### 5. CLAUDE.md & Rules — SAFE (Defensive)

- Contains explicit prompt defense baseline
- Rejects authority claims, urgency pressure, embedded commands
- Validates and sanitizes external data
- No hidden directives or override instructions

### 6. Hook System — MEDIUM RISK

**Not malicious, but complex.**

- **20+ hooks** registered across PreToolUse, PostToolUse, UserPromptSubmit, SessionStart, Stop, SessionEnd
- **Wildcard matchers (`".*"`)** on 10+ hooks — fires on every tool call, adding latency
- **Bootstrap code** in `hooks.json` is a dense ~2KB inline JS one-liner per hook (path resolution only, not malicious)
- **No network calls** in any hook script — all data stays local
- All hooks have timeouts (5–300 seconds)
- All hooks exit 0 on errors (never block tool execution)
- Path traversal protection present (`resolveTarget()` rejects `../`)

| Hook | Risk | Purpose |
|------|------|---------|
| `plugin-hook-bootstrap.js` | LOW | Spawns hook scripts with 30s timeout |
| `observe-runner.js` | LOW | Continuous learning observer (local only) |
| `governance-capture.js` | LOW | Scans for leaked secrets (opt-in) |
| `session-start-bootstrap.js` | LOW | Loads session context |
| `session-end.js` | LOW | Writes session summary locally |
| `cost-tracker.js` | LOW | Writes cost metrics to `~/.claude/metrics/` |
| `hookify-runtime.js` | MEDIUM | Dense; fires on every tool use + prompt submit |

**Recommendation:** Start with `--hooks minimal` or `--hooks off`.

### 7. MCP Configuration — MEDIUM RISK

- `.mcp.json` declares only `chrome-devtools-mcp@1.10.1` (safe, version-pinned)
- `mcp-configs/mcp-servers.json` is a **catalog template**, not auto-installed
- All credentials use `YOUR_*_HERE` placeholders — no hardcoded secrets
- **Concerns:**
  - `filesystem` MCP could expose entire disk if misconfigured
  - `memxus` and `parallel-search` are third-party HTTP endpoints (data leaves local machine)
  - `ecc-memory-vault` warns about memory-poisoning risk

**Recommendation:** Only enable MCP servers you actually use. Scope filesystem access narrowly.

### 8. Auto-Update — MEDIUM RISK

- `auto-update.js` runs `git fetch --all --prune` and `git pull --ff-only`
- Validates repo identity via package name check (`ecc-universal` or `everything-claude-code`)
- No arbitrary network calls beyond git

### 9. Python & Docker — SAFE

- Docker uses `node:22-bookworm-slim` (official image), runs as non-root
- Python dependencies: `anthropic` and `openai` (standard LLM libraries)
- No suspicious subprocess calls

### 10. Binary Files — CLEAN

No compiled binaries found (`.so`, `.dll`, `.exe`, `.wasm`, `.o`, `.a` — all zero results).

---

## Summary Matrix

| Area | Rating | Key Finding |
|------|--------|-------------|
| Install scripts | **SAFE** | `--ignore-scripts` enforced |
| Supply chain | **SAFE** | 5 known deps, no lifecycle hooks |
| Credential access | **SAFE** | GateGuard blocks sensitive files |
| Agents & Skills | **SAFE** | Prompt defense baseline on all |
| CLAUDE.md | **SAFE** | Defensive guardrails |
| Hooks (logic) | **LOW** | No exfiltration, local only |
| Hooks (surface) | **MEDIUM** | 20+ wildcard hooks add latency |
| MCP catalog | **MEDIUM** | Third-party endpoints if configured |
| Auto-update | **MEDIUM** | Runs git pull, validated |
| Python/Docker | **SAFE** | Standard tooling |
| Binaries | **CLEAN** | None found |

---

## Recommendations

1. Install with `--hooks minimal` or `--hooks off` initially
2. Only enable MCP servers you actually use
3. Disable `observe-runner` if you don't want session tracking: `ECC_DISABLED_HOOKS=observe-runner`
4. Never commit `.env` files (GateGuard helps, but verify `.gitignore`)
5. Review hook updates when upgrading ECC versions
6. Use per-project scope (`--scope local`) for sensitive repositories
