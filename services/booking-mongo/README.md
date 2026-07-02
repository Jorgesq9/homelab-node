# booking-mongo

Base de datos (MongoDB 7) para `booking-api`.

## Rol en la arquitectura

Almacena los datos de `booking-api`. Corre sobre la misma red Docker compartida `homelab` que el resto de servicios de aplicación (`portfolio`, `sales-api`, `caddy`) — **no está en una red aislada**. Técnicamente cualquier contenedor conectado a esa red podría alcanzar su puerto interno; en la práctica, solo `booking-api` se conecta a él porque es el único servicio con su URI de conexión configurada.

Sin puertos publicados al host: solo alcanzable desde dentro de la red `homelab`, nunca desde fuera del host.

## Puerto

- **27017/tcp** interno (no publicado al host).

## Nota de despliegue

En producción, este servicio se declara **junto con `booking-api`** en un único `docker-compose.yml` (mismo ciclo de vida, mismo `docker compose up`). Aquí se documenta como carpeta independiente para mantener la convención de "una carpeta por servicio" del repositorio, pero el compose que realmente se usa para desplegar es el de `services/booking-api/`.

## Decisiones no obvias

- **Imagen `mongo:7`** (tag de major version, no `latest`): reproducible, no cambia de versión sin que se actualice explícitamente el compose.
- **Volumen nombrado** (`booking-mongo-data`): persiste los datos aunque se recree el contenedor (`docker compose down` sin `-v`).
