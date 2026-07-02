# Hardening del host

Documentación de las decisiones de endurecimiento del host que corre este homelab (Ubuntu Server 24.04 LTS).

## Estado inicial detectado

Antes de tocar nada se auditó el host, aunque se esperaba partir de una base "limpia". Se encontró configuración de sistema previa que había sobrevivido a un borrado anterior de contenedores y servicios:

- Repositorios de paquetes ya configurados (Docker, entre otros).
- Docker Engine y Docker Compose ya instalados y funcionando — se aprovechó directamente en vez de reinstalar.
- Reglas de firewall persistidas de una configuración anterior, sin ningún servicio real detrás. Se eliminaron para partir de un firewall limpio.

**Lección**: "borrar todo" en la práctica limpia contenedores y datos de aplicación, pero no necesariamente configuración de sistema (fuentes de apt, reglas de firewall, servicios habilitados). Una reconstrucción real desde cero requiere auditar también eso, no asumir que está vacío.

## Acceso sudo para automatización

Para poder ejecutar el proceso de hardening de forma encadenada sin exponer contraseñas en cada paso, se configuró sudo **sin contraseña pero acotado** a un conjunto concreto de comandos (gestión de paquetes, firewall, reinicio/estado de servicios concretos como fail2ban, SSH y Docker, y escritura solo sobre rutas de configuración específicas).

**Por qué acotado y no acceso total**: reduce el radio de impacto si esta automatización se viera comprometida — no puede borrar archivos arbitrarios ni escribir en cualquier ruta como root. El coste es mantenimiento manual (cada nuevo comando necesario amplía la lista explícitamente), aceptado conscientemente a cambio de la reducción de riesgo.

Esta configuración de sudo no se versiona en ningún repositorio: vive únicamente en el host, como credencial de automatización local.

## Eliminación de snapd

No había ningún snap instalado, así que snapd solo consumía recursos sin aportar nada — este stack se gestiona íntegramente vía apt y Docker. Se purgó y se limpiaron las dependencias huérfanas.

## Firewall (UFW)

Instalado y configurado con política **deny por defecto en entrada, allow en salida**, permitiendo explícitamente solo SSH antes de activarlo.

**Por qué no se activó de inmediato**: activar el firewall antes de confirmar que el acceso SSH funciona es el error clásico que te deja fuera de tu propia máquina. La regla de SSH se preparó y verificó primero; el firewall se activó al final, después de confirmar el acceso por clave.

## fail2ban — protección SSH

```ini
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 5

[sshd]
enabled = true
port    = ssh
backend = systemd
```

Configuración versionada como fuente de verdad, desplegada al fichero de overrides del paquete (nunca se edita la configuración por defecto directamente, porque el propio paquete la sobreescribe en cada actualización).

**Por qué estos valores**: 5 intentos fallidos en 10 minutos provoca 1 hora de baneo — suficientemente estricto para frenar fuerza bruta automatizada, sin ser tan agresivo que un typo legítimo banee de forma permanente.

## SSH — solo claves, sin root login

Se sustituyó el acceso SSH existente por una clave ED25519 nueva, generada fuera del propio host y protegida con passphrase (para que el robo del dispositivo cliente no implique acceso directo).

**Hallazgo crítico de configuración**: la configuración de SSH incluye ficheros de overrides en orden alfabético, y las imágenes cloud de Ubuntu traen un drop-in preexistente que fuerza la autenticación por contraseña. En `sshd`, ante una directiva repetida **gana el primer valor encontrado**, no el último — así que editar el fichero principal de configuración no habría tenido ningún efecto, porque ese drop-in se carga antes.

**Solución**: un drop-in propio nombrado para que ordene *antes* que el de cloud-init (prefijo numérico menor), con:

```
PasswordAuthentication no
PermitRootLogin no
```

Proceso seguido, en orden:
1. Backup de la configuración original, como red de seguridad.
2. Creación del drop-in propio.
3. Validación de sintaxis (`sshd -t`) **antes** de reiniciar el servicio — evita quedarte fuera por un error de sintaxis.
4. Reinicio del servicio SSH.
5. Verificación desde una sesión **nueva**, sin cerrar la existente: acceso por clave aceptado, acceso forzando solo contraseña rechazado.

**Por qué verificar en una sesión nueva sin cerrar la actual**: si la configuración quedara mal, la sesión ya abierta sigue viva (los cambios no afectan a conexiones ya establecidas) y permite revertir desde ahí sin quedar bloqueado fuera del host.

## Activación final del firewall

Con el acceso por clave confirmado y el acceso por contraseña confirmado como rechazado, se activa el firewall. Se pospuso hasta este punto exacto a propósito: activarlo antes de confirmar el acceso por clave es el error que puede dejarte fuera de tu propia máquina si algo en SSH no estaba bien.
