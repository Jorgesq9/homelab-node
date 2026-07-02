# homelab-node

**🌐 Idioma / Language:** [English](#english) · [Español](#español)

---

## Español

**Un homelab autoalojado, construido y operado desde cero como Infraestructura como Código.**

Este repositorio documenta la arquitectura, el hardening y las decisiones operativas de un pequeño homelab en producción que corre sobre un único mini PC. Cada servicio está contenedorizado, detrás de un reverse proxy, y expuesto a internet **sin abrir un solo puerto de entrada** en el host.

> 👤 Construido por **Jorge Esquiva** — Administrador de Sistemas.
> Portfolio en vivo (servido desde este mismo homelab): **[jorgeesquivafullstack.es](https://jorgeesquivafullstack.es)**

### Por qué existe

Administro sistemas críticos de producción (Linux y mainframe z/OS) en mi trabajo. Este homelab es donde diseño, despliego y opero mi propia infraestructura de principio a fin — aplicando la misma disciplina que uso profesionalmente, pero donde controlo cada decisión: el SO, el hardening, la red, el despliegue.

Es deliberadamente **honesto**: lo que corre, corre; lo que está planificado, se etiqueta como planificado. Sin luces verdes aspiracionales.

### Arquitectura

El tráfico llega a los servicios a través de un **Cloudflare Tunnel** (solo salida, sin puertos de entrada abiertos en el host) hacia un único reverse proxy **Caddy**, que enruta a cada contenedor por una red Docker privada. Los contenedores de aplicación **no exponen ningún puerto al host** — solo Caddy es accesible, y solo Caddy habla con ellos.

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

**Decisiones de diseño destacables:**

- **Cero puertos de entrada.** El túnel es solo de salida, así que el firewall del host nunca expone 80/443 a internet. La superficie de ataque se mantiene mínima.
- **Único punto de entrada.** Solo Caddy es accesible; cada contenedor de aplicación vive en una red privada sin puertos publicados al host.
- **Un compose por servicio.** Cada servicio se despliega, versiona y documenta de forma independiente — nada de un compose monolítico.

### Hardening del host

El host se reconstruyó desde cero y se endureció antes de desplegar nada:

| Medida | Detalle |
|---|---|
| **SSH** | Autenticación solo por clave (ED25519). Login por contraseña y root deshabilitados. |
| **Firewall** | UFW, deny por defecto en entrada / allow en salida. Solo puertos necesarios, acotados a LAN. |
| **Anti fuerza bruta** | fail2ban con jail SSH (5 intentos / 10 min → 1h de baneo). |
| **Superficie de ataque** | snapd purgado; sin daemons innecesarios. |
| **Mínimo privilegio** | sudo de automatización acotado a comandos concretos, nunca `NOPASSWD:ALL`. |

Una decisión documentada como ejemplo del razonamiento aplicado en todo el proyecto: las imágenes cloud de Ubuntu incluyen un drop-in que fuerza `PasswordAuthentication yes`, y `sshd` aplica el **primer** valor coincidente en su orden de `Include`. Editar la config principal no tiene efecto — la solución es un drop-in que ordene *antes* que el de cloud-init. El hardening no es ejecutar comandos de un tutorial; es entender *por qué* hace falta cada uno.

### Stack

**En producción hoy:**

`Ubuntu Server 24.04 LTS` · `Docker` · `Docker Compose` · `Caddy` · `Cloudflare Tunnel` · `Node.js` · `MongoDB` · `Uptime Kuma`

**Planificado (aún no desplegado — etiquetado con honestidad):**

`Prometheus` · `Grafana` · `Loki` · `Ansible` · `Terraform` · `K3s` · `GitHub Actions self-hosted runner`

### Estructura del repositorio

```
homelab-node/
├── README.md              # este archivo
├── docs/                  # notas de arquitectura, decisiones
└── services/               # una carpeta por servicio (compose + notas)
```

> Esta es la vista **pública y curada** del homelab. El código de las aplicaciones vive en sus propios repositorios, enlazados desde el portfolio.

### Hoja de ruta

- [x] Host reconstruido desde cero, endurecido, versionado como IaC
- [x] Reverse proxy + Cloudflare Tunnel, cero puertos de entrada
- [x] Portfolio + dos APIs desplegados en producción
- [x] Panel de telemetría en vivo (uptime, CPU, RAM, disco, health checks)
- [ ] Stack de observabilidad (Prometheus / Grafana / Loki)
- [ ] Gestión de configuración (Ansible)
- [ ] Pipeline CI/CD con runner self-hosted

---

## English

**A self-hosted homelab, built and operated from scratch as Infrastructure as Code.**

This repository documents the architecture, hardening, and operational decisions behind a small production homelab running on a single mini PC. Every service is containerised, reverse-proxied, and exposed to the internet **without opening a single inbound port** on the host.

> 👤 Built by **Jorge Esquiva** — Systems Administrator.
> Live portfolio (served from this very homelab): **[jorgeesquivafullstack.es](https://jorgeesquivafullstack.es)**

### Why this exists

I administer critical production systems (Linux and z/OS mainframe) in my day job. This homelab is where I design, deploy, and operate my own infrastructure end to end — applying the same discipline I use professionally, but where I own every decision: the OS, the hardening, the networking, the deployment.

It is intentionally **honest**: what runs, runs; what is planned, is labelled as planned. No aspirational green lights.

### Architecture

Traffic reaches the services through a **Cloudflare Tunnel** (outbound-only, no inbound ports open on the host) into a single **Caddy** reverse proxy, which routes to each container over a private Docker network. Application containers expose **no ports to the host** — only Caddy is reachable, and only Caddy talks to them.

```
                        Internet
                           │
                           ▼
              ┌─────────────────────────┐
              │   Cloudflare Tunnel      │  outbound-only, no open inbound ports
              │   (managed)              │
              └────────────┬─────────────┘
                           │
                           ▼
              ┌─────────────────────────┐
              │   Caddy (reverse proxy)  │  single entrypoint, :80
              └────────────┬─────────────┘
                           │  private Docker network "homelab"
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

  uptime-kuma (monitoring) — separate, LAN-only on :3001
```

**Design choices worth noting:**

- **No inbound ports.** The tunnel is outbound-only, so the host's firewall never exposes 80/443 to the internet. Attack surface stays minimal.
- **Single entrypoint.** Only Caddy is reachable; every app container lives on a private network with no host-published ports.
- **One compose per service.** Each service is independently deployable, versioned, and documented — no monolithic compose file.

### Host hardening

The host was rebuilt from zero and hardened before anything was deployed:

| Measure | Detail |
|---|---|
| **SSH** | Key-only authentication (ED25519). Password and root login disabled. |
| **Firewall** | UFW, default deny inbound / allow outbound. Only required ports, LAN-scoped. |
| **Brute-force protection** | fail2ban with an SSH jail (5 attempts / 10 min → 1h ban). |
| **Attack surface** | snapd purged; no unnecessary daemons running. |
| **Least privilege** | Automation sudo scoped to specific commands, never `NOPASSWD:ALL`. |

One decision documented as an example of the reasoning applied throughout: Ubuntu cloud images ship a drop-in that forces `PasswordAuthentication yes`, and `sshd` applies the **first** matching value in its `Include` order. Editing the main config has no effect — the fix is a drop-in that sorts *before* the cloud-init one. Hardening isn't running commands from a tutorial; it's understanding *why* each one is needed.

### Stack

**Running in production today:**

`Ubuntu Server 24.04 LTS` · `Docker` · `Docker Compose` · `Caddy` · `Cloudflare Tunnel` · `Node.js` · `MongoDB` · `Uptime Kuma`

**Planned (not yet deployed — labelled honestly):**

`Prometheus` · `Grafana` · `Loki` · `Ansible` · `Terraform` · `K3s` · `GitHub Actions self-hosted runner`

### Repository layout

```
homelab-node/
├── README.md              # this file
├── docs/                  # architecture notes, decisions
└── services/               # one folder per service (compose + notes)
```

> This is the **curated, public** view of the homelab. Application source code lives in its own repositories, linked from the portfolio.

### Roadmap

- [x] Host rebuilt from zero, hardened, versioned as IaC
- [x] Reverse proxy + Cloudflare Tunnel, zero inbound ports
- [x] Portfolio + two APIs deployed to production
- [x] Live telemetry dashboard (uptime, CPU, RAM, disk, health checks)
- [ ] Observability stack (Prometheus / Grafana / Loki)
- [ ] Configuration management (Ansible)
- [ ] CI/CD pipeline with self-hosted runner

---

*Administrador de Sistemas en transición a DevOps / Platform Engineering. Abierto a roles remotos, a nivel internacional.*
