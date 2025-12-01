# Configuración Dinámica de Traefik

Este directorio contiene configuración que Traefik recarga automáticamente sin reiniciar el contenedor.

## Archivos

- **middlewares.yml**: Middlewares reutilizables (headers seguridad, rate limit, auth, etc.)
- **routers.yml**: Routers HTTP/HTTPS (ejemplos comentados)
- **services.yml**: Servicios backend (ejemplos comentados)

**Nota:** Los archivos NO usan la raíz `http:` porque Traefik los carga automáticamente bajo `http.middlewares`, `http.routers`, y `http.services` respectivamente.

## Autenticación Básica

El middleware `auth-basic` está **habilitado por defecto** para proteger el dashboard de Traefik.

### Configurar tu contraseña

1. Genera hash bcrypt:
```bash
docker run --rm httpd:alpine htpasswd -nbB admin tu_password
```

2. Copia el hash completo (después de `admin:`)

3. Edita `middlewares.yml` línea 35:
```yaml
auth-basic:
  basicAuth:
    users:
      - "admin:$2y$05$HASH_GENERADO_AQUI"
```

4. Guarda → recarga automática en ~10 segundos

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

- **Routers y Servicios:** Ver ejemplos comentados en `routers.yml` para configurar rutas sin labels Docker
- **Middlewares avanzados:** Consulta la wiki: https://git.ictiberia.com/groales/traefik/wiki/Middlewares-Seguridad
