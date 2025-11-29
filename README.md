# Infraestructura: Traefik

# Traefik v2 — Reverse Proxy con Let's Encrypt

Este repositorio despliega **Traefik** como proxy inverso con HTTPS automático (Let's Encrypt), listo para servir como puerta de entrada a tus servicios Docker mediante la red compartida `proxy`.

## ¿Qué es Traefik?

Traefik es un reverse proxy moderno y ligero que detecta servicios Docker automáticamente y los expone vía HTTP/HTTPS con reglas declarativas (labels) y certificados TLS automáticos con Let's Encrypt.

## Características

- 🔒 HTTPS automático con Let's Encrypt (HTTP-01)
- 🧠 Descubrimiento automático de servicios Docker
- 🧷 Redirección HTTP→HTTPS
- 🧰 Dashboard web (seguro por dominio)
- 🧩 Middlewares: auth básica, headers de seguridad, rate limit, etc.

## Requisitos

- Docker + Docker Compose
- Red Docker externa `proxy`
- Dominio apuntando al servidor (para dashboard y certificados)
- Puertos 80 y 443 accesibles desde Internet

## Arquitectura

```
Internet → Traefik (80/443) → Servicios (en red proxy)
```

## Despliegue

### 1) Crear red `proxy`

```bash
docker network create proxy
```

### 2) Clonar y desplegar

```bash
git clone https://git.ictiberia.com/groales/traefik
cd traefik

# Crear carpeta y archivo para ACME
mkdir -p letsencrypt
# Windows PowerShell
echo $null > .\letsencrypt\acme.json
# Linux/macOS
# touch ./letsencrypt/acme.json

# Permisos (Linux)
# chmod 600 ./letsencrypt/acme.json

# Desplegar
docker compose up -d
```

### 3) Configurar dominio y email

Edita `traefik.yml` y ajusta:
- `certificatesResolvers.letsencrypt.acme.email`
- (Opcional) Cambia el dominio del dashboard en labels del contenedor (`traefik.tudominio.com`)

Reinicia:
```bash
docker compose up -d
```

## Dashboard Seguro

Este stack expone el dashboard por dominio usando TLS y el servicio interno `api@internal`.

Para proteger con autenticación básica (opcional):

```yaml
labels:
  - "traefik.http.routers.traefik.middlewares=traefik-auth"
  - "traefik.http.middlewares.traefik-auth.basicauth.users=admin:$$apr1$$<hash>"
```

Generar hash (htpasswd):
```bash
# Linux/macOS
htpasswd -nb admin 'TuPassword'

# PowerShell con OpenSSL (alternativa)
# openssl passwd -apr1 TuPassword
```

## Exponer Servicios Detrás de Traefik

Conecta tus servicios a la red `proxy` y añade labels. Ejemplo: Portainer

```yaml
services:
  portainer:
    image: portainer/portainer-ce:lts
    networks:
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.portainer.rule=Host(`portainer.tudominio.com`)"
      - "traefik.http.routers.portainer.entrypoints=websecure"
      - "traefik.http.routers.portainer.tls=true"
      - "traefik.http.routers.portainer.tls.certresolver=letsencrypt"
      - "traefik.http.services.portainer.loadbalancer.server.port=9443"
      - "traefik.http.services.portainer.loadbalancer.server.scheme=https"

networks:
  proxy:
    external: true
```

## Buenas Prácticas

- Usa `exposedByDefault=false` (ya configurado) y habilita servicios con `traefik.enable=true`
- No expongas puertos directos (`ports:`) si accedes vía Traefik
- Protege el dashboard con dominio + auth o restringe por IP
- Usa Let's Encrypt staging para pruebas intensivas:

```yaml
certificatesResolvers:
  letsencrypt:
    acme:
      caServer: https://acme-staging-v02.api.letsencrypt.org/directory
```

## Troubleshooting

- Certificado no se emite:
  - Verifica DNS y apertura del puerto 80
  - Revisa logs: `docker logs traefik | Select-String -Pattern cert,acme,error`
- 404 en servicio:
  - Revisa labels y que el contenedor tenga `traefik.enable=true`
  - Verifica que el servicio está en la red `proxy`
- Dashboard no carga:
  - Asegura que el dominio apunta al servidor
  - Verifica las labels del router `traefik`

## Documentación adicional

Consulta la **Wiki**: https://git.ictiberia.com/groales/traefik/wiki

- Configuración avanzada (middlewares, serversTransport, mTLS)
- Ejemplos por servicio (Jellyfin, Nextcloud, Vaultwarden)
- Seguridad (headers, rate limiting, IP whitelist)

---

**Última actualización**: Noviembre 2025
