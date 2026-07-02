# Arquitectura

## Visión general

El tráfico público llega a través de un **Cloudflare Tunnel** (solo salida, sin puertos de entrada abiertos en el host) hasta un único reverse proxy **Caddy**, que enruta por dominio a cada contenedor de aplicación sobre una red Docker privada (`homelab`). Ningún contenedor de aplicación publica puertos al host: solo Caddy es accesible desde fuera, y solo Caddy conoce el puerto interno de cada servicio.

```
                        Internet
                           │
                           ▼
              ┌─────────────────────────┐
              │   Cloudflare Tunnel      │  solo salida, cero puertos de entrada
              │   (gestionado)           │
              └────────────┬─────────────┘
                           │
                           ▼
              ┌─────────────────────────┐
              │   Caddy (reverse proxy)  │  único punto de entrada, :80
              └────────────┬─────────────┘
                           │  red Docker privada "homelab"
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
  ┌───────────┐     ┌────────────┐     ┌─────────────┐
  │ portfolio │     │ booking-api│────▶│booking-mongo│
  │ (Next.js) │     │ (Express)  │     │  (MongoDB)  │
  └───────────┘     └────────────┘     └─────────────┘
                           ▼
                    ┌────────────┐
                    │  sales-api │
                    │ (Express)  │
                    └────────────┘

  uptime-kuma (monitorización) — aparte, solo LAN en :3001
```

## Enrutado por dominio

Caddy enruta según el `Host` de la petición hacia el contenedor correspondiente por su nombre en la red Docker (resolución DNS interna de Docker, no IPs fijas):

| Dominio | Servicio destino | Puerto interno |
|---|---|---|
| jorgeesquivafullstack.es | `portfolio` | 3000 |
| api-reservas.jorgeesquivafullstack.es, reservas.jorgeesquivafullstack.es | `booking-api` | 5000 |
| api-sales.jorgeesquivafullstack.es | `sales-api` | 3000 |

`auto_https` está desactivado en Caddy porque el TLS se termina en el edge del proveedor del túnel — Caddy solo sirve HTTP plano puertas adentro.

## Red

- Una única red Docker de tipo bridge, `homelab`, compartida por **todos** los servicios de aplicación (`caddy`, `portfolio`, `booking-api`, `booking-mongo`, `sales-api`) — `uptime-kuma` es la única excepción, corre aparte.
- Los contenedores se resuelven entre sí por nombre de servicio (DNS interno de Docker), no por IP.
- **No hay segmentación entre servicios dentro de `homelab`**: cualquier contenedor conectado a esa red puede alcanzar el puerto interno de cualquier otro (por ejemplo, `portfolio` podría alcanzar `booking-mongo` en el puerto 27017 si quisiera). El aislamiento real es solo frente al exterior — ningún contenedor de aplicación publica puertos al host salvo Caddy (80/tcp) y `uptime-kuma` (3001/tcp, solo LAN).
- El firewall del host (UFW) deniega toda entrada por defecto; solo abre explícitamente SSH (22/tcp) a cualquier origen, y el resto de puertos necesarios acotados a la red local.

## Por qué esta forma

- **Cero puertos de entrada para tráfico web**: al no publicar 80/443 directamente al host, el firewall nunca necesita abrir esos puertos a internet — todo pasa por el túnel gestionado.
- **Un único punto de entrada**: si Caddy cae, cae todo el tráfico externo de golpe (visible, fácil de diagnosticar) en vez de fallos parciales silenciosos por servicio.
- **Compose independiente por servicio**: cada servicio se despliega, actualiza y versiona sin afectar a los demás.
- **Red compartida sin segmentar**: es una simplificación consciente para un homelab de este tamaño, no una garantía de aislamiento entre servicios — ver `services/booking-mongo/README.md` para un ejemplo concreto de esa limitación.
