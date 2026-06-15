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
# JKR LAB — Homepage Dashboard

Self-hosted start page for JKR LAB built with [Homepage](https://gethomepage.dev) — live service status, real-time metrics, and quick access to every container and VM.

![Homepage Dashboard](homepage-dashboard/assets/Homepage-dashboard.jpg)

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
| `hosts` | Internal hostname resolution for widget polling |

---

## How Widget URLs Work

Most Homepage service widgets poll internal hostnames directly to avoid TLS overhead on frequent API calls and keep polling traffic on the LAN.

These hostnames are resolved locally via `/etc/hosts` on the Homepage LXC (CT 222) and are never exposed externally. Nginx Proxy Manager does not proxy them.

**Exceptions — services using direct IPs:**

| Service | Widget URL | Reason |
|---|---|---|
| OPNsense | `https://192.168.1.1` | Direct IP — widget requires HTTPS to OPNsense |
| AdGuard Home | `http://192.168.1.1` | Direct IP — no proxy needed |
| Proxmox | `https://192.168.2.21:8006` | Forces HTTPS internally |
| Portainer | `https://192.168.2.23:9443` | Forces HTTPS internally |

---

## Services

7 categories matching the lab's network segmentation. CT ID suffix matches the last octet of the IP. Most entries pull live data via the service API.

### Gateway
OPNsense with integrated DNS, threat intel, and WireGuard VPN — the edge of the network.

| Service | CT / VM | IP | Role | Widget |
|---|---|---|---|---|
| OPNsense | VM 100 | `192.168.1.1` | Firewall & router | ✅ |
| AdGuard Home | VM 100 | `192.168.1.1` | DNS & ad blocker | ✅ |
| Unbound | VM 100 | `192.168.1.1` | Recursive DNS resolver | — |
| WireGuard VPN | VM 100 | `192.168.1.1` | Remote access VPN | — |
| Nginx Proxy Manager | CT 113 | `192.168.1.13` | Reverse proxy — handles all `*.jkr-lab.ca` subdomains | ✅ |

### Infrastructure
Hypervisor, dashboard, and container management.

| Service | CT / VM | IP | Role | Widget |
|---|---|---|---|---|
| Proxmox | HOST | `192.168.2.21` | Hypervisor — runs all VMs and LXCs | ✅ |
| Homepage | CT 222 | `192.168.2.22` | This dashboard | — |
| Portainer | CT 223 | `192.168.2.23` | Docker container manager | ✅ |

> **Homepage** (`192.168.2.22`) is not listed as a service widget — it is this dashboard itself. Adding it would be circular.

> **Portainer note:** Portainer requires the `--trusted-origins` flag to be passed at startup to allow the dashboard domain to make API calls. Without it, all environment creation and API actions return a `forbidden origin` error.

### Security
SIEM, IDS/IPS, and threat intelligence across the lab.

> **Agent Coverage:** Wazuh agent is installed on all containers. CrowdSec is deployed on public-facing services — NPM (CT 113) and Wazuh (CT 131).

| Service | CT / VM | IP | Role | Widget |
|---|---|---|---|---|
| Wazuh | CT 131 | `192.168.3.31` | SIEM & log analysis | ❌ No Homepage widget type exists |
| CrowdSec | — | cloud-managed | Crowd-sourced threat intelligence | ❌ Local agent API not exposed |

### Monitoring
Full visibility into uptime, metrics, and changes.

| Service | CT / VM | IP | Role | Widget |
|---|---|---|---|---|
| Change Detection | CT 141 | `192.168.4.41` | Monitors websites for changes | ✅ |
| Grafana | CT 142 | `192.168.4.42` | Metrics dashboards | ✅ |
| Prometheus | CT 143 | `192.168.4.43` | Metrics collector | ✅ |
| Prometheus PVE Exporter | CT 144 | `192.168.4.44` | Proxmox metrics scrape target | — |
| Uptime Kuma | CT 145 | `192.168.4.45` | Service uptime monitor | ✅ |

> **Prometheus PVE Exporter** (CT 144) is not listed as a service widget — it is a raw Prometheus scrape target with no dashboard UI or Homepage widget support. It exposes Proxmox metrics at `:9221/metrics` which are scraped by Prometheus and visualized in Grafana.

### Media
Self-hosted media — video, music, books.

| Service | CT / VM | IP | Role | Widget |
|---|---|---|---|---|
| Jellyfin | CT 151 | `192.168.5.51` | Media server | ✅ |
| Kavita | CT 152 | `192.168.5.52` | Comics & manga library | ✅ |
| Navidrome | CT 153 | `192.168.5.53` | Music streaming | ✅ |

### Acquisition
Requesting and downloading media.

| Service | CT / VM | IP | Role | Widget |
|---|---|---|---|---|
| qBittorrent | CT 161 | `192.168.6.61` | Torrent client | ✅ |
| Seerr | CT 162 | `192.168.6.62` | Media request manager | ✅ |

### Media Pipeline
Automated media management — the *arr stack.

| Service | CT / VM | IP | Role | Widget |
|---|---|---|---|---|
| Bazarr | CT 171 | `192.168.7.71` | Subtitle manager | ✅ |
| Prowlarr | CT 172 | `192.168.7.72` | Indexer manager | ✅ |
| Radarr | CT 173 | `192.168.7.73` | Movie manager | ✅ |
| Sonarr | CT 174 | `192.168.7.74` | TV series manager | ✅ |

### Sandbox
Intentionally vulnerable apps for security practice. Running as Docker containers inside Portainer (CT 223) — isolated network access only. **Never expose to WAN.**

| Service | IP | Role | Widget |
|---|---|---|---|
| DVGA | `192.168.2.23` | Damn Vulnerable GraphQL App | ❌ No widget type |
| DVWA | `192.168.2.23` | Damn Vulnerable Web App | ❌ No widget type |
| Juice Shop | `192.168.2.23` | OWASP Juice Shop | ❌ No widget type |
| WebGoat | `192.168.2.23` | OWASP WebGoat | ❌ No widget type |

---

## Known Maintenance Items

> **Cloudflare API Token** — Nginx Proxy Manager uses a Cloudflare DNS-01 challenge for wildcard TLS certificates (`*.jkr-lab.ca`). If the token expires, NPM will fail to renew certificates and all `*.jkr-lab.ca` domains will go down. Check the token expiry in the Cloudflare dashboard and rotate before it expires.

---

*Last updated: June 15, 2026*