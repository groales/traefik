# Traefik con Docker Compose

Traefik es un proxy inverso que enruta solicitudes a servicios Docker mediante etiquetas. Este proyecto usa la imagen oficial `traefik:latest`, redirige HTTP a HTTPS y contiene un servicio de prueba `whoami`.

## Requisitos y acceso

- Docker Engine o Docker Desktop con Docker Compose v2.
- `openssl` para crear el certificado autofirmado local.
- Puertos TCP 80 y 443 disponibles. Compose los publica en todas las interfaces del host.
- Dashboard: <https://dashboard.docker.localhost/dashboard/>
- Servicio de prueba: <https://whoami.docker.localhost/>. Para `curl`, usa `curl -k https://whoami.docker.localhost/`.

`*.docker.localhost` suele resolver a loopback. El certificado es autofirmado y el cliente lo marcará como no confiable hasta que lo instales como confiable.

## Archivos locales

El repositorio contiene `compose.yaml`. `.gitignore` excluye `.env`, `certs/` y `dynamic/`; cada equipo debe crear `certs/local.crt`, `certs/local.key` y `dynamic/tls.yaml` localmente. El archivo TLS dinámico debe contener:

```yaml
tls:
  certificates:
    - certFile: /certs/local.crt
      keyFile: /certs/local.key
```

El certificado y la clave se montan en Traefik como solo lectura. No compartas ni confirmes `certs/local.key`. Conserva una copia segura de los archivos locales, ya que Git no los sincroniza.

## Despliegue

Clona el repositorio:

```bash
git clone https://github.com/groales/traefik.git
cd traefik
```

O prepara manualmente una carpeta con `compose.yaml`. En ambos casos, desde el directorio del proyecto, crea el certificado y la configuración local:

```bash
mkdir -p certs dynamic
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout certs/local.key -out certs/local.crt \
  -subj "/CN=*.docker.localhost"
cat > dynamic/tls.yaml <<'EOF'
tls:
  certificates:
    - certFile: /certs/local.crt
      keyFile: /certs/local.key
EOF
docker compose config --quiet
docker compose up -d --pull always
```

## Enrutamiento y red

Traefik y `whoami` se conectan a la red Docker local `proxy`. Los servicios definidos en otros proyectos deben conectarse a una red externa llamada `proxy` y usar etiquetas, por ejemplo:

```yaml
services:
  app:
    image: ejemplo/app
    networks: [proxy]
    labels:
      - traefik.enable=true
      - traefik.http.routers.app.rule=Host(`app.docker.localhost`)
      - traefik.http.routers.app.entrypoints=websecure
      - traefik.http.routers.app.tls=true
      - traefik.http.services.app.loadbalancer.server.port=8080

networks:
  proxy:
    external: true
```

El puerto de la etiqueta debe coincidir con el puerto interno de la aplicación. Traefik solo descubre servicios que declaran `traefik.enable=true`.

## Operación y seguridad

- Estado: `docker compose ps`
- Registros: `docker compose logs -f traefik`
- Validar: `docker compose config --quiet`
- Actualizar: `docker compose up -d --pull always`
- Parar: `docker compose down`

El dashboard se sirve por HTTPS, pero el Compose no configura autenticación; el certificado autofirmado cifra la conexión, no autentica al usuario. No expongas el dashboard a redes no confiables sin añadir autenticación y controles de acceso. El socket Docker se monta con `:ro`, pero esto no limita las operaciones que la API del daemon permite. Limita el acceso al host y al repositorio. Para producción, usa certificados de confianza y revisa la exposición de las métricas Prometheus.

## Documentación oficial

- [Instalación oficial con Docker](https://doc.traefik.io/traefik/setup/docker/)
- [Proveedor Docker](https://doc.traefik.io/traefik/reference/install-configuration/providers/docker/)
- [API y dashboard](https://doc.traefik.io/traefik/reference/install-configuration/api-dashboard/)
- [Imagen oficial de Docker Hub](https://hub.docker.com/_/traefik)
