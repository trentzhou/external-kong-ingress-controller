# External Kong ingress controller

Helm chart to install ingress controller for kong outside the kubernetes cluster.

The official helm chart of `stable/kong` includes an ingress controller. However it installs a kong server by itself.
We probably have installed kong outside the kubernetes cluster, in this case we need to modify the ingress controller to make it work with the external kong server.

The modified kong kubernetes ingress controller can be found at https://github.com/trentzhou/kubernetes-ingress-controller/tree/external-kong. Right now it has not been accepted by the Kong community. The modified version adds an option `--kong-outside-kubernetes` to indicate that the kong service lives outside kubernetes.

Most part of the chart is just copied from [the official kong chart](https://github.com/helm/charts/tree/master/stable/kong). 

## How to use it

If you already have a kong server running, you can just use it. Otherwise you can deploy a new kong server. The simplest way to run kong is to use [the official docker-compose file](https://github.com/Kong/docker-kong/tree/master/compose). You can just clone the repo and run `docker-compose up -d`.

If you would like to run kong on a IPv4/IPv6 dual stack server, just use this `docker-compose.yaml` file:

```yaml
version: '2.1'

volumes:
    kong_data: {}

services:
  kong-migrations:
    image: "${KONG_DOCKER_TAG:-kong:latest}"
    command: kong migrations bootstrap
    depends_on:
      db:
        condition: service_healthy
    environment:
      KONG_DATABASE: postgres
      KONG_PG_DATABASE: ${KONG_PG_DATABASE:-kong}
      KONG_PG_HOST: localhost
      KONG_PG_PASSWORD: ${KONG_PG_PASSWORD:-kong}
      KONG_PG_USER: ${KONG_PG_USER:-kong}
    network_mode: host
    restart: on-failure
  kong:
    image: "${KONG_DOCKER_TAG:-kong:latest}"
    user: "${KONG_USER:-root}"
    depends_on:
      db:
        condition: service_healthy
    environment:
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: '[::]:8001'
      KONG_PROXY_LISTEN: '[::]:8000'
      KONG_CASSANDRA_CONTACT_POINTS: db
      KONG_DATABASE: postgres
      KONG_PG_DATABASE: ${KONG_PG_DATABASE:-kong}
      KONG_PG_HOST: localhost
      KONG_PG_PASSWORD: ${KONG_PG_PASSWORD:-kong}
      KONG_PG_USER: ${KONG_PG_USER:-kong}
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
    network_mode: host
    ports:
      - "8000:8000/tcp"
      - "8001:8001/tcp"
      - "8443:8443/tcp"
      - "8444:8444/tcp"
    restart: on-failure
  db:
    image: postgres:9.5
    environment:
      POSTGRES_DB: ${KONG_PG_DATABASE:-kong}
      POSTGRES_PASSWORD: ${KONG_PG_PASSWORD:-kong}
      POSTGRES_USER: ${KONG_PG_USER:-kong}
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "${KONG_PG_USER:-kong}"]
      interval: 30s
      timeout: 30s
      retries: 3
    restart: on-failure
    stdin_open: true
    tty: true
    network_mode: host
    volumes:
      - kong_data:/var/lib/postgresql/data

```

Then you can deploy the external kong ingress controller.

```
helm install --namespace=$NAMESPACE --name=$NAME --set kong.address=$KONG_ADDRESS chart/external-kong-ingress-controller
```


