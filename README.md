




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
```

## Despliegue

### 1) Clonar y configurar

```bash

# Crear carpeta para ACME
mkdir -p letsencrypt
touch ./letsencrypt/acme.json
chmod 600 ./letsencrypt/acme.json
```

### 2) Editar configuración

**IMPORTANTE:** Antes de desplegar, edita los siguientes archivos con tus datos reales:

```yaml
certificatesResolvers:
  letsencrypt:
    acme:
      email: tu-email@tudominio.com  # ← EDITA AQUÍ
```

**compose.yaml:**
```yaml
labels:
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


**Usuario por defecto:** `admin` (cambia el hash según tu contraseña)



```yaml
services:
    networks:
      - proxy
    labels:

networks:
  proxy:
    external: true
```

## Buenas Prácticas

- Protege el dashboard con dominio + auth o restringe por IP
- Usa Let's Encrypt staging para pruebas intensivas:

```yaml
certificatesResolvers:
  letsencrypt:
    acme:
      caServer: https://acme-staging-v02.api.letsencrypt.org/directory
```

## Logs


```bash
# Ver logs en tiempo real

# Filtrar errores

# Ver logs de acceso
```


## Troubleshooting

- Certificado no se emite:
  - Verifica DNS y apertura del puerto 80
- 404 en servicio:
  - Verifica que el servicio está en la red `proxy`
- Dashboard no carga:
  - Asegura que el dominio apunta al servidor

## Documentación adicional


- Configuración avanzada (middlewares, serversTransport, mTLS)
- Ejemplos por servicio (Jellyfin, Nextcloud, Vaultwarden)
- Seguridad (headers, rate limiting, IP whitelist)

---

**Última actualización**: Noviembre 2025
