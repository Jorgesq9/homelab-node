# caddy

Reverse proxy — único punto de entrada del tráfico HTTP hacia los servicios de aplicación.

## Rol en la arquitectura

Recibe el tráfico que llega a través del túnel gestionado y lo enruta por dominio (`Host`) al contenedor correspondiente, resolviendo por nombre de servicio dentro de la red Docker `homelab`. Es el único contenedor con un puerto publicado al host (80/tcp) — el resto de servicios de aplicación solo son accesibles a través de él.

## Puerto

- **80/tcp** publicado al host (único punto de entrada).

## Decisiones no obvias

- `auto_https off`: TLS se termina en el borde del proveedor del túnel, no en este contenedor — servir HTTPS aquí sería redundante y añadiría complejidad de certificados sin beneficio real.
- `admin off`: desactiva la API de administración de Caddy, que por defecto escucha en el propio contenedor — no se usa y reduce superficie de ataque.
- El `Caddyfile` se monta como solo lectura (`:ro`) — el contenedor nunca necesita escribir su propia configuración.

## Levantar el servicio

```bash
cd services/caddy
docker compose up -d
```

Requiere que la red externa `homelab` exista de antemano (`docker network create homelab`) y que los contenedores a los que enruta ya estén corriendo en esa red.
