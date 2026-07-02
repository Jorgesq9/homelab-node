# booking-api

API de reservas (Node.js/Express). El código fuente vive en su propio repositorio, no aquí — este directorio solo documenta cómo se despliega en producción.

## Rol en la arquitectura

Contenedor de aplicación conectado a `booking-mongo` como base de datos, ambos sobre la red compartida `homelab` — la misma red que usan `portfolio`, `sales-api` y `caddy`. Sin puertos publicados al host — solo accesible a través de `caddy`.

## Puerto

- **5000/tcp** interno (no publicado al host).

## Decisiones no obvias

- `depends_on: booking-mongo` garantiza el orden de arranque, pero no espera a que Mongo esté realmente listo para aceptar conexiones — la app maneja sus propios reintentos de conexión.
- `booking-mongo` no está en una red aislada: comparte la red `homelab` con el resto de servicios. En la práctica solo `booking-api` se conecta a ella, pero no hay una restricción de red que lo imponga — ver `services/booking-mongo/README.md`.

## Despliegue

El build (`build: .`) espera un `Dockerfile` en este mismo directorio y el código fuente de la API clonado junto a él. El código de la aplicación se gestiona en su propio repositorio.

```bash
cd services/booking-api
docker compose up -d
```
