# MinIO

MinIO es un servidor de almacenamiento de objetos compatible con la API de Amazon S3.

## Descripción

Permite almacenar y recuperar archivos (objetos) mediante la API S3. Incluye una consola web para administrar buckets y objetos de forma visual.

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
