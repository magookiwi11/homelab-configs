# NAS pool layout

TrueNAS SCALE configuration reference for the NuggetNAS pool.
TrueNAS SCALE version: **25.10.3.1** (Electric Eel)

---

## Pool

| Setting | Value |
|---------|-------|
| **Pool name** | NuggetNAS |
| **VDEV type** | RAIDZ1 |
| **Drives** | 4x WD 8TB |
| **Raw capacity** | 29.1 TB |
| **Usable capacity** | ~21 TB (3-of-4 drives usable with RAIDZ1) |
| **Connection** | SFF-8087 DAS via SATA controller passthrough (ESXi RDM) |
| **Health** | ONLINE — no errors |
| **Dedup** | 1.00x (disabled — correct for spinning rust at this scale) |

Note on drive identifiers: Drives appear as UUIDs rather than model names because
ESXi RDM passthrough abstracts hardware identifiers from TrueNAS. This is expected
behaviour — ZFS identifies vdev members by UUID regardless.

---

## Datasets

| Dataset | Mountpoint | Purpose |
|---------|-----------|---------|
| `NuggetNAS/media` | `/mnt/NuggetNAS/media` | Jellyfin media library (TV, movies) |
| `NuggetNAS/backups/maxpc` | `/mnt/NuggetNAS/backups/maxpc` | Primary PC backups |
| `NuggetNAS/backups/pc2` | `/mnt/NuggetNAS/backups/pc2` | Secondary PC backups |
| `NuggetNAS/cloud` | `/mnt/NuggetNAS/cloud` | Personal cloud storage |
| `NuggetNAS/gameservers` | `/mnt/NuggetNAS/gameservers` | Game server data / world saves |
| `NuggetNAS/lancache` | `/mnt/NuggetNAS/lancache` | LAN game download cache |
| `NuggetNAS/minio` | `/mnt/NuggetNAS/minio` | MinIO object storage backend |
| `NuggetNAS/surveillance` | `/mnt/NuggetNAS/surveillance` | Security camera footage (planned) |
| `NuggetNAS/appdata` | `/mnt/NuggetNAS/appdata` | Application config and state |

---

## Shares

| Share type | Purpose |
|------------|---------|
| SMB | Windows / general file access |
| NFS | Linux VM access |

---

## Snapshots

ZFS snapshot schedule configured — periodic automatic snapshots for data protection.

---

## Data integrity

- **ECC RAM** in host (64 GB ECC DDR4) — prevents bit-rot from memory errors reaching the pool
- **ZFS checksumming** — detects and corrects silent data corruption at the block level
- **RAIDZ1** — survives single drive failure

---

## Physical enclosure

- **Design:** MASS/MODCASE (Printables)
- **Printer:** Elegoo Centauri Carbon
- **Filament:** Prusament PETG Galaxy Black
- **Power:** ALITOVE 12V brick → picoPSU-80 → drives + fans
- **Drive power control:** Manual rocker switch

---

## ESXi passthrough notes

- SATA controller passed through to TrueNAS VM as RDM — TrueNAS sees and owns the drives natively
- NAS VM allocation: 1 vCPU, 8 GB RAM, 20 GB SSD (OS only — pool lives on the passthrough drives)
- Boot pool runs on a separate internal drive (sda3), not on the NuggetNAS vdev
