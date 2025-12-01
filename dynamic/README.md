# Configuración Dinámica de Traefik

Este directorio contiene configuración que Traefik recarga automáticamente sin reiniciar el contenedor.

## Archivos

- **config.yml**: Configuración dinámica completa (serversTransports, middlewares, routers, servicios)

**Importante:** Con `directory:` en el proveedor file, se debe usar un único archivo consolidado con la estructura `http:` como raíz.

## ServersTransports

El archivo incluye el transport `insecure` para servicios con certificados autofirmados (como Portainer):

```yaml
http:
  serversTransports:
    insecure:
      insecureSkipVerify: true
```

**Uso en labels**:
```yaml
- "traefik.http.services.mi-servicio.loadbalancer.serversTransport=insecure@file"
```

## Autenticación Básica

El middleware `auth-basic` está **habilitado por defecto** para proteger el dashboard de Traefik.

### Configurar tu contraseña

1. Genera hash bcrypt:
```bash
docker run --rm httpd:alpine htpasswd -nbB admin tu_password
```

2. Copia el hash completo (después de `admin:`)

3. Edita `config.yml` (sección http > middlewares > auth-basic):
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

- **Routers y Servicios:** Ver ejemplos comentados en `config.yml` (secciones routers y services) para configurar rutas sin labels Docker
- **Middlewares avanzados:** Consulta la wiki: https://git.ictiberia.com/groales/traefik/wiki/Middlewares-Seguridad
