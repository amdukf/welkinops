# Deploying MinIO with Docker Compose with Traefik

This guide explains how to deploy [MinIO](https://min.io/) with Docker Compose and Traefik as a reverse proxy.

## Prerequisites

Before proceeding, ensure the following prerequisites are met:

- Docker installed on your system.
- Docker Compose installed.
- A domain name pointed to your server (e.g., `minio.yourdomain.com`).

## Step 1: Project Setup

1. Create a directory for your MinIO and Traefik setup:

```
   mkdir minio-traefik
   cd minio-traefik
```
2. Create docker-compose.yaml file in the minio-traefik directory
```
version: "3.3"

services:
  minio:
    image: minio/minio:RELEASE.2024-07-26T20-48-21Z
    networks:
      - traefik-proxy2
    volumes:
      - minio-data:/data
    command:
      - server
      - /data
      - --console-address
      - ":9001"
    environment:
      - MINIO_ROOT_USER=minioadmin
      - MINIO_ROOT_PASSWORD=uikui5choRith0ZieVg2zohN5aish5r
      - MINIO_BROWSER_REDIRECT_URL=http://console.minio.mydomain.com
    deploy:
      labels:
        - traefik.enable=true
        - traefik.docker.network=traefik-proxy2
        - traefik.constraint-label=traefik-proxy2
        - traefik.http.routers.minio.service=minio
        - traefik.http.routers.minio.rule=Host(`minio.mydomain.com`)
        - "traefik.http.routers.minio.entrypoints=web"
        - "traefik.http.routers.minio-secure.entrypoints=websecure"
        - "traefik.http.routers.minio-secure.service=minio-secure"
        - traefik.http.services.minio.loadbalancer.server.port=9000
        - "traefik.http.routers.minio-secure.tls.certresolver=myresolver"

        - traefik.http.routers.minio-console.service=minio-console
        - traefik.http.routers.minio-console.rule=Host(`console.minio.mydomain.com`)
        - "traefik.http.routers.minio-console.entrypoints=web"
        - "traefik.http.routers.minio-console-secure.entrypoints=websecure"
        - "traefik.http.routers.minio-console-secure.service=minio-console-secure"
        - traefik.http.services.minio-console.loadbalancer.server.port=9001
        - "traefik.http.routers.minio-console-secure.tls.certresolver=myresolver"

  # Traefik
  traefik:
    image: traefik:v2.10
    container_name: traefik
    restart: unless-stopped
    command:
      - "--api.insecure=true                                                 
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--log.level=DEBUG"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.myresolver.acme.email=example@gmail.com"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
      - "--certificatesresolvers.myresolver.acme.httpchallenge=true"
      - "--certificatesresolvers.myresolver.acme.httpchallenge.entrypoint=web"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - "./letsencrypt:/letsencrypt"
    networks:
      - traefik-proxy2
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.entrypoints=web"
      - "traefik.http.routers.traefik.service=traefik"
      - "traefik.http.services.traefik.loadbalancer.server.port=8080"
      - "traefik.http.routers.traefik-secure.entrypoints=websecure"
      - "traefik.http.routers.traefik-secure.tls.certresolver=myresolver"

volumes:
  minio-data:

networks:
  traefik-proxy2:
    external: true
```
3. Deploy the minio with docker-compose.yaml file:
```
docker compose up -d
```
