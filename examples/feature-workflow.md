# /feature-dev — Detailed Workflow Example

## When to Use

- Building any significant new feature
- When the feature touches multiple files or layers
- When you want Claude to understand your codebase before making changes

## Example: Adding Role-Based Access Control

### You type:

```
/feature-dev add user role-based access control to the API. 
Admins can manage all resources, members can only read.
```

### Phase 1 — Discovery

Claude reads the request and identifies:
- Two roles needed: `admin` and `member`
- Admin: full CRUD on all resources
- Member: read-only access
- Need authorization middleware
- Need role storage mechanism

### Phase 2 — Codebase Exploration

Claude uses the `code-explorer` agent to:
- Find existing `User` model
- Trace the auth middleware chain
- Map all route files and their patterns
- Identify integration points

```
Found: User model at models/user.py
Found: Auth middleware at middleware/auth.py  
Found: 12 route files under routes/
Found: No existing role/permission system
```

### Phase 3 — Clarifying Questions

Before designing anything, Claude asks:

1. Should roles be stored in the database or in JWT claims?
2. Should we create a separate `roles` table or add a `role` column to `users`?
3. Are there any routes that should remain public (no auth required)?
4. Should role changes take effect immediately or on next login?

### Phase 4 — Architecture Design

After you answer, Claude creates a blueprint:

```
Architecture Blueprint:
├── models/role.py          (new) Role enum + permission mapping
├── middleware/auth.py       (modify) Add role-checking decorator
├── middleware/permissions.py (new) @require_role('admin') decorator  
├── routes/*.py              (modify) Add permission decorators
└── tests/test_rbac.py       (new) Permission matrix tests
```

*Waiting for your approval before implementing...*

### Phase 5 — Implementation

Claude implements following the approved design:
- Creates files in dependency order
- Writes tests alongside implementation (TDD where appropriate)
- Keeps commits small and focused

### Phase 6 — Quality Review

Claude runs a self-review on what it built:
- Security check on the authorization logic
- Verify all routes have correct permissions
- Check for missing edge cases
- Confirm tests pass

## Why This Is Better Than "Just Build It"

| Without /feature-dev | With /feature-dev |
|---------------------|-------------------|
| Claude dives in immediately | Claude explores your codebase first |
| May miss existing patterns | Follows your existing conventions |
| You discover problems mid-build | Problems surface during planning |
| Inconsistent with existing code | Integrates with existing architecture |
| One big commit | Small, focused changes |
