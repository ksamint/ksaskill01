# Table AI can01 direct release

Use a clean checkout of the exact remote default-branch revision. Run `npm run release:preflight` and `npm run release:check` with a disposable local PostgreSQL parity database. Keep passing receipts for that exact commit and tree.

Build the Linux/amd64 images with `CAN01_DIRECT_ARCHIVE=1 CAN01_REMOTE_BUILD=0 npm run can01:image:build`. Set `CAN01_BUILD_RECEIPT_DIR` to the absolute archive directory and `CAN01_RELEASE_PREFLIGHT_RECEIPT` to the passing preflight receipt. An unchanged workbook converter can reuse the previously verified image if the source fingerprint matches and the release scans it. Verify the archive SHA-256 and image identity before shipping.

Use the same receipt for the read-only plan and the release. Replace `<sha>` with the pinned commit.

```bash
npm run can01:release -- --direct --plan --receipt=.runtime/builds/can01/<sha>/direct/receipt.json
npm run can01:release -- --direct --confirm=<sha> --receipt=.runtime/builds/can01/<sha>/direct/receipt.json
```

Read the target's `/srv/AGENTS.md` before release operations. The direct receiver verifies the delivered archive before loading it. Keep the established read-only warmup, progressive traffic, single-writer cutover, and public probes. Record the release receipt, active slot, health, and the affected-flow smoke result.
