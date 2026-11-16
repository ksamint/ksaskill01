# Agent notes for ksamintskill01

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

## Local validation and direct deployment

Run the release checks from a clean checkout of the exact source tree. Include
the tested tree SHA and results in the PR.

```bash
node --test gui/lib/*.test.js
python3 scripts/lint-skills.py
bash scripts/scan-secrets.sh skills/
bash -n scripts/package-release.sh
bash -n scripts/receive-release.sh
```

Use `direct-deploy` for an authorized Skill Hub application release. Build the
completed package on the local or authorized build host. The historical
`scripts/receive-release.sh` expects a GitHub artifact URL. Application
publishing requires a tested direct receiver that verifies the package
checksum and exact revision before activation and preserves release locking,
health checks and rollback. Source PRs and local skill installs are separate
from application publishing.

## Skill Hub GUI

Local panel at `gui/` (`bash scripts/dev-up.sh` → http://127.0.0.1:7979). It observes repo/sync status, skills, `.work` runs, and can launch **allowlisted** templates. Coverage and fidelity gates remain authoritative in the Python scripts — do not treat the GUI as a substitute for `check-coverage.py` / `deck-audit`. Stages b–c of `longdoc2mdpages` are still agent-written; bootstrap output is a mechanical draft.

The public `long4hslides` template normalizes the source, closes the **file pack** (`deck-plan.json` + `pack.json` + `MANIFEST.md`), and requires explicit project approval. Only then may its hidden slides stage render, measure in Chrome, and run hop2. Historical `alongslides`, `longdoc2mdpages`, `longdoc-to-deck`, `baslide-slides`, and `deck-audit-hop2` ids remain hidden executable compatibility aliases for old projects and job reruns. `BASLIDE_ROOT` defaults to `modules/baslide01`.
