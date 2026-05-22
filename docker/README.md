# Docker services

All services run on the Docker VM (`10.0.0.10`) and are managed via Portainer CE.
Each service lives in its own subdirectory with a `docker-compose.yml`.

---

## Services

| Service | Port(s) | Purpose |
|---------|---------|---------|
| [Nginx Proxy Manager](./nginx-proxy-manager/) | 80, 443, 81 | Reverse proxy + SSL termination |
| [Jellyfin](./jellyfin/) | 8096 | Self-hosted media server (GPU transcoding) |
| [Jellyseerr](./jellyseerr/) | 5055 | Media request management |
| [Vaultwarden](./vaultwarden/) | 8080, 3012 | Self-hosted password manager |
| [MinIO](./minio/) | 9000, 9001 | S3-compatible object storage |
| [Homarr](./homarr/) | 7575 | Homelab dashboard |
| [Uptime Kuma](./uptime-kuma/) | 3001 | Service uptime monitoring |
| [Glances](./glances/) | 61208 | System resource monitoring |
| [Watchtower](./watchtower/) | — | Automated container image updates |
| [RustDesk Server](./rustdesk/) | 21115–21119 | Self-hosted remote desktop relay |
| [KasmVNC apps](./kasm/) | 3003–3016 | Browser-based GUI containers |
| [Pterodactyl Panel](./pterodactyl/) | 8090 | Game server management |
| [TeamSpeak](./teamspeak/) | 9987, 10011, 30033 | Voice chat server |
| [IT Tools](./it-tools/) | 8083 | Browser-based IT utility collection |

---

## Secret management

Services with credentials use one of two patterns:

**`.env` file** (preferred — used by Pterodactyl, Homarr, MinIO):
.env lives next to docker-compose.yml — never committed to version control
DB_PASSWORD=your_password_here
Each service with secrets includes a `.env.example` documenting the required variables.

**Vaultwarden** manages its own user authentication internally — no credentials in the Compose file.

---

## Deployment

Each service is deployed independently:

```bash
cd /home/maximo/docker/<service-name>
docker compose up -d
```

Portainer CE provides a management UI and visibility across all running containers.

---

## Reverse proxy

All services are proxied through **Nginx Proxy Manager** with Let's Encrypt SSL certificates.
NPM handles all HTTP/HTTPS ingress on ports 80/443 — internal services are not directly exposed.

---

## Automatic updates

**Watchtower** runs nightly at 3 AM and updates all container images except Portainer,
which is pinned to avoid disrupting the management UI during an active session.
