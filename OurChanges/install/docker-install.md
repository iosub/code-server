# Docker Installation Guide (Isolated) for This VPS

## Source of Truth in This Repository
This guide follows the standard Docker installation approach documented in:
- docs/install.md (Docker section)

The commands below keep that baseline and adapt it for isolated operation (no impact on existing VS Code/VS Code Server services).

## Objective
Run a separate code-server instance on this VPS using Docker, with:
- isolated port
- isolated container name
- isolated config/data directories
- explicit mount for your local VPS repositories

## 1) Pre-checks
Run these checks first:

```bash
ss -ltnp
docker --version
docker image ls | head
```

Pick a free local port (example used in this guide: 18080).

## 2) Create Isolated Host Directories

```bash
sudo mkdir -p /opt/code-server-isolated/config
sudo mkdir -p /opt/code-server-isolated/data
sudo mkdir -p /opt/code-server-isolated/extensions
```

Optional (if you want ownership to your current Linux user):

```bash
sudo chown -R "$(id -u):$(id -g)" /opt/code-server-isolated
```

## 3) Define Repository Mount Path
Choose which VPS repos should be editable remotely. Example:
- host path: /root
- container path: /home/coder/workspaces

If your repos are elsewhere, replace the host path accordingly.

## 4) Start Isolated code-server Container

```bash
docker run -d \
  --name code-server-isolated \
  --restart unless-stopped \
  -p 127.0.0.1:18080:8080 \
  -v /opt/code-server-isolated/data:/home/coder/.local/share/code-server \
  -v /opt/code-server-isolated/config:/home/coder/.config/code-server \
  -v /opt/code-server-isolated/extensions:/home/coder/.local/share/code-server/extensions \
  -v /root:/home/coder/workspaces \
  -u "$(id -u):$(id -g)" \
  -e "DOCKER_USER=$USER" \
  -e "PASSWORD=CHANGE_ME_STRONG_PASSWORD" \
  codercom/code-server:latest
```

Notes:
- The image name matches the repo standard procedure.
- Binding to 127.0.0.1 avoids exposing this service publicly until reverse proxy is ready.
- The /root mount is only an example. Mount only the directories you need.

## 5) Validate Health and Access

```bash
docker ps --filter name=code-server-isolated
docker logs --tail 100 code-server-isolated
curl -I http://127.0.0.1:18080
```

Then open through SSH tunnel or reverse proxy and sign in with PASSWORD.

## 6) Work with Local VPS Repositories
Inside code-server file explorer, open:
- /home/coder/workspaces

You will be able to edit the mounted host repositories remotely.

## 7) Optional Reverse Proxy Publication
Only after validation:
- Add a dedicated host/subdomain or isolated route.
- Keep existing VS Code/VS Code Server routes unchanged.
- Reload proxy and test both services.

## 8) Routine Operations

```bash
docker stop code-server-isolated
docker start code-server-isolated
docker restart code-server-isolated
docker logs -f code-server-isolated
```

## 9) Rollback (No Impact on Existing Services)

```bash
docker stop code-server-isolated
docker rm code-server-isolated
```

Optional full cleanup:

```bash
sudo rm -rf /opt/code-server-isolated
```

## 10) Security and Permissions Notes
- Use a strong password or hashed password strategy.
- Mount only required repo paths (principle of least access).
- If mounted files are root-owned, run container with matching UID/GID or adjust ownership/ACLs.
- Keep this instance on loopback until proxy/TLS is configured.
