# Traefik con Docker Compose

Traefik es un proxy inverso y balanceador que detecta contenedores Docker y enruta solicitudes HTTP/HTTPS según las etiquetas de cada servicio. Este proyecto usa la imagen oficial `traefik:latest` y una configuración sencilla para desarrollo local.

## Enlaces oficiales

- [Documentación de Traefik](https://doc.traefik.io/traefik/)
- [Inicio rápido oficial con Docker Compose](https://doc.traefik.io/traefik/getting-started/docker/)
- [Proveedor Docker y etiquetas](https://doc.traefik.io/traefik/reference/install-configuration/providers/docker/)
- [API y dashboard: configuración y seguridad](https://doc.traefik.io/traefik/reference/install-configuration/api-dashboard/)
- [Imagen oficial de Docker Hub](https://hub.docker.com/_/traefik)

## Requisitos y puertos

- Docker Engine o Docker Desktop con Docker Compose v2.
- Acceso al socket Docker del host (`/var/run/docker.sock`) para descubrir servicios.
- Puerto TCP 80 para HTTP, 443 para HTTPS y 8080 para el dashboard. De forma predeterminada se enlazan solo a `127.0.0.1`; cambia los puertos en `.env` si ya están ocupados.

## Decisiones de configuración

- `traefik:latest` se conserva según lo solicitado; para despliegues reproducibles, fija una etiqueta de versión aprobada en `TRAEFIK_IMAGE`.
- El proveedor Docker solo publica servicios con la etiqueta `traefik.enable=true`, evitando exponerlos por defecto.
- El dashboard/API en modo inseguro se enlaza únicamente a localhost. No cambies `TRAEFIK_BIND_ADDRESS` a una interfaz pública sin configurar autenticación, TLS y controles de acceso.
- El socket Docker se monta en modo de solo lectura. Esto no elimina el riesgo inherente de dar acceso a la API del daemon; considera un socket proxy con permisos limitados en entornos expuestos.
- `config/dynamic/` contiene configuración dinámica persistente y está montada en modo de solo lectura. No se crea un almacén ACME porque esta configuración inicial no activa resolutores TLS ni emite certificados.
- `.env` contiene solo valores locales no secretos y queda excluido de Git. No incrustes secretos en imágenes ni los confirmes al repositorio.

## Despliegue clonando el repositorio

Cuando publiques el repositorio por tu cuenta, clónalo y copia la plantilla de entorno:

```bash
git clone <URL_DEL_REPOSITORIO> traefik
cd traefik
cp .env.example .env
docker compose config --quiet
docker compose up -d --pull always
```

El proyecto local no tiene remoto configurado ni se publica automáticamente.

## Despliegue manual sin Git

1. Crea una carpeta `traefik` y dentro guarda `compose.yaml` y el archivo `.env` (cópialo desde `.env.example`).
2. Crea `config/dynamic/` y guarda allí `routes.yml` (se puede empezar con `http: {}`).
3. Desde esa carpeta ejecuta:

```bash
cp .env.example .env
docker compose config --quiet
docker compose up -d --pull always
```

En este proyecto, los archivos ya están organizados de esa manera.

## Variables de entorno

Edita `.env` antes de arrancar. Las variables también están documentadas en `.env.example`:

| Variable | Valor predeterminado | Uso |
| --- | --- | --- |
| `TRAEFIK_IMAGE` | `traefik:latest` | Imagen de Traefik |
| `TRAEFIK_BIND_ADDRESS` | `127.0.0.1` | Interfaz del host donde se publican los puertos |
| `TRAEFIK_HTTP_PORT` | `80` | Puerto HTTP del host |
| `TRAEFIK_HTTPS_PORT` | `443` | Puerto HTTPS del host |
| `TRAEFIK_DASHBOARD_PORT` | `8080` | Puerto local del dashboard |
| `TRAEFIK_LOG_LEVEL` | `INFO` | Nivel de registro |

No guardes credenciales ni otros secretos en `.env` si el archivo se comparte; usa un gestor de secretos para valores sensibles.

## Persistencia y configuración de servicios

- `config/dynamic/routes.yml` y los demás archivos de `config/dynamic/` se conservan en el proyecto y se leen automáticamente.
- Para enrutar otro contenedor, conéctalo a una red accesible por Traefik y añade etiquetas, por ejemplo:

```yaml
labels:
  - traefik.enable=true
  - traefik.http.routers.app.rule=Host(`app.localhost`)
  - traefik.http.routers.app.entrypoints=web
  - traefik.http.services.app.loadbalancer.server.port=8080
```

El puerto de servicio de la etiqueta debe ser el puerto interno donde escucha la aplicación. Para TLS, configura un entrypoint HTTPS y un resolvedor/certificado antes de exponer el servicio públicamente.

## Operación

- Ver estado: `docker compose ps`
- Consultar registros: `docker compose logs -f traefik`
- Validar cambios: `docker compose config --quiet`
- Aplicar cambios y actualizar la imagen: `docker compose up -d --pull always`
- Parar el servicio: `docker compose down`

`docker compose down` conserva los archivos de configuración del proyecto. No borres `.env` ni `config/dynamic/` si necesitas mantener los ajustes.

## Acceso y seguridad

- Dashboard: [http://localhost:8080/dashboard/](http://localhost:8080/dashboard/)
- Entrada HTTP: [http://localhost](http://localhost/)
- HTTPS escucha en el puerto configurado, pero esta configuración inicial no gestiona certificados.
- El dashboard/API sin autenticación no debe exponerse a Internet. El acceso al socket Docker es sensible y puede otorgar control amplio sobre el daemon; limita quién puede modificar este proyecto y protege el host.
