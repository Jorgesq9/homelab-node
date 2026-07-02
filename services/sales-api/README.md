# sales-api

API de ventas (Node.js/Express + Prisma). El código fuente vive en su propio repositorio, no aquí — este directorio solo documenta cómo se despliega en producción.

## Rol en la arquitectura

Contenedor de aplicación sobre la red `homelab`, sin puertos publicados al host — solo accesible a través de `caddy`.

## Puerto

- **3000/tcp** interno (no publicado al host).

## Decisiones no obvias

- Base de datos SQLite embebida en un volumen nombrado (`sales-api-data`), no un servicio de base de datos aparte — suficiente para el volumen de datos actual, sin la complejidad operativa de gestionar otro contenedor de base de datos.
- Autenticación por cabecera de API key en las rutas de escritura — el detalle vive en el propio repositorio de la aplicación.

## Despliegue

El build (`build: .`) espera un `Dockerfile` en este mismo directorio y el código fuente de la API clonado junto a él. El código de la aplicación se gestiona en su propio repositorio.

```bash
cd services/sales-api
docker compose up -d
```
