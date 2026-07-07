---
name: checkpoint
description: Create a concise git checkpoint commit describing the current changes.
---

# Checkpoint

When invoked:

1. Review the current git diff (`git diff --staged` and `git diff`).
2. Stage all modified and new files if appropriate.
3. Generate a short, descriptive commit message.

## Commit message rules

- Use imperative mood.
- Keep the subject under 72 characters.
- No period at the end.
- Format:

```
<type>: <summary>
```

Allowed types:

- feat
- fix
- refactor
- docs
- test
- chore
- perf
- ci
- build
- style

Examples:

```
feat: add CRUD API for users
feat: implement OAuth login
fix: run server on correct port
fix: handle null database response
refactor: simplify authentication middleware
refactor: extract SQL builder
docs: update installation guide
test: add login integration tests
chore: update dependencies
```

4. Commit using the generated message.

If the changes span multiple independent features, choose the primary purpose.

Never invent functionality that is not present in the diff.