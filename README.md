# homelab-configs

Personal homelab running on a **Qotom Q20331G9-S10** — a fanless, purpose-built mini server. This repo documents the architecture, configuration, and deployment files for the full stack.

---

## Hardware

| Component | Spec |
|-----------|------|
| **Host** | Qotom Q20331G9-S10 |
| **CPU** | Intel Atom C3758R — 8 cores / 8 threads @ 2.4 GHz, 26W TDP |
| **RAM** | 64 GB ECC DDR4 |
| **Boot drive** | 1 TB NVMe M.2 Gen3 (in use) + 1x M.2 free |
| **Additional storage** | 2x SATA 3.0 ports |
| **Network** | 4x 10GbE SFP+ · 5x 2.5G RJ45 · 1x SFF-8087 MiniSAS · 1x RS-232 |
| **Form factor** | Fanless / passive cooling |

**NAS expansion:** 4x WD 8TB drives in a 3D-printed enclosure connected via SFF-8087 DAS.
Power: ALITOVE 12V brick → picoPSU-80 → drives and fans.

---

## Hypervisor: VMware ESXi

All workloads run as VMs on ESXi. Autostart order: OPNsense → TrueNAS → Docker VM.

### VM layout

| VM | vCPU | RAM | Storage | OS / Role |
|----|------|-----|---------|-----------|
| **OPNsense** | 2 | 4 GB | 64 GB SSD | Firewall, router, VPN |
| **Docker VM** | 2 | 20 GB | 150 GB SSD | Ubuntu Server 22.04 — containerized services |
| **NAS VM** | 1 | 8 GB | 20 GB SSD | TrueNAS SCALE — NAS storage |
| **Game Server VM** | 3 | 12 GB | 100 GB SSD | Ubuntu Server 22.04 — Pterodactyl |

### Static IP scheme

| Host | IP |
|------|----|
| OPNsense (LAN gateway) | `10.0.0.1` |
| ESXi host | `10.0.0.50` |
| TrueNAS SCALE | `10.0.0.60` |
| Docker VM | `10.0.0.10` |

---

## Networking: OPNsense

- **WAN:** AT&T BGW320-505 in IP Passthrough mode (DHCPS-fixed, OPNsense WAN MAC)
- **LAN:** `10.0.0.0/24` subnet
- **DHCP:** ISC DHCPv4 (static leases for all hosts above)
- **DNS:** Dnsmasq
- **Firewall:** Custom ruleset with VLAN segmentation
- **IDS/IPS:** Enabled
- **VPN:** Configured
- **QAT offload:** Enabled (hardware crypto acceleration via C3758R)
- **SSL cert:** Let's Encrypt via `os-acme-client` + DuckDNS
- **Switching:** 10GbE SFP+ for inter-VM traffic; 2.5G RJ45 for client devices

---

## Storage: TrueNAS SCALE

- **Pool:** NuggetNAS — RAIDZ1 across 4x WD 8TB drives
- **Drive connection:** RDM via SATA controller passthrough from ESXi
- **Shares:** SMB and NFS configured
- **Snapshots:** ZFS snapshot schedule configured
- **Data integrity:** ECC RAM + ZFS checksumming

---

## Docker VM services

All containerized services run on the Docker VM (`10.0.0.10`).
Managed via **Portainer CE**. Deployed with Docker Compose.

See [`/docker/`](./docker/) for individual Compose files.

| Service | Description |
|---------|-------------|
| **Portainer CE** | Container management UI |
| **Nginx Proxy Manager** | Reverse proxy + SSL termination |
| **Jellyfin** | Self-hosted media server (GPU transcoding) |
| **Jellyseerr** | Media request management |
| **Vaultwarden** | Self-hosted password manager |
| **MinIO** | S3-compatible object storage |
| **Homarr** | Homelab dashboard |
| **Uptime Kuma** | Service uptime monitoring |
| **Glances** | System resource monitoring |
| **Watchtower** | Automated container image updates |
| **RustDesk Server** | Self-hosted remote desktop relay |
| **KasmVNC apps** | Browser-based GUI containers |
| **Pterodactyl Panel** | Game server management |
| **TeamSpeak** | Voice chat server |
| **IT Tools** | Browser-based IT utility collection |

---

## Game Server VM

- **OS:** Ubuntu Server 22.04 LTS
- **Panel:** Pterodactyl Wings daemon
- **Active servers:** Minecraft Java (Paper MC, Java 21, Aikar's JVM flags), Project Zomboid, CS2
- **Note:** One game server active at a time given resource constraints

---

## AI Stack

Running on a separate dedicated workstation (not the Qotom). See [`AI-STACK.md`](./AI-STACK.md) for full details.

**Quick summary:** Self-hosted multi-model LLM environment on dual-RTX 5090 hardware (64 GB GDDR7 VRAM, 128 GB DDR5) using Ollama + AnythingLLM with RAG pipelines, Fabric prompt orchestration, and MCP multi-agent tool framework.

---

## Repo structure
```
homelab-configs/
├── README.md
├── AI-STACK.md
├── docker/
│   ├── README.md
│   ├── jellyfin/
│   ├── glances/
│   ├── jellyseerr/
│   ├── watchtower/
│   ├── vaultwarden/
│   ├── homarr/
│   ├── teamspeak/
│   ├── rustdesk/
│   ├── nginx-proxy-manager/
│   ├── minio/
│   ├── it-tools/
│   ├── uptime-kuma/
│   ├── pterodactyl/
│   └── kasm/
├── nas/
│   └── pool-layout.md
└── network/
└── opnsense-notes.md

```

---

## Notes

- OPNsense IP Passthrough resets after BGW320-505 reboots. Recovery: set static IP on client, access `192.168.1.254`, reapply DHCPS-fixed with OPNsense WAN MAC, renew WAN lease on OPNsense.
- ESXi autostart ensures OPNsense comes up before any other VM so network is available on boot.
- NAS enclosure designed and printed on Elegoo Centauri Carbon with Prusament PETG Galaxy Black (MASS/MODCASE Printables design).

---

*Work in progress — configs being added incrementally.*
