# traefik

Deploy Traefik Service

Основная магия будет происходить при  развёртывании контейнеров.

В контейнере нужно будет прописать метку, по которой traefik поймёт какой 
внутренний сервис дёрнуть по внешнему url

## YAML Template.
---
version: '3'

services:

  basis-webserver:
    # A container that exposes an API to show its IP address
    image: traefik/whoami
    container_name: basis-webserver
    restart: always
    labels:
      - "traefik.http.routers.basis-webserver.rule=Host(`basis.etalon.local`)"      

При обращении к basis.etalon.local трафик будет перенаправлен на порт сервиса
(контейнера) basis-webserver (указано в метке labels)