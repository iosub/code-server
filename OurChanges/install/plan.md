# Isolated code-server Installation Plan (No Impact on Existing VS Code/VS Code Server)

## Goal
Install code-server on this VPS in an isolated way so it does not interfere with the currently running VS Code / VS Code Server setup.

## Scope
- Keep existing services and files untouched.
- Use separate port, process, config path, data path, and extensions path.
- Validate before exposing publicly.
- Keep a fast rollback path.

## Option Summary
- Preferred: Docker-based isolated deployment.
- Alternative: Standalone binary with dedicated user.

## Phase 1: Pre-Install Checks
1. Identify active listeners and current editor-related services.
2. Confirm a free local port for the isolated instance (example: `18080`).
3. Confirm reverse proxy (if any) routes currently in use.
4. Confirm available resources (CPU, RAM, disk) for an additional editor process.

## Phase 2A (Preferred): Docker Isolated Install
1. Create dedicated host directories:
	- `/opt/code-server-isolated/config`
	- `/opt/code-server-isolated/data`
	- `/opt/code-server-isolated/extensions`
2. Start container on loopback only (`127.0.0.1`) using a different port mapping:
	- Host: `127.0.0.1:18080`
	- Container: `8080`
3. Set authentication via environment variable (`PASSWORD` or hashed secret strategy).
4. Configure restart policy (`unless-stopped`).
5. Do not modify existing proxy routes yet.

## Phase 2B (Alternative): Standalone Isolated Install
1. Create a dedicated service user (example: `csisolated`).
2. Download standalone release into user-owned app directory.
3. Run with explicit isolated paths:
	- `--bind-addr 127.0.0.1:18080`
	- `--user-data-dir <isolated-path>`
	- `--extensions-dir <isolated-path>`
	- `--config <isolated-path>`
4. Keep service name distinct from any existing unit.

## Phase 3: Validation (Before Public Exposure)
1. Verify process is running and healthy.
2. Verify login works and editor loads.
3. Confirm no changes in existing VS Code/VS Code Server processes.
4. Confirm existing reverse proxy routes still resolve correctly.
5. Confirm acceptable resource impact.

## Phase 4: Optional Reverse Proxy Publication
1. Add a separate host/subdomain or clearly isolated route.
2. Keep current production route unchanged.
3. Reload proxy and test both old and new endpoints.

## Rollback Plan (1-2 Minutes)
1. Stop isolated service/container.
2. Remove isolated container/unit only.
3. Keep existing VS Code/VS Code Server untouched.
4. Remove isolated directories only if full cleanup is needed.

## Risks and Mitigations
- Port collision -> use explicit free port and validate first.
- Proxy route collision -> use a dedicated host/route.
- Data overlap -> never share data/extensions/config directories.
- Resource contention -> monitor CPU/RAM and cap resources if using Docker.

## Success Criteria
- Existing VS Code/VS Code Server remains fully functional.
- New isolated code-server is reachable and authenticated.
- No shared state between existing and isolated environments.
- Rollback can be executed without downtime for existing services.
