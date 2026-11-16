# Signed macOS release runbook

## Contents

1. Inspect and choose scope
2. Build on an authorized Mac
3. Monitor signing and notarization
4. Diagnose failures
5. Verify and install
6. Publish formally
7. Lessons preserved from the 2.4.0 signing incident

## 1. Inspect and choose scope

Run from the repository:

```sh
git status -sb
git rev-parse HEAD
git log -1 --oneline
scripts/inspect-release-state.sh "$(pwd)"
```

Inspect the version in the Xcode project and the installed App. Confirm the requested commit is pushed. Do not include unrelated dirty files in a signing commit.

Choose:

- `arm64`: local Apple Silicon installation; fastest useful path.
- `universal`: formal GitHub Release and Homebrew; requires ARM64 and Intel validation.

The current Mac may have only Command Line Tools. Do not require a macOS upgrade just to build. Use an already authorized Mac build host with compatible full Xcode when local Xcode is unavailable; report missing build capacity without requesting GitHub runners.

## 2. Build on an authorized Mac

Build the exact commit locally or on an already authorized Mac host. Read the project signing scripts for the required Xcode version, architectures and profile names. Run the same project acceptance checks before signing. If build tooling is unavailable, report that blocker.

1. Test CoreEditor, Web, and Cloud code.
2. Load Developer ID credentials at runtime into a temporary keychain and install the five active `MAC_APP_DIRECT` provisioning profiles.
3. Archive the selected architecture with hardened runtime and export with manual Developer ID signing and the exact certificate SHA-1.
4. Preserve the signed unstapled artifact before notarization. Submit to Apple, record the submission ID, poll with transient-network retries, then staple and run every verification gate.
5. Transfer the verified binary and its commit/hash manifest directly to the requested destination over authenticated SSH when server delivery is requested. Verify received bytes. Preserve the existing safe local-install procedure for Mac installation.

## 3. Monitor signing and notarization

Poll `notarytool info` for the existing submission using runtime-only credentials. A long `In Progress` state is not a rejection; preserve the signed artifact and avoid duplicate submissions. Once accepted, staple that exact App and verify it before installation. Record submission ID, source commit and artifact hash without printing credentials.

## 4. Diagnose failures

### Invalid notarization

Fetch the log by submission ID without printing credentials:

```sh
xcrun notarytool log "$submission_id" <credential-options>
```

Fix only the reported bundle or executable. Common incident findings:

- `ConversationCaptureHelper.app` lacked hardened runtime.
- `QuickActionExtension.appex` lacked hardened runtime.

Ensure `ENABLE_HARDENED_RUNTIME = YES` for Release and Debug where these targets are embedded and tested. Rebuild and resubmit an exact new commit.

### Archive/export signing failure

Check:

- A Developer ID Application G2 identity with private key is imported.
- `xcodebuild -exportArchive` uses `method=developer-id`, manual signing, expected Team ID, and the imported identity SHA-1.
- Each bundle identifier maps to its active Developer ID provisioning profile.
- The App Group is assigned to the main app, capture helper, Finder extension, and Quick Action extension as required.
- No native dynamic library or unsigned nested code is introduced accidentally.

### Notary queue stall

Apple may leave a valid first submission `In Progress` for hours. Keep one useful current request, retain its artifact, and report the external blocker honestly. Do not weaken the local installer or call the App notarized prematurely.

## 5. Verify and install

Use the signed artifact from the local or authorized Mac build. Verify the artifact checksum against its release manifest.

Extract into a `mktemp -d` directory. Verify:

```sh
/usr/libexec/PlistBuddy -c 'Print :KSAMINTBuildCommit' "$app/Contents/Info.plist"
lipo -archs "$app/Contents/MacOS/ksamint MarkEdit"
codesign --verify --deep --strict --verbose=2 "$app"
spctl --assess --type execute -vv "$app"
xcrun stapler validate "$app"
codesign -dvv "$app"
```

Also inspect the version/build and expected nested bundles. For ARM-only local installation, require `arm64`; do not require `x86_64`. For Universal publication, require both.

Use the exact verified local/build-host artifact. Inspect install helpers before use: a helper that dispatches a workflow is outside this direct-artifact procedure. Preserve the same safe-install sequence below.

Safe-install sequence:

1. Ask the App to quit normally.
2. If it remains open because of windows or unsaved documents, defer; never force-quit.
3. Move the old App to `/Applications/ksamint MarkEdit.backup-YYYYMMDD-HHMMSS.app`.
4. Copy with `ditto`; never use a broad recursive deletion target.
5. Re-run signature and Gatekeeper verification on the installed path.
6. Launch and confirm the process stays alive.
7. Confirm version, build, commit, architecture, Helper, Finder, and Quick Action bundles.
8. On any failure, restore the backup and relaunch it.

## 6. Publish formally

Local ARM success does not authorize release. Before merging and Homebrew:

- All required PR checks pass, including Universal/Intel checks.
- The sequential PR stack is merged in order.
- A Universal Developer ID signed and notarized artifact passes `Scripts/verify-universal.sh`.
- The DMG meets the repository size gate.
- GitHub Release assets, checksums, Apple signature, notarization, and recovery checks pass.
- Homebrew is updated only to the final signed/notarized Release asset.

## 7. Lessons preserved from the 2.4.0 signing incident

- A functional App can still fail notarization because one nested Helper/extension lacks hardened runtime.
- Xcode automatic cloud signing may fail even with the correct certificate; deterministic manual export plus explicit profiles is more reliable.
- A successful archive is not equivalent to a successful export, notarization, or install.
- `notarytool --wait` can terminate on a temporary build-host network outage while Apple continues processing. Preserve the signed artifact before waiting and poll explicitly with retries.
- Always compare the signed artifact's embedded full commit to the requested commit.
- The local installed version must be read from `/Applications`; repository version or CI success is not proof of installation.
- ARM64-only is appropriate for an immediate Apple Silicon local install. Keep Universal validation for formal distribution rather than blocking local testing unnecessarily.
