# MinIO

MinIO es un servidor de almacenamiento de objetos compatible con la API de Amazon S3.

## Descripción

Permite almacenar y recuperar archivos (objetos) mediante la API S3. Incluye una consola web para administrar buckets y objetos de forma visual.

### Imagen

MinIO dejó de publicar imágenes oficiales (`quay.io/minio/minio` y `minio/minio` ya no se pueden descargar). Se usa [`pgsty/minio`](https://hub.docker.com/r/pgsty/minio) de Docker Hub, un fork comunitario mantenido por Pigsty que es compatible: mismo comando `server`, mismas variables `MINIO_ROOT_*` e incluye `mc`.

La versión está fijada en _compose.yaml_ (`RELEASE.AAAA-MM-DDT...`). Para actualizar, revisar los [tags disponibles](https://hub.docker.com/r/pgsty/minio/tags), cambiar la versión y ejecutar `docker compose up -d`.

## Configuración

### Variables de entorno

Crea un archivo `.env` con las siguientes variables:

```bash
MINIO_ROOT_USER=admin
MINIO_ROOT_PASSWORD=password123
MINIO_SERVER_PORT=9000
MINIO_WEB_PORT=9090
```

### Puertos

- **9000**: API S3
- **9090**: Consola web

## Uso

### Iniciar los servicios

```bash
docker compose up -d
```

### Acceder a la consola web

Abre tu navegador en: `http://localhost:9090`

Inicia sesión con las credenciales definidas en `.env` (`MINIO_ROOT_USER` y `MINIO_ROOT_PASSWORD`).

### Configurar tu aplicación

Configura tu aplicación para usar MinIO como almacenamiento S3 con los siguientes parámetros:

- **Endpoint**: `http://localhost:9000`
- **Access Key**: valor de `MINIO_ROOT_USER`
- **Secret Key**: valor de `MINIO_ROOT_PASSWORD`
- **Region**: (cualquier valor, ej. `us-east-1`)
- **Path style**: habilitado

### API S3

Listar buckets:

```bash
curl http://localhost:9000 \
  --aws-sigv4 "aws:amz:us-east-1:s3" \
  -u "admin:password123"
```

### Cliente mc (MinIO Client)

```bash
# Agregar el servidor como alias
mc alias set local http://localhost:9000 admin password123

# Crear un bucket
mc mb local/mi-bucket

# Subir un archivo
mc cp archivo.txt local/mi-bucket/

# Listar objetos
mc ls local/mi-bucket
```

### Detener los servicios

```bash
docker compose down
```

## Backups

Los backups se hacen **por bucket**: se descarga el contenido completo del bucket y se comprime en un `.tar.gz`. Los archivos se guardan en **/backups**.

Para backups automáticos (a otro servidor con rsync o a AWS S3 Glacier Deep Archive), cómo descargarlos y cómo restaurarlos en una instancia nueva, ver **[BACKUPS.md](BACKUPS.md)**.

### Cliente S3 (servicio `backup`)

_compose.yaml_ incluye el servicio `backup` con `aws-cli`, ya configurado con las credenciales del _.env_ y conectado a MinIO. No se inicia con `docker compose up`; se usa con `docker compose run`:

```bash
# Listar buckets
docker compose run --rm backup s3 ls

# Listar objetos de un bucket
docker compose run --rm backup s3 ls s3://mi-bucket --recursive
```

### Backup manual

Reemplazar `mi-bucket` por el bucket a respaldar (con `sudo`, porque los archivos descargados quedan con dueño root):

```bash
docker compose run --rm backup s3 sync s3://mi-bucket /backups/mi-bucket --only-show-errors
sudo tar -czf mi-bucket-$(date +%Y%m%d-%H%M).tar.gz -C backups/mi-bucket .
sudo rm -rf backups/mi-bucket
```

### Restaurar backup

Ver [Restaurar en una instancia nueva](BACKUPS.md#restaurar-en-una-instancia-nueva).
