# SBOMlyze Action demo

[![SBOM integrity gate](https://github.com/rezmoss/sbomlyze-action-demo/actions/workflows/sbomlyze.yml/badge.svg)](https://github.com/rezmoss/sbomlyze-action-demo/actions/workflows/sbomlyze.yml)
[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-SBOMlyze%20Diff-blue?logo=github)](https://github.com/marketplace/actions/sbomlyze-diff)

This repository is a minimal, reproducible demonstration of
[SBOMlyze Diff](https://github.com/marketplace/actions/sbomlyze-diff). It shows
how a pull request can pass for an ordinary dependency update and fail when a
component hash changes without a version change.

The workflow is intentionally small and safe:

- SBOMlyze and the other Actions are pinned to full commit SHAs.
- The checkout includes git history so SBOMlyze can read the pull request base.
- No pull-request-provided generator commands are executed.
- Every run writes a Job Summary. SARIF upload is skipped for forked pull
  requests, where GitHub normally provides a read-only token.

## Try the passing example

Create a branch from `main`, replace the tracked SBOM with the reviewed version
update, and open a pull request:

```bash
git switch -c demo/version-update
cp examples/version-update.cdx.json sbom/current.cdx.json
git add sbom/current.cdx.json
git commit -m "demo: update demo-lib to 1.1.0"
git push -u origin demo/version-update
```

The **SBOM integrity gate** should pass and report one changed component with
zero integrity-drift findings. Close this demonstration pull request without
merging it so `main` retains the original `1.0.0` baseline for the next example.

## Try the failing example

Start another branch from `main`, replace the tracked SBOM with the tampered
fixture, and open a pull request:

```bash
git switch main
git switch -c demo/integrity-drift
cp examples/integrity-drift.cdx.json sbom/current.cdx.json
git add sbom/current.cdx.json
git commit -m "demo: simulate same-version hash drift"
git push -u origin demo/integrity-drift
```

The **SBOM integrity gate** should fail while still publishing its outputs and
Job Summary. It reports one integrity-drift finding because `demo-lib` remains
at version `1.0.0` but its SHA-256 hash changes.

## Files

- `.github/workflows/sbomlyze.yml` — the consumer workflow.
- `sbom/current.cdx.json` — the baseline tracked on `main`.
- `examples/version-update.cdx.json` — expected passing pull-request content.
- `examples/integrity-drift.cdx.json` — expected failing pull-request content.

See the [SBOMlyze repository](https://github.com/rezmoss/sbomlyze) for all
inputs, policies, output formats, and installation options.
