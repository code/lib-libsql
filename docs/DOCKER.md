<!-- markdownlint-disable MD025 -->

# Docker image quick reference

## Launch a primary instance

```console
docker run --name some-sqld -p 8080:8080 -ti \
    -e SQLD_NODE=primary \
    ghcr.io/tursodatabase/libsql-server:latest
```

## Launch a replica instance

```console
docker run --name some-sqld-replica -p 8081:8080 -ti \
    -e SQLD_NODE=replica \
    -e SQLD_PRIMARY_URL=https://<host>:<port> \
    ghcr.io/tursodatabase/libsql-server:latest
````

## Running on Apple Silicon

```console
docker run --name some-sqld  -p 8080:8080 -ti \
    -e SQLD_NODE=primary \
    --platform linux/amd64 \
    ghcr.io/tursodatabase/libsql-server:latest
```

_Note: the latest images for arm64 are available under the tag
`ghcr.io/tursodatabase/libsql-server:latest-arm`, however for tagged versions,
and stable releases please use the x86_64 versions via Rosetta._

## Docker Repository

[https://github.com/tursodatabase/libsql/pkgs/container/libsql-server](https://github.com/tursodatabase/libsql/pkgs/container/libsql-server)

# How to extend this image

## Data Persistence

Database files are stored in the `/var/lib/sqld` in the image. To persist the
database across runs, mount this location to either a docker volume or a bind
mount on your local disk.

```console
docker run --name some-sqld -ti \
    -v $(pwd)/sqld-data:/var/lib/sqld \ # you can mount local path
    -e SQLD_NODE=primary \
    ghcr.io/tursodatabase/libsql-server:latest

docker run --name some-sqld -ti \
    -v sqld-data:/var/lib/sqld \ # or create named volume
    -e SQLD_NODE=primary \
    ghcr.io/tursodatabase/libsql-server:latest

docker run --name some-sqld -ti \
    -v sqld-data:/data/sqld \ # to mount data in different directory set SQLD_DB_PATH env var
    -e SQLD_NODE=primary \
    -e SQLD_DB_PATH=/data/sqld \
    ghcr.io/tursodatabase/libsql-server:latest
```

## Authentication

### `SQLD_HTTP_AUTH`

Specifies legacy HTTP basic authentication. The argument must be in format `basic:$PARAM`,
where `$PARAM` is base64-encoded string `$USERNAME:$PASSWORD`.

### `SQLD_AUTH_JWT_KEY_FILE`

Path to a file with a JWT decoding key used to authenticate clients in the Hrana and HTTP
APIs. The key is either a PKCS#8-encoded Ed25519 public key in PEM, or just plain bytes of
the Ed25519 public key in URL-safe base64.

You can also pass the key directly in the env variable SQLD_AUTH_JWT_KEY.

## Environment variables

### `SQLD_NODE`

**default:** `primary`

The `SQLD_NODE` environment variable configures the type of the launched
instance. Possible values are: `primary` (default), `replica`, and `standalone`.
Please note that replica instances also need the `SQLD_PRIMARY_URL` environment
variable to be defined.

### `SQLD_PRIMARY_URL`

The `SQLD_PRIMARY_URL` environment variable configures the gRPC URL of the primary instance for replica instances.

**See:** `SQLD_NODE` environment variable

### `SQLD_DB_PATH`

**default:** `iku.db`

The location of the db file inside the container. Specifying only a filename
will place the file in the default directory inside the container at
`/var/lib/sqld`.

### `SQLD_HTTP_LISTEN_ADDR`

**default:** `0.0.0.0:8080`

Defines the HTTP listen address that sqld listens on and clients will connect
to. Recommended to leave this on the default port and remap ports at the
container networking level.

### `SQLD_GRPC_LISTEN_ADDR`

**default:** `0.0.0.0:5001`

Defines the GRPC listen address and port for sqld. Primarily used for
inter-node communication. Recommended to leave this on default.

## Docker Compose

Simple docker compose for local development:

```yaml
version: "3"
services:
  db:
    image: ghcr.io/tursodatabase/libsql-server:latest
    platform: linux/amd64
    ports:
      - "8080:8080"
      - "5001:5001"
    # environment:
    #   - SQLD_NODE=primary
    volumes:
      - ./data/libsql:/var/lib/sqld
```
