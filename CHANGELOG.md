# Changelog

All notable changes to this repo are documented here. Newest first.

## [2.1.0] - 2026-10-09

### Removed
- `RaspberryPiFoundation.RaspberryPiImager` from `custom_blocklist.txt`, as a trial to see whether auto-updating it causes an issue.

### Added
- Reason comments for every entry in `custom_blocklist.txt`, and a "Why Each App Is Blocked" table in README.md covering master and HMF entries.

## [2.0.0] - 2026-10-04

### Fixed
- WAU on HMF PCs exited on every run with `White/Black List doesn't exist` (no updates since at least 2026-09-23). WAU requests `<WAU_ListPath>/excluded_apps.txt`, but the repo published `blocklist.txt` and the policy pointed at that file. The workflow now publishes `excluded_apps.txt`, and `WAU_ListPath` was changed in GPO to the folder URL.
- Scheduled workflow had been disabled by GitHub for inactivity since 2026-06-29, so master list changes stopped reaching HMF. The workflow now re-enables itself on each scheduled run.

### Changed
- Output file renamed from `blocklist.txt` to `excluded_apps.txt`; `blocklist.txt` removed.
- Master list fetch tries `excluded_apps.txt` first, then `blocklist.txt`, with retries and a timeout.
- Build now strips CR and whitespace, de-duplicates, and refuses to publish if the master list is empty or any entry is not a valid WinGet ID pattern.
- Push trigger limited to `main`; added a concurrency group.

### Added
- README.md, CLAUDE.md, CHANGELOG.md.

## [1.0.0] - 2026-04-29

### Added
- Initial custom blocklist and BUILD BLOCKLIST workflow producing `blocklist.txt`.
