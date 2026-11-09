# Agent notes for ksamintskill01

<!-- ksamint-policy-pointer v1 -->
> **Global agent policy:** this repository follows ksamint ADR-0000 (claudememo01 `global/AGENT-POLICY.md`):
> TypeScript first, Rust first (no new JS) · no CI runners (no GitHub Actions, hosted or self-hosted) · validate locally on the
> exact tree · the agent merges on green local validation · deploy directly from the local deploy machine with automatic
> rollback · hard limits on shared infrastructure, secrets and product rules. Stricter safety rules below still apply.
<!-- /ksamint-policy-pointer -->

## Source of truth

- Authored skills live under `skills/<name>/` with a `SKILL.md`.
- Do not edit copies under `~/.cursor/skills`, `~/.claude/skills`, or `~/.codex/skills` when those paths are symlinks into this repo. Edit here, then re-run `scripts/install-links.sh` if needed.
- Never commit `vendor/` — it is produced by `scripts/sync-vendor.sh`.

## longdoc2mdpages

Zero-loss is enforced by a **coverage ledger that closes**:

1. `segment.py` → `index.json` + `index.md`
2. Agent writes `outline.md` (every unit id once) → `check-coverage.py --stage outline`
3. Agent paginates into `deck.json` + `pages/` → `check-coverage.py --stage deck` and `estimate-fit.py`
4. `emit-pack.py` → shared GF `deck-plan.json`; fidelity + schema gates close the pack

Do not emit HTML or CSS from `longdoc2mdpages`.

## Secrets

Before committing skills that mention 1Password, private hosts, or API keys, run:

```bash
bash scripts/scan-secrets.sh skills/
```

Any plaintext key blocks the commit.

## Catalog

When the user names a short skill command or ID (for example `orche` or `k023`), resolve its `name` using `registry/reviews-authored.json` or `registry/reviews-mattpocock.json`, then read that skill's SKILL.md before using it. These files are the command directory and static quality-review ledger. Keep assigned commandId values stable; append new IDs and never recycle deleted ones. Recommendations are review evidence, not permission to delete skills. Other repositories need this directory or the canonical skill name to resolve these shortcuts.

After adding or renaming a skill:

```bash
python3 scripts/build-catalog.py
```

## Skill import releases

This public repository deploys directly from the local deploy machine, with no GitHub Actions or runners (hosted or self-hosted): re-run the checks there on the exact `main` SHA, deploy, verify, roll back automatically on failure, and write a deploy report. Production credentials belong to the main-only `production` Environment. PR validation runs locally on the exact commit in a clean checkout, without production secrets. Follow the deployment run through completion and verify the live SHA, catalog, projects, runs, registry, and jobs before reporting success. Initial installation of the restricted deployment receiver is infrastructure setup, not an alternate release route.

When the user supplies a skill package for this repository and asks to add or install it, validate it, rebuild the catalog, update local links, commit on a branch, open a PR, merge it when local validation of the PR head rebased on `main` is green, and deploy the exact remote `main` SHA to the existing Skill Hub production target. Stop before publication if the worktree has unrelated changes, validation fails, the remote diverges, or the established deployment target, rollback path, or smoke checks are unavailable.

## Skill Hub GUI

Local panel at `gui/` (`bash scripts/dev-up.sh` → http://127.0.0.1:7979). It observes repo/sync status, skills, `.work` runs, and can launch **allowlisted** templates. Coverage and fidelity gates remain authoritative in the Python scripts — do not treat the GUI as a substitute for `check-coverage.py` / `deck-audit`. Stages b–c of `longdoc2mdpages` are still agent-written; bootstrap output is a mechanical draft.

The public `long4hslides` template normalizes the source, closes the **file pack** (`deck-plan.json` + `pack.json` + `MANIFEST.md`), and requires explicit project approval. Only then may its hidden slides stage render, measure in Chrome, and run hop2. Historical `alongslides`, `longdoc2mdpages`, `longdoc-to-deck`, `baslide-slides`, and `deck-audit-hop2` ids remain hidden executable compatibility aliases for old projects and job reruns. `BASLIDE_ROOT` defaults to `modules/baslide01`.
