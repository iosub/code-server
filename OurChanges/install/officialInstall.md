# code-server: oficial minimo + extras opcionales

Este documento separa claramente:
- lo oficial minimo de Coder
- los extras opcionales para tu VPS (Nginx + GitHub OAuth + no romper VS Code Server)

## 1) Instalacion oficial minima (tal cual docs)

Referencia oficial:
- https://coder.com/docs/code-server

Comandos oficiales:

	curl -fsSL https://code-server.dev/install.sh | sh -s -- --dry-run
	curl -fsSL https://code-server.dev/install.sh | sh

Con eso ya tienes code-server instalado.

## 2) Extras opcionales para este VPS (recomendado en tu caso)

Objetivo de estos extras:
- mantener login por GitHub en mycode.mysaasplace.com
- ejecutar code-server nativo como root
- no afectar VS Code Server ya existente

### 2.1 Limpiar contenedor viejo de code-server

	docker rm -f code-server-isolated

Nota: no toca oauth2-proxy.

### 2.2 Config nativa de code-server en localhost

	mkdir -p /root/.config/code-server
	cat > /root/.config/code-server/config.yaml <<'EOF'
	bind-addr: 127.0.0.1:18081
	auth: none
	cert: false
	user-data-dir: /root/.local/share/code-server
	extensions-dir: /root/.local/share/code-server/extensions
	EOF

### 2.3 Servicio systemd

	cat > /etc/systemd/system/code-server-root.service <<'EOF'
	[Unit]
	Description=code-server root native
	After=network.target

	[Service]
	Type=simple
	User=root
	Group=root
	WorkingDirectory=/root
	Environment=HOME=/root
	ExecStart=/usr/bin/code-server --config /root/.config/code-server/config.yaml
	Restart=always
	RestartSec=3

	[Install]
	WantedBy=multi-user.target
	EOF

	systemctl daemon-reload
	systemctl enable --now code-server-root.service

Si el binario no esta en /usr/bin/code-server, obtener ruta con:

	command -v code-server

y ajustar ExecStart.

### 2.4 Verificacion local

	systemctl status code-server-root.service --no-pager
	ss -ltnp | grep 18081
	curl -I http://127.0.0.1:18081

### 2.5 Nginx apuntando al puerto nativo

En el host de mycode, upstream:

	proxy_pass http://127.0.0.1:18081;

Aplicar:

	nginx -t && systemctl reload nginx

### 2.6 Validacion final

1. Abrir https://mycode.mysaasplace.com
2. Login con GitHub
3. Entrar a code-server sin password interno
4. Abrir rutas reales del host como /root/.hermes

## 3) Operacion diaria

	systemctl restart code-server-root.service
	systemctl stop code-server-root.service
	systemctl start code-server-root.service
	journalctl -u code-server-root.service -n 200 --no-pager

## 4) Rollback rapido

1. Restaurar proxy_pass previo en Nginx
2. Recargar Nginx
3. Parar servicio nativo

Comandos:

	systemctl stop code-server-root.service
	nginx -t && systemctl reload nginx

## 5) Notas de seguridad

1. No exponer 18081 fuera de localhost.
2. auth none solo si oauth2-proxy delante esta activo.
3. Si el client secret de GitHub se compartio en texto plano, regenerarlo.
