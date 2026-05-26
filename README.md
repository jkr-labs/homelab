# JKR LAB — Homelab

> A single Lenovo M720q Tiny running Proxmox VE with 2 VMs and 20 LXC containers — organized across 8 service categories with full network segmentation, remote access via Tailscale, and local HTTPS via Caddy with automatic TLS certificates via Cloudflare DNS-01 challenge.

---

## Table of Contents

- [Hardware](#hardware)
- [Network](#network)
- [Storage](#storage)
- [Services](#services)
  - [Gateway](#1-gateway)
  - [Control Plane](#2-control-plane)
  - [Infrastructure](#3-infrastructure)
  - [Security](#4-security)
  - [Observability](#5-observability)
  - [Media](#6-media)
  - [Acquisition](#7-acquisition)
  - [Media Pipeline](#8-media-pipeline)
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
| **Threat Detection** | CrowdSec + Wazuh client (on OPNsense) |
| **Reverse Proxy** | Caddy with xcaddy — automatic TLS via Cloudflare DNS-01 challenge |
| **Remote Access / VPN** | Tailscale — OPNsense configured as exit node. No Cloudflare Tunnel — local network only |
| **Subnet** | `192.168.0.0/16` |

### IP Scheme

Services are organized into subnets by category within the `192.168.0.0/16` network, with the last octet matching the container ID suffix:

| Subnet | Category |
|---|---|
| `192.168.1.x` | Gateway |
| `192.168.2.x` | Control Plane |
| `192.168.3.x` | Infrastructure |
| `192.168.4.x` | Security |
| `192.168.5.x` | Observability |
| `192.168.6.x` | Media |
| `192.168.7.x` | Acquisition |
| `192.168.8.x` | Media Pipeline |
| `192.168.9.x` | IoT & Wireless |

---

## Storage

| Drive | Use |
|---|---|
| 256 GB NVMe M.2 2280 | Proxmox OS + VM/LXC root disks |
| 1 TB CMR 2.5" SATA | Data storage (media, configs, backups) |

---

## Services

### 1. Gateway

> OPNsense with integrated DNS, reverse proxy, and threat intel.

| ID | Service | IP |
|---|---|---|
| VM 100 | OPNsense | 192.168.1.1 |
| VM 100 | AdGuard Home | 192.168.1.1 |
| VM 100 | Unbound | 192.168.1.1 |
| CT 111 | Caddy | 192.168.1.11 |

---

### 2. Control Plane

> Proxmox host, Home Assistant, and the dashboard.

| ID | Service | IP |
|---|---|---|
| HOST | Proxmox | 192.168.2.22 |
| VM 200 | Home Assistant OS | 192.168.2.1 |
| CT 222 | Homepage | 192.168.2.2 |

---

### 3. Infrastructure

> Authentication, container management, and remote access.

| ID | Service | IP |
|---|---|---|
| CT 131 | Authentik | 192.168.3.31 |
| CT 132 | Portainer | 192.168.3.32 |
| — | Tailscale (web-managed) | https://login.tailscale.com |

---

### 4. Security

> SIEM, IDS/IPS, and threat intelligence.

> **Agent Coverage:** Wazuh agent is installed on all containers. CrowdSec is deployed selectively on public-facing and authentication services — Caddy (CT 111), Authentik (CT 131), and Wazuh (CT 141).

| ID | Service | IP |
|---|---|---|
| CT 141 | Wazuh | 192.168.4.41 |
| — | CrowdSec | https://app.crowdsec.net/security-engines |

---

### 5. Observability

> Uptime, metrics, and change tracking.

| ID | Service | IP |
|---|---|---|
| CT 151 | Change Detection | 192.168.5.51 |
| CT 152 | Grafana | 192.168.5.52 |
| CT 153 | Prometheus | 192.168.5.53 |
| CT 154 | Prometheus PVE Exporter | 192.168.5.54 |
| CT 155 | Uptime Kuma | 192.168.5.55 |

---

### 6. Media

> Self-hosted media stack — photos, video, music, books.

| ID | Service | IP |
|---|---|---|
| CT 161 | Immich | 192.168.6.61 |
| CT 162 | Jellyfin | 192.168.6.62 |
| CT 163 | Kavita | 192.168.6.63 |
| CT 164 | Navidrome | 192.168.6.64 |

---

### 7. Acquisition

> Download client and media request management.

| ID | Service | IP |
|---|---|---|
| CT 171 | qBittorrent | 192.168.7.71 |
| CT 172 | Jellyseerr | 192.168.7.72 |

---

### 8. Media Pipeline

> *arr stack for automated media management.

| ID | Service | IP |
|---|---|---|
| CT 181 | Bazarr | 192.168.8.81 |
| CT 182 | Kapowarr | 192.168.8.82 |
| CT 183 | Lidarr | 192.168.8.83 |
| CT 184 | Prowlarr | 192.168.8.84 |
| CT 185 | Radarr | 192.168.8.85 |
| CT 186 | Sonarr | 192.168.8.86 |

---

### Sandbox

> Intentionally vulnerable applications for security learning and testing. Running as Docker containers inside Portainer (CT 132) — isolated network access only.

| Service | IP |
|---|---|
| DVGA (Damn Vulnerable GraphQL App) | 192.168.3.32 |
| DVWA (Damn Vulnerable Web App) | 192.168.3.32 |
| OWASP Juice Shop | 192.168.3.32 |
| WebGoat | 192.168.3.32 |

> **Warning:** These are intentionally insecure apps. Never expose them outside the local network.

---

## Goals & Roadmap

- [ ] **Second NVMe slot mod** — Solder missing SMD components onto the M720q board to enable the unpopulated M.2 slot (M.2 SATA only on M720q). Micro-soldering required.
- [ ] **PCIe riser NAS expansion** — Replace current PCIe riser with a powered riser including SATA power output + HBA or NVMe-to-SATA adapter, connected to HDDs in a 3D-printed enclosure attached to the M720q chassis for bulk media and backup storage.
- [ ] **VLAN segmentation** — Reinstall with proper VLANs replacing current subnet-only segmentation for network redundancy and isolation.
- [ ] **Cloudflare Tunnel** — Evaluate exposing select services externally once the security stack is fully hardened.

---

## Resources

### M720q Second NVMe Slot Mod
- [badger707/m920q-dual-NVME — GitHub](https://github.com/badger707/m920q-dual-NVME) — SMD soldering guide; M720q gets M.2 SATA (not NVMe) in the second slot

### M720q PCIe NAS Enclosure
- [6-Bay 10GbE NAS from a Lenovo M720q — Reddit](https://www.reddit.com/r/homelab/comments/1sdmgzm/built_a_6bay_10gbps_nas_from_a_lenovo_m720q/) — Full build guide with parts and 3D printed enclosure

---

*Last updated: May 24, 2026*
