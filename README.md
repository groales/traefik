# Infraestructura: Traefik

# Traefik — Reverse Proxy con Let's Encrypt

Este repositorio despliega **Traefik** como proxy inverso con HTTPS automático (Let's Encrypt), listo para servir como puerta de entrada a tus servicios Docker mediante la red compartida `proxy`.

## ¿Qué es Traefik?

Traefik es un reverse proxy moderno y ligero que detecta servicios Docker automáticamente y los expone vía HTTP/HTTPS con reglas declarativas (labels) y certificados TLS automáticos con Let's Encrypt.

## Características

- 🔒 HTTPS automático con Let's Encrypt (HTTP-01)
- 🧠 Descubrimiento automático de servicios Docker
- 🧿 Redirección HTTP→HTTPS
- 🧰 Dashboard web (seguro por dominio)
- 🧩 Middlewares: auth básica, headers de seguridad, rate limit, etc.
- 📊 Logs de acceso y errores (stdout/stderr)

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

### 1) Clonar y configurar

```bash
git clone https://git.ictiberia.com/groales/traefik
cd traefik

# Crear carpeta para ACME
mkdir -p letsencrypt
touch ./letsencrypt/acme.json
chmod 600 ./letsencrypt/acme.json
```

### 2) Editar configuración

**IMPORTANTE:** Antes de desplegar, edita los siguientes archivos con tus datos reales:

**traefik.yml:**
```yaml
certificatesResolvers:
  letsencrypt:
    acme:
      email: tu-email@tudominio.com  # ← EDITA AQUÍ
```

**docker-compose.yml:**
```yaml
labels:
  - "traefik.http.routers.traefik.rule=Host(`traefik.tudominio.com`)"  # ← EDITA AQUÍ
```

### 3) Desplegar

```bash
docker compose up -d
```

## Dashboard Seguro

Este stack expone el dashboard por dominio usando TLS y el servicio interno `api@internal`.

**Autenticación básica habilitada:** El dashboard está protegido mediante el middleware `auth-basic@file` definido en `dynamic/config.yml`.

### Configurar contraseña

1. Genera el hash bcrypt:
```bash
docker run --rm httpd:alpine htpasswd -nbB admin tu_password_segura
```

2. Edita `dynamic/config.yml` (sección middlewares > auth-basic) y reemplaza el hash de ejemplo:
```yaml
auth-basic:
  basicAuth:
    users:
      - "admin:$2y$05$tu_hash_generado_aqui"
```

3. Guarda el archivo. Traefik recargará automáticamente en ~10 segundos (no requiere reinicio).

**Usuario por defecto:** `admin` (cambia el hash según tu contraseña)

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

## Logs

Traefik envía logs a stdout/stderr (sin persistencia en disco):

```bash
# Ver logs en tiempo real
docker logs -f traefik

# Filtrar errores
docker logs traefik | Select-String -Pattern error

# Ver logs de acceso
docker logs traefik | Select-String -Pattern "GET|POST"
```

**Nivel de log**: INFO (configurable en `traefik.yml` → `log.level`)

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
