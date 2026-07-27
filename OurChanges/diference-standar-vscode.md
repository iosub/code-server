# code-server vs microsoft/vscode: Practical Decision Guide

## Important Clarification
`microsoft/vscode` also supports remote and web scenarios (for example, through VS Code Server / Remote workflows). So this is not "upstream cannot do remote".

The real difference is focus:
- `microsoft/vscode`: upstream editor platform, broad desktop-first ecosystem with remote capabilities.
- `code-server` (this repo): opinionated path to run VS Code in the browser as a self-hosted service.

## What is materially better in code-server

## 1) Browser-first self-hosting
- You can work from any device with only a browser.
- No local desktop client is required for daily usage.

## 2) Centralized environment
- Extensions, toolchains, and repositories stay on the VPS.
- Reduces workstation drift and local machine differences.

## 3) Server-side compute model
- Builds/tests run on the server by default.
- Better for low-power clients and stable remote workflows.

## 4) Service-style operation
- Clean operation via Docker/systemd with restart policies.
- Straightforward 24/7 hosting model in VPS environments.

## 5) Predictable web publishing
- Works well behind reverse proxies (Nginx/Caddy), TLS, and route isolation.
- Easy to run isolated instances on separate ports/domains.

## 6) Packaging options for ops
- Script, package, Docker, Helm, standalone options.
- Useful when you want a controlled deployment shape on VPS.

## 7) Server/web-oriented patch set
- This repo carries patches tuned for browser/server operation.
- Better fit when your main product is "IDE as a web service".

## 8) Low friction for personal remote workspace
- Good for persistent personal VPS workspaces.
- Strong fit when repos and tooling already live on the server.

---

## When to choose each option

## Choose code-server when
- Your primary access model is browser-to-VPS.
- You want an isolated, service-like deployment (Docker/systemd) quickly.
- You value operational simplicity over maximum upstream parity.

## Choose microsoft/vscode remote workflows when
- You prefer the official desktop VS Code client as your daily driver.
- You need upstream-first behavior and ecosystem alignment.
- You are already happy with VS Code Remote and do not need browser-first access.

## Direct answer to "if not substantially better, why not use upstream?"
If your current VS Code Remote setup already solves your needs cleanly, using upstream is a valid and often preferable choice.

Use code-server when you specifically want the browser-first, self-hosted service model; otherwise, upstream + Remote is often the simpler strategic default.
