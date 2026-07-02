# portfolio

Portfolio profesional (Next.js), servido en producción desde este mismo homelab.

## Rol en la arquitectura

Contenedor de aplicación sin puertos publicados al host — solo accesible a través de `caddy`, que lo enruta por dominio.

## Puerto

- **3000/tcp** interno (no publicado al host).

## Decisiones no obvias

- El build usa como contexto el repositorio del portfolio, clonado aparte en el host (no vive dentro de este repositorio).
- Monta como solo lectura un fichero JSON con métricas del host (uptime, CPU, RAM, disco), generado periódicamente por un script fuera de Docker — así el contenedor de la app puede leer telemetría del host sin tener acceso directo a él.
- `PORTFOLIO_REPO_PATH` y `NODE_INFO_JSON_PATH` se definen en un `.env` local, no versionado, con las rutas reales de tu propio host.

## Levantar el servicio

```bash
cd services/portfolio
docker compose up -d
```
