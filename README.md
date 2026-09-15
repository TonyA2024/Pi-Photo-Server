# Self-Hosted Photo Server on Raspberry Pi 5

A privately-hosted, self-managed alternative to Google Photos / iCloud, built on a Raspberry Pi 5 running [Immich](https://immich.app) in Docker, with secure remote access from any network via Tailscale.

## Overview

This project replaces cloud photo storage with a personal server that I fully own and control. Photos are stored on a locally-attached SSD, indexed and served by Immich (an open-source photo management platform), and accessible from my phone or desktop from anywhere — not just my home network — without exposing any ports to the public internet.

## Architecture

```
┌─────────────┐        Tailscale (WireGuard-based)       ┌──────────────────────┐
│  Phone / PC  │ ───────────── encrypted tunnel ────────▶ │   Raspberry Pi 5 (8GB) │
└─────────────┘                                           │                       │
                                                            │  ┌─────────────────┐  │
                                                            │  │ Docker Compose  │  │
                                                            │  │  ├─ immich-server│  │
                                                            │  │  ├─ postgres     │  │
                                                            │  │  ├─ redis        │  │
                                                            │  │  └─ machine-     │  │
                                                            │  │     learning     │  │
                                                            │  └─────────────────┘  │
                                                            │         │             │
                                                            │   1TB USB SSD (ext4)  │
                                                            │   mounted at /mnt/    │
                                                            │   photos, persistent  │
                                                            │   via /etc/fstab      │
                                                            └──────────────────────┘
```

## Hardware

| Component | Spec |
|---|---|
| Board | Raspberry Pi 5, 8GB RAM |
| Storage | 1TB USB 3.0 SSD (ext4), mounted at `/mnt/photos` |
| Cooling | Active cooling case |
| Networking | Gigabit Ethernet |

## Software Stack

- **OS:** Raspberry Pi OS (64-bit)
- **Containerization:** Docker + Docker Compose
- **Photo management:** [Immich](https://github.com/immich-app/immich) (server, ML/facial-recognition worker, PostgreSQL, Redis)
- **Remote access:** [Tailscale](https://tailscale.com) (WireGuard-based mesh VPN)
- **Auth:** SSH key-based authentication (password auth disabled)

## What This Project Demonstrates

- **Linux system administration** — disk partitioning/formatting (`mkfs.ext4`), persistent mount configuration via `/etc/fstab` with UUID referencing, systemd service management
- **Containerization** — multi-container orchestration with Docker Compose (app server, database, cache, and ML worker running as separate coordinated services)
- **Networking & security** — key-based SSH authentication, private mesh VPN (Tailscale) for remote access instead of exposing ports directly to the internet, principle of least exposure
- **Data migration** — bulk import of an existing ~80GB photo library, with folder-structure independence handled via EXIF-based indexing
- **Infrastructure-as-config** — environment-based configuration (`.env`), version-controlled deployment setup

## Setup Summary

1. Flash/boot Raspberry Pi OS, enable SSH (key-based only)
2. Partition and format external SSD to ext4, mount persistently via `/etc/fstab` (UUID-referenced, `nofail` for boot resilience)
3. Install Docker; deploy Immich via `docker-compose.yml` pointed at the SSD mount for storage
4. Import existing photo library (bulk upload via Immich web client)
5. Install and configure Tailscale on the Pi and all client devices for zero-port-forwarding remote access
6. Harden SSH (key-based auth only, password auth disabled)

See `setup-notes.md` for the detailed command-by-command walkthrough.

## Future Improvements

- [ ] Scheduled backup job to a secondary drive (redundancy beyond the primary SSD)
- [ ] Automated Docker image updates with a maintenance window
- [ ] Monitoring/alerting for disk usage and container health
