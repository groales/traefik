# Configuración Dinámica de Traefik

Este directorio contiene configuración que Traefik recarga automáticamente sin reiniciar el contenedor.

## Archivos

- **middlewares.yml**: Middlewares reutilizables (headers seguridad, rate limit, auth, etc.)

## Uso

### Aplicar middleware a un servicio

En el `docker-compose.yaml` de tu servicio:

```yaml
services:
  mi-servicio:
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.mi-servicio.rule=Host(`app.tudominio.com`)"
      - "traefik.http.routers.mi-servicio.entrypoints=websecure"
      - "traefik.http.routers.mi-servicio.tls.certresolver=letsencrypt"
      - "traefik.http.routers.mi-servicio.middlewares=security-headers@file,rate-limit@file"
```

**Nota:** El sufijo `@file` indica que el middleware viene de configuración dinámica.

### Cadena de middlewares

Puedes combinar varios:
```yaml
- "traefik.http.routers.app.middlewares=security-headers@file,rate-limit@file,ip-whitelist@file"
```

## Recarga automática

Traefik detecta cambios en este directorio y recarga sin reiniciar. Espera ~10 segundos tras editar.

## Ejemplos adicionales

Consulta la wiki: https://git.ictiberia.com/groales/traefik/wiki/Middlewares-Seguridad
