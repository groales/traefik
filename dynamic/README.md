

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
```

## Autenticación Básica


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

En el `compose.yaml` de tu servicio:

```yaml
services:
  mi-servicio:
    labels:
```

**Nota:** El sufijo `@file` indica que el middleware viene de configuración dinámica.

### Cadena de middlewares

Puedes combinar varios:
```yaml
```

## Recarga automática


## Ejemplos adicionales

- **Routers y Servicios:** Ver ejemplos comentados en `config.yml` (secciones routers y services) para configurar rutas sin labels Docker
