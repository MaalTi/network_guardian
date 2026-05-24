# Network Guardian

**Home network watchdog**: nmap scan, telemetry, desktop notifications. Detects devices added or removed on your local network and alerts you.

## Features

- **nmap scan** (`nmap -sn`) on a configurable subnet (CIDR)
- **Comparison** between scans to detect **added** and **removed** hosts
- **Telemetry mode** (optional): scan history + debounce to reduce false positives (N consecutive scans to confirm a change)
- **Desktop notifications**: `fyi` (Linux) or `osascript` (macOS)
- **Log files**: current scan, previous scan, alerts, main log; optional XML output for Zenmap
- **Ignore list**: IP or CIDR ranges to exclude from alerts
- **systemd service** (user) for background execution at session startup

## Requirements

- **Bash** (with `pipefail`)
- **nmap** (required)

For notifications:

- Linux: `fyi` (https://codeberg.org/dnkl/fyi)
- macOS: `osascript`

## Installation

1. Make the script executable:

   ```bash
   chmod +x network_guardian
   ```

2. (Optional) Add it to your `PATH` or call it by full path.

## Usage

### One-shot (or cron)

```bash
./network_guardian [options]
```

A single scan is run, compared to the previous one (or to history if enabled), and an alert is sent if changes are detected.

### Daemon mode (loop)

```bash
./network_guardian --daemon --interval=120
```

Scans run every 120 seconds (or the value of `INTERVAL` in config).

### Options

| Option              | Description                                                       |
|---------------------|-------------------------------------------------------------------|
| `--network=CIDR`    | Subnet to scan (default: `192.168.1.0/24`)                        |
| `--config=FILE`     | Config file to load                                               |
| `--zenmap`          | Open Zenmap with target or last XML result after scan             |
| `--daemon`, `-d`    | Daemon mode (loop with `--interval`)                              |
| `--interval=SECS`   | Interval between scans in seconds (daemon mode)                   |
| `--history-max=N`   | Number of scans kept in history (`0` = legacy mode, default: `0`) |
| `--install-service` | Create `~/.config/systemd/user/network-guardian.service`          |
| `--quiet`, `-q`     | Fewer messages on standard output                                 |
| `--help`, `-h`      | Show this help                                                    |

## Configuration

Default config file:  
`${XDG_CONFIG_HOME:-$HOME/.config}/network_guardian/config`

Variables (export or set in that file):

| Variable             | Description                                         | Default            |
|----------------------|-----------------------------------------------------|--------------------|
| `NETWORK`            | CIDR network to scan                                | `192.168.1.0/24`   |
| `LOG_DIR`            | Log directory                                       | `$HOME/.nmap_logs` |
| `QUIET`              | Reduce output                                       | `false`            |
| `INTERVAL`           | Interval in seconds (daemon)                        | —                  |
| `HISTORY_MAX`        | Number of scans in history (`0` = legacy)           | `0`                |
| `CONFIRM_SCANS`      | Consecutive scans required to confirm an alert      | `2`                |
| `MIN_PRESENCE_SCANS` | Scans where IP must be seen to count as “added”     | `2`                |
| `MIN_ABSENCE_SCANS`  | Scans where IP must be absent to count as “removed” | `2`                |
| `COMPARE_BY_IP_ONLY` | Compare by IP only (otherwise hostname+IP)          | `true`             |
| `IGNORE_IPS`         | Array of IPs to ignore                              | —                  |
| `IGNORE_FILE`        | File with one IP or CIDR per line (`#` = comment)   | —                  |

Example `config`:

```bash
NETWORK="192.168.1.0/24"
LOG_DIR="$HOME/.nmap_logs"
INTERVAL=120
HISTORY_MAX=20
CONFIRM_SCANS=2
MIN_PRESENCE_SCANS=2
MIN_ABSENCE_SCANS=2
COMPARE_BY_IP_ONLY=true
IGNORE_FILE="$HOME/.config/network_guardian/ignore.txt"
```

## Log Files

In `LOG_DIR` (default `~/.nmap_logs`):

- `current_scan.txt` — latest nmap result (text)
- `previous_scan.txt` — previous scan (legacy mode)
- `current_scan.xml` — latest nmap result (XML, if used)
- `alerts.log` — alert history (adds/removals)
- `network_guardian.log` — main script log
- `history/` — historical scans (if `HISTORY_MAX` > 0)
- `.pending_alert` — alert pending confirmation (telemetry)

## systemd Service (user)

Create the service:

```bash
./network_guardian --install-service
```

Then:

```bash
systemctl --user daemon-reload
systemctl --user start network-guardian
systemctl --user enable network-guardian   # at session startup
systemctl --user status network-guardian
```

Unit file: `~/.config/systemd/user/network-guardian.service`. Interval can be set via `Environment=INTERVAL=...` in the unit or via the config file. Note that the service binds to `graphical-session.target` to ensure notifications work.

## License

Script provided as-is. Check nmap usage terms on your network.
