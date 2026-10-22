# server-stack

A Docker Compose monolith for a homelab: 30+ containers across networking, media, monitoring, and home-automation profiles.

## patch_containers

`patch_containers` is an idempotent update script that iterates over every running container, detects its OS distro (alpine / debian / ubuntu), upgrades packages inside it, and restarts it — ordered by service dependency priority. It runs in the background so a single failure does not halt the batch.

### Restart order (defined in `DEPENDENT_CONTAINERS`)

The script patches **first** in dependency order, then falls back to all remaining containers:

| Priority (patched first) | Dependents (restarted after priority restarts) |
|---|---|
| `gluetun` | `qbittorrent`, `qbittorrent-age-tagger` |
| `mariadb` | `homeassistant`, `kodi` |
| `mosquitto` | `zigbee2mqtt` |
| `victoriametrics` | `grafana` |
| `victorialogs` | `grafana` |

This ordering matters: a priority container is patched **before** its dependents, and once it restarts the dependent(s) follow immediately.

### Distroless containers

Containers matching `victoria*` or `cloudflare*` are detected as distroless (no `/etc/os-release`, no package manager) and are skipped entirely — they receive only an informational log line.

### Known exceptions: non-root / unpatchable images

Some linuxserver-derived containers run their entrypoint under a non-root user and lack a writable apk/apt database. The script detects this with a privilege check (`id -u` + `/lib/apk/db` writability, `whoami` + `sudo` availability for apt) and emits an error message but **does not crash**. Affected containers:

- `grafana` (runs as `nobody`, no apk)
- `seerr` (same pattern)
- `unpackerr` (same pattern)
- `bgg` (no root/sudo, no apt)
- `qbittorrent-age-tagger` (same pattern)

These will appear in output with one of the following patterns:
```
'grafana' cannot run 'apk': not root or apk database not writable
'bgg' cannot run 'apt': not root or sudo not available
```
This is expected — do not assume a crash. The script simply moves on to the next container.

### Error handling model

The script deliberately **does not use `set -e`** (no `errexit`). Each patch operation wraps failures in an `if / else / || echo "no"` pattern so that:
- A failed update does not terminate the entire batch.
- Failed containers log an error line and the loop continues.
- The user must check the output for per-container status; there is no aggregate pass/fail exit code.

### Important notes

- **Do not change `IFS`**. The script relies on default space/tab/newline splitting in `for … in ${DEPENDENT_CONTAINERS[...]}  to expand multi-word values like `"homeassistant kodi"`. Setting IFS to `$'\n\t'` would prevent the space split and cause the restart chain to target a non-existent container.
- **The script has no dry-run mode**. Every detected update is applied in-place and every patched container is restarted immediately.
- **It operates only on running containers** (`docker ps`), not stopped or exited ones.
