# Lunavi Stable — fork maintenance guide

This file lives on the `lunavi-stable` branch only. If you are reading it on `main` after an upstream merge, ignore it — `main` exists to track upstream `Azure/avdaccelerator/main` cleanly.

## Why this fork exists

Flow Control Group (FCG) bicep deployments fetch PowerShell and ARM scripts from this repository at deploy time. Pulling directly from `Azure/avdaccelerator/main` proved brittle — upstream changes (e.g., the April 2025 removal of `-FslogixStorageFqdn` from `Set-SessionHostConfiguration.ps1`) silently broke deploys on the next run.

`lunavi-stable` is the fork's deploy-pin branch. FCG bicep URLs point here. Upstream changes only land on `lunavi-stable` after deliberate review.

## Branch model

| Branch | Tracks | Purpose |
|---|---|---|
| `main` | `Azure/avdaccelerator/main` (1:1) | Upstream tracker. PRs from upstream land here first. |
| `lunavi-stable` | `main` + Lunavi patches | Production deploy-pin. FCG bicep fetches scripts from this branch. Protected. |

## Cadence

- **Quarterly**: routine upstream merge into `lunavi-stable`. Calendar reminder owned by the platform team.
- **On-demand**: security fixes or upstream changes that affect a known FCG deploy path.
- **Never**: direct pushes to `lunavi-stable`. Branch protection enforces PR-only.

## Process for taking upstream changes

```
Azure/main  --PR-->  LunaviCorp/main  --PR-->  LunaviCorp/lunavi-stable
```

1. Sync `LunaviCorp/main` from `Azure/main`. This should be a fast-forward — no conflicts because `main` carries no Lunavi patches.
2. Open a PR `LunaviCorp/main` → `LunaviCorp/lunavi-stable`. Resolve any conflicts against the Lunavi patches. CODEOWNERS auto-tags reviewers (`@lswords` and `@ThojoUno`).
3. Before merging the PR, run a lunavilab full-path test deploy against the candidate `lunavi-stable` HEAD with `configureFslogix=true`. Validation criteria:
   - new session host comes up Available
   - joins the host pool
   - FSLogix profile mounts
   - AMA reports Perf data
   - AVD alerts pack has telemetry
4. Merge.

## Conflict-resolution authority

Default: `@ThojoUno` plus one platform peer (currently `@lswords`). Either can approve dropping a Lunavi patch that upstream has rendered obsolete.

## What lives on `lunavi-stable`

As of the 2026-05-06 rebuild, `lunavi-stable` carries 12 Lunavi patches on top of upstream `main` (commit `63c55c85`). The patches live in `workload/scripts/`:

- `Install-7Zip.ps1`, `Install-NotepadPlusPlus.ps1`, `Install-ODBCDriver18.ps1`, `Install-SSMS.ps1` — net-new installers used by AIB customizations.
- `Remove-OfficeAppx.ps1` — net-new cleanup script for Office consumer apps.
- `Optimize_OS_for_AVD.ps1` — patches the upstream VDOT entrypoint to avoid `UnauthorizedAccessException` during OneDrive cleanup in Packer/AIB builds.

Four upstream-era patches were dropped during the rebuild: three targeted the deleted `Set-VirtualDesktopOptimizations.ps1` (replaced by `Optimize_OS_for_AVD.ps1` in January 2025), and one was an earlier OneDrive fix superseded by the consolidated `Optimize_OS_for_AVD.ps1` patch.

## Decommission of the old personal fork

`ThojoUno/avdaccelerator/scriptfix` was the prior pin source. After FCG bicep migrates to `LunaviCorp/lunavi-stable` (planned in the FCG-side `docs/planning/lunavi-stable-plan.md`), the personal fork's `scriptfix` branch is deprecated. Do not pull from it.
