---
name: direct-deploy
description: Release an explicitly authorized service or app directly from a local or authorized build host. Use for local validation, artifact packaging, authenticated transfer, rollout, health checks, or rollback.
metadata:
  author: ksamint
  command-id: k028
  short-name: ship
  version: "1.0.0"
---

# Direct deploy

1. Confirm the authorized repository and environment. Read the project's release guide and the target's `/srv/AGENTS.md` before server operations. Pin the clean release commit and tree, target OS and architecture, and existing authenticated receiver.
2. Run required checks locally on the exact tree. Record the commands and results. Use GitHub for source control and PR review. GitHub Actions and runners are disabled for builds, tests, and releases. Convert any required runner check to its local equivalent within the authorized scope.
3. Build the deployable binary, OCI image, or static package on the local machine or an already authorized build host. Record the commit, tree, artifact SHA-256, and size. Keep secrets in the existing secret manager.
4. Transfer the completed artifact and manifest to the existing authenticated receiver. Verify the received size, hash, revision, and platform before activation. Use the repository's established SSH or direct provider upload path.
5. Activate under the existing release lock with health gates, persistent data, single-writer fences, and rollback intact. Restore the previous healthy release if a gate fails. Reconcile an interrupted deployment before retrying.
6. Verify the live revision, dependency health, and affected user flow. Record before and after versions, artifact identity, check results, and rollback outcome.

If the project lacks a direct receiver, build and test that path within the authorized scope before publishing. Treat historical runner instructions as archival.

For the verified Table AI can01 route, see [vanahom-fb-hom01](references/vanahom-fb-hom01.md). For an established multi-stage rollout, use `meta-rollingdeploy-skill`.
