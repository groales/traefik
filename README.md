
# Traefik

Reverse proxy moderno para exponer servicios Docker con HTTPS automático y enrutado por dominio.

Referencia oficial de instalación: https://doc.traefik.io/traefik/

## Características

- HTTPS automático con Let's Encrypt (HTTP-01).
- Descubrimiento automático de servicios Docker.
- Redirección HTTP -> HTTPS.
- Dashboard protegido por dominio y autenticación básica.
- Middlewares de seguridad, rate-limit e IP allow list.

## Requisitos Previos

- Docker Engine instalado.
- Docker Compose instalado.
- Red Docker externa `proxy` creada.
- Dominio apuntando al servidor para certificados y dashboard.
- Puertos 80 y 443 accesibles desde Internet.

## Archivos de este Repositorio

- `compose.yaml` - Servicio Traefik.
- `traefik.yml` - Configuración estática.
- `dynamic/config.yml` - Configuración dinámica y middlewares.
- `README.md` - Esta documentación.

---

## Despliegue con Docker Compose

### 1. Clonar el repositorio

```bash
git clone https://github.com/groales/traefik.git
cd traefik
```

### 2. Preparar almacenamiento ACME

```bash
mkdir -p letsencrypt
touch ./letsencrypt/acme.json
chmod 600 ./letsencrypt/acme.json
```

### 3. Revisar configuración

- En `traefik.yml`, ajusta email ACME:

```yaml
certificatesResolvers:
  letsencrypt:
    acme:
      email: admin@tudominio.com
```

- En `compose.yaml`, ajusta dominio del dashboard:

```yaml
- "traefik.http.routers.traefik.rule=Host(`traefik.tudominio.com`)"
```

- En `dynamic/config.yml`, cambia el hash de `auth-basic`.

### 4. Levantar servicio

```bash
docker network create proxy
docker compose up -d
```

---

## Método Alternativo: Despliegue Manual

Puedes usar Docker CLI replicando volúmenes, puertos y labels del `compose.yaml`.

---

## Acceso Inicial

- Dashboard: `https://traefik.tudominio.com`.
- El dashboard usa `api@internal` y middleware `auth-basic@file`.

## Comandos Útiles

```bash
docker compose logs -f traefik
docker compose restart traefik
docker compose pull
docker compose up -d
docker compose down
```

## Estructura de Volúmenes

```text
Bind mounts:
├── /var/run/docker.sock -> /var/run/docker.sock:ro
├── ./traefik.yml -> /traefik.yml:ro
├── ./dynamic -> /etc/traefik/dynamic:ro
└── ./letsencrypt -> /letsencrypt
```

## Configuración Avanzada

- `dynamic/config.yml` incluye middlewares:
  - `security-headers`
  - `rate-limit`
  - `ip-allowlist`
  - `auth-basic`
- Puedes añadir routers y servicios dinámicos en el mismo archivo.
- Para pruebas ACME intensivas, usa servidor staging.

## Solución de Problemas

- Certificado no se emite:
  - Verifica DNS y puerto 80 abierto.
- 404 en servicio:
  - Verifica labels y red `proxy` del contenedor destino.
- Dashboard no carga:
  - Verifica regla Host y certificado.

## Seguridad

- Mantén dashboard protegido con auth y/o allow-list.
- No expongas `:8080` sin protección adicional.
- Revisa headers de seguridad y rate-limit en producción.

## Backup y Restauración

```bash
# Backup básico
tar -czf traefik-backup-$(date +%Y%m%d).tar.gz ./traefik.yml ./dynamic ./letsencrypt

# Restauración
docker compose down
tar -xzf traefik-backup-YYYYMMDD.tar.gz
docker compose up -d
```

## Actualización

```bash
docker compose pull
docker compose up -d
docker compose logs -f traefik
```

## Recursos

- Documentación oficial Traefik: https://doc.traefik.io/traefik/
- Referencia Docker provider: https://doc.traefik.io/traefik/providers/docker/
- Referencia ACME: https://doc.traefik.io/traefik/https/acme/

## Licencia

Este repositorio de configuración es de uso libre. Revisa la licencia del proyecto original en su repositorio oficial.
