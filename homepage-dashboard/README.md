```
▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓
▓▓                                                                            ▓▓
▓▓                 ██╗██╗  ██╗██████╗     ██╗      █████╗ ██████╗             ▓▓
▓▓                 ██║██╔╝██╔╝██╔══██╗    ██║     ██╔══██╗██╔══██╗            ▓▓
▓▓                 ██║████╔╝  ██████╔╝    ██║     ███████║██████╔╝            ▓▓
▓▓            ██   ██║██╔╝██╗ ██╔══██╗    ██║     ██╔══██║██╔══██╗            ▓▓
▓▓            ╚█████╔╝██║  ██╗██║  ██║    ███████╗██║  ██║██████╔╝            ▓▓
▓▓             ╚════╝ ╚═╝  ╚═╝╚═╝  ╚═╝    ╚══════╝╚═╝  ╚═╝╚═════╝             ▓▓
▓▓                                                                            ▓▓
▓▓         jkr-lab.ca  |  Proxmox  |  Self-Hosted  |  Always Breaking         ▓▓
▓▓                                                                            ▓▓
▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓
```
Self-hosted start page for JKR LAB built with [Homepage](https://gethomepage.dev). Single pane of glass for the entire lab — live service status, real-time metrics, and quick access to every container and VM.

For setup refer to the [official Homepage documentation](https://gethomepage.dev).

> Secrets are managed via `.env` and never committed to this repository.

---

## Configuration Files

| File | Purpose |
|---|---|
| `services.yaml` | Service definitions, icons, links, and live API widgets |
| `settings.yaml` | Layout, column structure, appearance, and theme |
| `widgets.yaml` | Top-level info bar — greeting, live weather, search, date/time |
| `bookmarks.yaml` | Quick-access external links — dev resources, security tools, career |
| `hosts` | Internal hostname resolution for `.wgt` widget polling domain |

---

## How Widget URLs Work

Most Homepage service widgets poll a separate internal domain — `*.jkr-lab.wgt` — instead of the public `*.jkr-lab.ca` URLs. This avoids TLS overhead on frequent widget API calls and keeps polling traffic on the LAN.

These hostnames are resolved locally via `/etc/hosts` on the Homepage LXC and are never exposed externally. Caddy does not proxy them.

**Exceptions — services that don't use `.wgt`:**

| Service | Widget URL | Reason |
|---|---|---|
| OPNsense | `https://192.168.1.1` | Direct IP — widget requires HTTPS to OPNsense |
| AdGuard | `http://192.168.1.1:8080` | Direct IP — no `.wgt` entry needed |
| Proxmox | `https://192.168.2.22:8006` | Forces HTTPS internally |
| Home Assistant | `http://192.168.2.1:8123` | Direct IP — runs plain HTTP, no `.wgt` entry |
| Portainer | `https://192.168.3.32:9443` | Forces HTTPS internally |
| Tailscale | External Tailscale API | Cloud-managed, no internal URL |

---

## Services

9 categories matching the lab's network segmentation. Most entries pull live data via the service API.

### Gateway
Firewall, DNS filtering, and reverse proxy — the edge of the network.

| Service | IP | Role | Widget |
|---|---|---|---|
| OPNsense | `192.168.1.1` | Firewall & router | ✅ |
| AdGuard Home | `192.168.1.1:8080` | DNS & ad blocker | ✅ |
| Caddy | `192.168.2.2` | Reverse proxy — handles all `*.jkr-lab.ca` subdomains | ❌ Admin API is localhost only |

### Control Plane
Primary interfaces for managing the hypervisor and home automation.

| Service | IP | Role | Widget |
|---|---|---|---|
| Proxmox | `192.168.2.22` | Hypervisor — runs all VMs and LXCs | ✅ |
| Home Assistant | `192.168.2.1` | Home automation | ✅ |

> **Homepage** (`192.168.2.2`) is not listed as a service — it is this dashboard itself. Adding it would be circular.

### Infrastructure
Identity, containers, and remote access.

| Service | IP | Role | Widget |
|---|---|---|---|
| Authentik | `192.168.3.31` | SSO & identity provider | ✅ |
| Portainer | `192.168.3.32` | Docker container manager | ✅ |
| Tailscale | External | Remote access VPN — web managed | ✅ |

> **Portainer note:** Portainer requires the `--trusted-origins` flag to be passed at startup to allow the dashboard domain to make API calls. Without it, all environment creation and API actions return a `forbidden origin` error. See the Portainer container run command in the infrastructure notes.

### Security
Threat detection and intelligence across the lab.

| Service | IP | Role | Widget |
|---|---|---|---|
| Wazuh | `192.168.4.41` | SIEM & log analysis | ❌ No Homepage widget type exists |
| CrowdSec | External | Crowd-sourced threat intelligence | ❌ Local agent API not exposed |

### Observability
Full visibility into uptime, metrics, and changes.

| Service | IP | Role | Widget |
|---|---|---|---|
| Change Detection | `192.168.5.51` | Monitors websites for changes | ✅ |
| Grafana | `192.168.5.52` | Metrics dashboards | ✅ |
| Prometheus | `192.168.5.53` | Metrics collector | ✅ |
| Uptime Kuma | `192.168.5.55` | Service uptime monitor | ✅ |

> **Prometheus PVE Exporter** (`192.168.5.54`) is not listed as a service — it is a raw Prometheus scrape target with no dashboard UI or Homepage widget support. It exposes Proxmox metrics at `:9221/metrics` which are scraped by Prometheus and visualized in Grafana.

### Media
Self-hosted media — photos, video, music, books.

| Service | IP | Role | Widget |
|---|---|---|---|
| Immich | `192.168.6.61` | Photo & video library | ✅ |
| Jellyfin | `192.168.6.62` | Media server | ✅ |
| Kavita | `192.168.6.63` | Comics & manga library | ✅ |
| Navidrome | `192.168.6.64` | Music streaming | ✅ |

### Acquisition
Requesting and downloading media.

| Service | IP | Role | Widget |
|---|---|---|---|
| qBittorrent | `192.168.7.71` | Torrent client | ✅ |
| Seerr | `192.168.7.72` | Media request manager | ✅ |

### Media Pipeline
Automated media management — the *arr stack.

| Service | IP | Role | Widget |
|---|---|---|---|
| Bazarr | `192.168.8.81` | Subtitle manager | ✅ |
| Kapowarr | `192.168.8.82` | Comics & manga manager | ✅ |
| Lidarr | `192.168.8.83` | Music manager | ✅ |
| Prowlarr | `192.168.8.84` | Indexer manager | ✅ |
| Radarr | `192.168.8.85` | Movie manager | ✅ |
| Sonarr | `192.168.8.86` | TV series manager | ✅ |

### Sandbox
Intentionally vulnerable apps for security practice. Isolated from the internet via OPNsense firewall rules — **never expose to WAN.**

| Service | IP | Role | Widget |
|---|---|---|---|
| DVGA | `192.168.3.32:5013` | Damn Vulnerable GraphQL API | ❌ No widget type |
| DVWA | `192.168.3.32:8081` | Damn Vulnerable Web App | ❌ No widget type |
| Juice Shop | `192.168.3.32:3000` | OWASP Juice Shop | ❌ No widget type |
| WebGoat | `192.168.3.32:8080` | OWASP WebGoat | ❌ No widget type |

---

## Known Maintenance Items

> **Cloudflare API Token** — Caddy uses a Cloudflare DNS challenge for wildcard TLS certificates (`*.jkr-lab.ca`). The token is stored in `/etc/caddy/caddy.env` on the Caddy LXC as `CF_API_TOKEN`. If the token expires, Caddy will fail to renew certificates and all `*.jkr-lab.ca` domains will go down. Check the token expiry in the Cloudflare dashboard and rotate before it expires.

---

*Last updated: May 25, 2026*