# Infraestructura: Traefik

# Traefik — Reverse Proxy con Let's Encrypt

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

### 2) Clonar y configurar

```bash
git clone https://git.ictiberia.com/groales/traefik
cd traefik

# Crear carpeta para ACME
mkdir -p letsencrypt
touch ./letsencrypt/acme.json
chmod 600 ./letsencrypt/acme.json
```

### 3) Editar configuración

**IMPORTANTE:** Antes de desplegar, edita los siguientes archivos con tus datos reales:

**traefik.yml:**
```yaml
certificatesResolvers:
  letsencrypt:
    acme:
      email: tu-email@tudominio.com  # ← EDITA AQUÍ
```

**docker-compose.yaml:**
```yaml
labels:
  - "traefik.http.routers.traefik.rule=Host(`traefik.tudominio.com`)"  # ← EDITA AQUÍ
```

### 4) Desplegar

```bash
docker network create proxy  # Si no existe
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
