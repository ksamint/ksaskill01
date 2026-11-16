---
name: ksamint-githubaction-skill
description: Audit and retire existing GitHub Actions workflows, runner dependencies, and deployment triggers. Use to migrate checks to the local release path or diagnose a stale workflow reference.
---

# Retire GitHub Actions

1. Identify active workflows, their triggering events, the jobs behind required branch checks, and any production mutation. Inspect existing runs read-only.
2. Map each required check to a local command. Run the commands on the exact release tree and attach results to the PR. Preserve the check's coverage and failure behavior.
3. Replace the deployment trigger with the repository's validated direct release path. Use `direct-deploy` for the artifact, receiver, release lock, health, and rollback procedure.
4. When authorized to retire the workflow, prevent new runs before merging a change that would trigger it. Treat any in-flight deployment as active until its terminal state and production revision are verified. Change required branch checks only within the owner's authorized migration scope.
5. Verify that the new entry point validates locally and that the direct release path works. Report any remaining checks or workflows that still require migration.

Do not dispatch workflows, provision runners, or use a hosted fallback. Read archived runbooks only as evidence.
