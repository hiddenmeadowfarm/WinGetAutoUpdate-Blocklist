README.md v2.0.0 (Last Rev: 2026-10-04)

# WinGetAutoUpdate-Blocklist (Hidden Meadow Farm)

## Overview

Publishes the WinGet-AutoUpdate (WAU) excluded apps list for Hidden Meadow Farm (HMF) PCs. Apps on this list are never auto-updated by WAU.

The published list is the MVTS-Corp master list plus the HMF-specific entries in `custom_blocklist.txt`, merged by a GitHub Actions workflow. HMF PCs download it from:

```
https://raw.githubusercontent.com/HiddenMeadowFarm/WinGetAutoUpdate-Blocklist/main/excluded_apps.txt
```

## Files

| File | Purpose |
|------|---------|
| `excluded_apps.txt` | **Generated. Do not edit by hand.** The list WAU downloads. The name is fixed by WAU and must not change. |
| `custom_blocklist.txt` | HMF-only additions. Edit this file to block an app for HMF only. `#` comment lines and blank lines are allowed and are stripped from the output. |
| `.github/workflows/build-blocklist.yml` | Merges the master list and `custom_blocklist.txt` into `excluded_apps.txt`. Runs on push to `custom_blocklist.txt` or the workflow, daily at 06:00 UTC, and on manual dispatch. |

Master list source: `MVTS-Corp/WinGetAutoUpdate-Blocklist`, file `excluded_apps.txt`, falling back to `blocklist.txt` if that does not exist.

## Quick Start

### Block An App For HMF Only

1. Add the WinGet ID (for example `Valve.Steam`, or a wildcard such as `Google.Chrome*`) on its own line in `custom_blocklist.txt`. A `#` comment line above it explaining why is encouraged.
2. Commit to `main`. The workflow rebuilds `excluded_apps.txt` within a minute or two.
3. PCs pick up the change on their next WAU run.

### Block An App For All MVTS Clients

Add it to the master list in `MVTS-Corp/WinGetAutoUpdate-Blocklist` instead. HMF picks it up on the next daily run, or immediately via Actions > BUILD BLOCKLIST > Run workflow.

## After Install Configuration

WAU policy (GPO) on HMF PCs:

| Setting | Value |
|---------|-------|
| `WAU_ListPath` | `https://raw.githubusercontent.com/HiddenMeadowFarm/WinGetAutoUpdate-Blocklist/main` |
| `WAU_UseWhiteList` | `0` (blacklist mode) |

`WAU_ListPath` must be the **folder**, with no file name on the end. WAU always appends `/excluded_apps.txt` itself.

Check the value a PC actually reads:

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Policies\Romanitho\Winget-AutoUpdate" | Select-Object WAU_ListPath, WAU_UseWhiteList
```

## Troubleshooting

### Logs

- `C:\Program Files\Winget-AutoUpdate\logs\updates.log` - main WAU log.
- `C:\Program Files\Winget-AutoUpdate\logs\error.txt` - present only when the last run failed. This is the real failure signal: the `Winget-AutoUpdate` scheduled task reports `LastTaskResult 0` even when WAU fails.

### Force A Test Run (PowerShell As Admin)

```powershell
gpupdate /force
Start-ScheduledTask -TaskName "Winget-AutoUpdate-Policies"
Start-ScheduledTask -TaskName "Winget-AutoUpdate"
Get-Content "C:\Program Files\Winget-AutoUpdate\logs\updates.log" -Tail 25
Test-Path "C:\Program Files\Winget-AutoUpdate\excluded_apps.txt"
```

Healthy output: `List downloaded/copied to local path`, then `WAU uses Black List config`.

### Common Errors

| Log message | Cause |
|-------------|-------|
| `Couldn't reach/find/compare/copy from ...` followed by `Critical: White/Black List doesn't exist, exiting...` | `<WAU_ListPath>/excluded_apps.txt` returned an error and the PC has no local copy. Check that `WAU_ListPath` is the folder URL, not a file URL, and that the URL above returns the list in a browser. |
| `PATH must end with a Directory, not a File...` | `WAU_ListPath` ends in a `*_apps.txt` file name. Remove it. |
| `0x41303` / `267011` on the UserContext or Notify tasks | Harmless. The task has never run (`WAU_UserContext` is `0`). |

### Workflow Not Running

GitHub disables scheduled workflows after 60 days with no repo activity. The workflow re-enables itself on each scheduled run to prevent this. If it shows as disabled anyway, re-enable it under Actions > BUILD BLOCKLIST > Enable workflow, then run it manually.
