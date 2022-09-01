---
layout: default
title: Servei web
parent: Hipervisor
nav_order: 5
---

# Com montar un servei web bàsic emprant Docker Compose.

Volem montar un servei web de manera senzilla per a tenir un servei expost en línia on fer les nostres proves. Emprarem Plex, que és un servidor de contingut. Tant el podem usar per arxius de vídeo locals, connectant-lo a una base de dades, un disc dur amb recursos, com a gestor de subscripcions de vídeo com Netflix.

Aquest servei el montarem a sobre d'un contenidor Docker.

## Docker Compose
Primer de tot, crearem una carpeta anomenada "Plex" al directori que volguem, i crearem les subcarpetes necessàries per a després muntar els volums.

```bash
mkdir plex
mkdir plex/{database,transcode,media}
cd plex
```
Després, crearem un arxiu anomenat docker-compose.yml amb aquest contingut dins.
```yaml
version: '2'
services:
  plex:
    container_name: plex
    image: plexinc/pms-docker
    restart: unless-stopped
    ports:
      - 32400:32400/tcp
      - 3005:3005/tcp
      - 8324:8324/tcp
      - 32469:32469/tcp
      - 1900:1900/udp
      - 32410:32410/udp
      - 32412:32412/udp
      - 32413:32413/udp
      - 32414:32414/udp
    environment:
      - TZ=Europe/Madrid
      - PLEX_CLAIM=claim-ey6ekAqeQjosd1P
      - ADVERTISE_IP=http://172.16.0.200:32400/
    hostname: plexserver.espardenya.me
    volumes:
      - ./plex/database:/config
      - ./plex/transcode:/transcode
      - ./plex/media:/data
```
És un docker-compose bastant simple. Descarrega la imatge dels repositoris oficials, exposa els ports necessaris per a que el servei funcioni i crea els volums al host per a guardar la configuració d'inici.

L'executarem amb docker-compose up -d als nostres hosts, i els nostres serveis hauríen d'estar operatius a les IP assenyalades.