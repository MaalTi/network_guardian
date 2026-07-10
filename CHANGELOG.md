# Changelog

All notable changes to this project are documented in this file.

## [1.1.0] — 2026-05-24

### Changed

- Replaced `notify-send` with `fyi` for desktop notifications.
- Re-bind systemd user service generator to `graphical-session.target` to ensure notifications work in the background.

### Fixed

- Fixed CIDR parsing bug in `is_ignored` function by implementing pure-bash bitwise math, properly supporting sub-octet prefixes (e.g., `/22`).
- Fixed unbound variable issues in array operations (`filter_ignored`, `pending_equals_current`).

### Optimized

- Added `-T4 --max-retries 1` to nmap scans to speed up host discovery.

## [1.0.0] — 2025-03-02

### Added

- Main script `network_guardian`: nmap-based network watchdog.
- **Scan**: `nmap -sn` on a configurable CIDR (default `192.168.1.0/24`).
- **Comparison**: detection of added and removed hosts between scans.
- **Legacy mode**: compare with last scan only (files `current_scan.txt` / `previous_scan.txt`).
- **Telemetry mode**: scan history (`HISTORY_MAX`), debounce with `CONFIRM_SCANS`, `MIN_PRESENCE_SCANS`, `MIN_ABSENCE_SCANS` to confirm adds/removals and reduce false positives.
- **Desktop notifications**: `notify-send` (Linux) and `osascript` (macOS).
- **Logs**: `current_scan.txt`, `previous_scan.txt`, `alerts.log`, `network_guardian.log`; optional XML for Zenmap.
- **Ignore list**: `IGNORE_IPS` variable and `IGNORE_FILE` (IP or CIDR, lines starting with `#` are comments).
- **Config file**: `${XDG_CONFIG_HOME:-$HOME/.config}/network_guardian/config` with log path reload.
- **CLI options**: `--network`, `--config`, `--zenmap`, `--daemon`, `--interval`, `--history-max`, `--install-service`, `--quiet`, `--help`.
- **systemd user service**: `--install-service` creates `~/.config/systemd/user/network-guardian.service` with restart on failure.
- **Compare by IP only**: `COMPARE_BY_IP_ONLY` option (default `true`) to ignore hostname changes.
