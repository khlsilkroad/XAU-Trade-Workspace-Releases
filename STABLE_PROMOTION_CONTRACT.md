# XAU Trade Workspace — Stable Promotion Contract

Status: ACTIVE OPERATING CONTRACT
Channel: `latest`
Release repository: `khlsilkroad/XAU-Trade-Workspace-Releases`

## Purpose

Make stable app updates operationally simple for the user: approved app changes are promoted to the stable `latest` channel so the installed XAU Trade Workspace can discover the newest approved version through its existing updater path. The user should not need to open `win-unpacked`, copy executables, or reinstall manually for routine approved updates.

## User-facing command

Canonical promotion command:

`PROMOTE XTW STABLE`

The command means: take the current approved master/HEAD, run all required gates, create the next stable version, publish the updater artifacts, verify readback/discovery, and return a release receipt.

## Eligibility policy

An app task is eligible for stable promotion when all are true:

1. Task status is `CLOSED / APPROVED`.
2. Implementation is present on approved `master`/HEAD.
3. Working tree is clean.
4. No unresolved blocker exists.
5. Task is not explicitly marked `NO_RELEASE`.
6. Required regression and post-merge HIGH gates pass.

Never publish directly from an unapproved worker branch.

## Stable promotion pipeline

`CLOSED / APPROVED`
→ resolve approved master/HEAD
→ verify clean state
→ run regression / HIGH gate
→ determine next stable semantic version
→ build Windows stable installer
→ generate updater artifacts
→ verify artifact hashes
→ publish non-prerelease GitHub Release
→ verify release asset readback
→ verify stable updater discovery
→ verify persistence across update
→ emit release receipt
→ `CLOSED / RELEASED`

Any failed gate stops promotion. Prior stable release remains authoritative.

## Required stable artifacts

Every stable release on `latest` must include at least:

- `XAU-Trade-Workspace-Setup-<version>-x64.exe`
- `XAU-Trade-Workspace-Setup-<version>-x64.exe.blockmap`
- `latest.yml`

`latest.yml` must reference the exact stable version and exact published installer metadata. Beta/prerelease artifacts must never overwrite or replace stable `latest.yml`.

## Versioning

- Stable versions use normal semantic versions such as `v2.5.1`, `v2.5.2`, `v2.5.3`.
- Beta versions remain isolated with prerelease identifiers such as `v2.5.2-beta.1`.
- Stable promotion must always move forward; never reuse an existing stable tag/version.
- The release receipt must bind the published version to the exact approved source master commit.

## Mandatory verification gates

Before publish:

- approved source commit verified
- clean working tree
- full required regression PASS
- post-merge HIGH gate PASS
- stable/beta channel isolation PASS
- build PASS
- artifact hash generation PASS

After publish:

- GitHub release exists and is non-prerelease
- exact installer asset present
- exact blockmap asset present
- exact `latest.yml` present
- published asset hashes/readback match expected
- updater discovers N→N+1 using stable channel
- persisted Workspace state survives update/restart

No release may be reported successful before post-publish readback is complete.

## Fail-closed conditions

Promotion must stop and preserve the previous stable release if any of these occur:

- dirty source tree
- source commit mismatch
- unresolved blocker
- regression failure
- HIGH gate failure
- version/tag collision
- missing or mismatched artifact
- hash mismatch
- malformed or mismatched `latest.yml`
- beta/stable channel contamination
- updater discovery failure
- persistence failure
- incomplete release readback

## Release receipt

Every successful stable promotion must produce a receipt containing:

- source master commit
- promoted task(s)
- stable version/tag
- release URL/ID
- installer asset name + SHA256
- blockmap asset name + SHA256
- `latest.yml` asset name + SHA256
- regression result
- HIGH gate result
- channel isolation result
- publish/readback result
- updater discovery result
- persistence result
- working tree final state
- final status: `CLOSED / RELEASED`

## Routine small-update policy

For routine XAU Trade Workspace app changes, the default operational rule is:

`CLOSED / APPROVED` + no blocker + not `NO_RELEASE` ⇒ eligible for stable promotion.

This contract does not auto-authorize a failing or unverified build. Automation may remove repetitive manual steps, but it must not bypass gates.

## Scope boundary

This contract governs application release/promotion only. It must not modify:

- trading semantics
- lifecycle semantics
- XAUUSD canonical history
- Drive source-of-truth rules
- frozen historical packages

## Target user experience

After a successful stable promotion, the user should only need to use the normal XAU Trade Workspace shortcut and the existing updater/restart flow to reach the latest approved stable build. Routine updates must not require the user to remember local build paths or manually launch `win-unpacked` binaries.
