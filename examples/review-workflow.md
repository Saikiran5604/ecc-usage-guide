# /code-review — Detailed Workflow Example

## When to Use

- Before every `git push`
- Before creating a pull request
- When reviewing someone else's PR
- After a large refactor to sanity-check changes

## Example: Local Review

### You type:

```
/code-review
```

### Claude's process:

**Phase 1 — Gather:**
```bash
git diff --name-only HEAD
```
Lists all changed files. If no changes found, stops with "Nothing to review."

**Phase 2 — Review each file:**

Claude reads every changed file in full and checks for:

**Security Issues (Critical):**
- SQL injection via string interpolation
- Missing authentication/authorization
- Exposed secrets or API keys
- XSS vulnerabilities
- Path traversal
- Command injection

**Logic Bugs:**
- Missing `await` on async calls
- Null/undefined dereferences
- Off-by-one errors
- Race conditions
- Incorrect error handling

**Style & Quality:**
- Unused imports/variables
- Inconsistent naming
- Missing error handling at boundaries
- Dead code

### Sample output:

```
Reviewing 4 changed files...

SECURITY [auth.py:42]: SQL query uses f-string interpolation.
  Fix: Use parameterized query → cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))

BUG [services/user.py:88]: Missing `await` on async DB call.
  Returns coroutine object instead of result.

STYLE [routes/health.py:15]: Unused import `json`.

Summary: 1 security issue, 1 bug, 1 style issue across 4 files.
```

## Example: PR Review

### You type:

```
/code-review 142
```

or:

```
/code-review https://github.com/user/repo/pull/142
```

Claude fetches the PR diff from GitHub and runs the same review process.

## What Makes This Better Than Manual Review

1. **Consistent** — checks the same security patterns every time
2. **Fast** — reviews 10+ files in seconds
3. **No blind spots** — checks every changed line, not just what looks different
4. **Actionable** — provides specific fix suggestions with line numbers
