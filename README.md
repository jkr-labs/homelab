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
# JKR LAB — Homelab

> A single Lenovo M720q Tiny running Proxmox VE with 23 LXC containers — organized across 7 service categories with full network segmentation, remote access via WireGuard VPN in OPNsense, and HTTPS via Nginx Proxy Manager with wildcard TLS certificates via Cloudflare DNS-01 challenge.

---

## Table of Contents

- [Hardware](#hardware)
- [Network](#network)
- [Storage](#storage)
- [Services](#services)
  - [Gateway](#1-gateway)
  - [Infrastructure](#2-infrastructure)
  - [Security](#3-security)
  - [Monitoring](#4-monitoring)
  - [Media](#5-media)
  - [Acquisition](#6-acquisition)
  - [Media Pipeline](#7-media-pipeline)
  - [Sandbox](#sandbox)
- [Goals & Roadmap](#goals--roadmap)
- [Resources](#resources)

---

## Hardware

| Component | Details |
|---|---|
| **Host** | Lenovo ThinkCentre M720q Tiny |
| **CPU** | Intel Core i5-8400T @ 1.70GHz (6 cores, 1 socket) |
| **RAM** | 16 GB DDR4 2666 MHz |
| **Storage** | 256 GB NVMe M.2 2280 + 1 TB CMR 2.5" SATA |
| **NIC** | Intel i226-V 2.5GbE dual-port (via PCIe riser card) |
| **Hypervisor** | Proxmox VE |
| **Switch** | UniFi USW Flex 2.5GbE |
| **Access Points** | UniFi U7 Lite + UniFi U7 Pro XGS |

---

## Network

| Component | Details |
|---|---|
| **Router / Firewall** | OPNsense (VM 100) |
| **DNS** | AdGuard Home (primary) + Unbound (recursive resolver) |
| **Threat Detection** | CrowdSec + Wazuh agent (on OPNsense) |
| **Reverse Proxy** | Nginx Proxy Manager — wildcard TLS via Cloudflare DNS-01 challenge |
| **Remote Access / VPN** | WireGuard VPN — running inside OPNsense, no separate LXC |
| **Subnet** | `192.168.0.0/16` |

### IP Scheme

Services are organized into subnets by category. CT ID suffix matches the last octet of the IP.

| Subnet | Category |
|---|---|
| `192.168.1.x` | Gateway |
| `192.168.2.x` | Infrastructure |
| `192.168.3.x` | Security |
| `192.168.4.x` | Monitoring |
| `192.168.5.x` | Media |
| `192.168.6.x` | Acquisition |
| `192.168.7.x` | Media Pipeline |

---

## Storage

| Drive | Use |
|---|---|
| 256 GB NVMe M.2 2280 | Proxmox OS + VM/LXC root disks |
| 1 TB CMR 2.5" SATA | Data storage (media, configs, backups) |

---

## Services

### 1. Gateway

> OPNsense with integrated DNS, threat intel, and WireGuard VPN.

| ID | Service | IP |
|---|---|---|
| VM 100 | OPNsense | 192.168.1.1 |
| VM 100 | AdGuard Home | 192.168.1.1 |
| VM 100 | Unbound | 192.168.1.1 |
| VM 100 | WireGuard VPN | 192.168.1.1 |
| CT 113 | Nginx Proxy Manager | 192.168.1.13 |

---

### 2. Infrastructure

> Hypervisor, dashboard, and container management.

| ID | Service | IP |
|---|---|---|
| HOST | Proxmox | 192.168.2.21 |
| CT 222 | Homepage | 192.168.2.22 |
| CT 223 | Portainer | 192.168.2.23 |

---

### 3. Security

> SIEM, IDS/IPS, and threat intelligence.

> **Agent Coverage:** Wazuh agent is installed on all containers. CrowdSec is deployed on public-facing services — NPM (CT 113) and Wazuh (CT 131).

| ID | Service | IP |
|---|---|---|
| CT 131 | Wazuh | 192.168.3.31 |
| — | CrowdSec | https://app.crowdsec.net/security-engines |

---

### 4. Monitoring

> Uptime, metrics, and change tracking.

| ID | Service | IP |
|---|---|---|
| CT 141 | Change Detection | 192.168.4.41 |
| CT 142 | Grafana | 192.168.4.42 |
| CT 143 | Prometheus | 192.168.4.43 |
| CT 144 | Prometheus PVE Exporter | 192.168.4.44 |
| CT 145 | Uptime Kuma | 192.168.4.45 |

---

### 5. Media

> Self-hosted media stack — video, music, and books.

| ID | Service | IP |
|---|---|---|
| CT 151 | Jellyfin | 192.168.5.51 |
| CT 152 | Kavita | 192.168.5.52 |
| CT 153 | Navidrome | 192.168.5.53 |

---

### 6. Acquisition

> Download client and media request management.

| ID | Service | IP |
|---|---|---|
| CT 161 | qBittorrent | 192.168.6.61 |
| CT 162 | Seerr | 192.168.6.62 |

---

### 7. Media Pipeline

> *arr stack for automated media management.

| ID | Service | IP |
|---|---|---|
| CT 171 | Bazarr | 192.168.7.71 |
| CT 172 | Prowlarr | 192.168.7.72 |
| CT 173 | Radarr | 192.168.7.73 |
| CT 174 | Sonarr | 192.168.7.74 |

---

### Sandbox

> Intentionally vulnerable applications for security learning and testing. Running as Docker containers inside Portainer (CT 223) — isolated network access only.

| Service | IP |
|---|---|
| DVGA (Damn Vulnerable GraphQL App) | 192.168.2.23 |
| DVWA (Damn Vulnerable Web App) | 192.168.2.23 |
| OWASP Juice Shop | 192.168.2.23 |
| WebGoat | 192.168.2.23 |

> **Warning:** These are intentionally insecure apps. Never expose them outside the local network.

---

## Goals & Roadmap

- [ ] **Second NVMe slot mod** — Solder missing SMD components onto the M720q board to enable the unpopulated M.2 slot (M.2 SATA only on M720q). Micro-soldering required.
- [ ] **PCIe riser NAS expansion** — Replace current PCIe riser with a powered riser including SATA power output + HBA or NVMe-to-SATA adapter, connected to HDDs in a 3D-printed enclosure attached to the M720q chassis for bulk media and backup storage.
- [ ] **VLAN segmentation** — Reinstall with proper VLANs replacing current subnet-only segmentation for improved network isolation.
- [ ] **Cloudflare Tunnel** — Evaluate exposing select services externally once the security stack is fully hardened.

---

## Resources

### M720q Second NVMe Slot Mod
- [badger707/m920q-dual-NVME — GitHub](https://github.com/badger707/m920q-dual-NVME) — SMD soldering guide; M720q gets M.2 SATA (not NVMe) in the second slot

### M720q PCIe NAS Enclosure
- [6-Bay 10GbE NAS from a Lenovo M720q — Reddit](https://www.reddit.com/r/homelab/comments/1sdmgzm/built_a_6bay_10gbps_nas_from_a_lenovo_m720q/) — Full build guide with parts and 3D printed enclosure

---

*Last updated: June 15, 2026*