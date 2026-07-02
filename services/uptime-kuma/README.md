# uptime-kuma

Monitorización de disponibilidad (Uptime Kuma).

## Rol en la arquitectura

Servicio independiente, no conectado a la red compartida `homelab` — publica su propio puerto directamente al host, accesible solo desde la red local.

## Puerto

- **3001/tcp** publicado al host, permitido en el firewall solo para la red local (LAN), no expuesto a internet.

## Decisiones no obvias

- **Imagen `louislam/uptime-kuma:1`** (tag de major version, no `latest`): reproducible — no cambia de versión sin que se actualice explícitamente el compose y quede en el historial de git.
- **Volumen nombrado** (`uptime-kuma-data`): persiste la configuración y el histórico aunque se recree el contenedor.
- **Solo accesible desde la LAN**, no desde internet: es un dashboard de monitorización interno. Si en el futuro hace falta acceso remoto, la vía correcta es una VPN o un reverse proxy con autenticación, no abrir el puerto directamente a internet.

## Levantar el servicio

```bash
cd services/uptime-kuma
docker compose up -d
```
