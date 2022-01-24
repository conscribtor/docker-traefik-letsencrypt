# Docker + Traefik + Let’s Encrypt

## Requirements

- Docker Engine: https://docs.docker.com/engine/install/
- Docker Compose: https://docs.docker.com/compose/install/

## Configuration

_Traefik_ has a [static configuration](https://doc.traefik.io/traefik/reference/static-configuration/overview/)
(provided by us) and a [dynamic configuration](https://doc.traefik.io/traefik/providers/docker/) (provided by _Docker_).

In this project the 
[environment variable based static configuration](https://doc.traefik.io/traefik/reference/static-configuration/env/)
is set within the [`environment` section](https://docs.docker.com/compose/compose-file/compose-file-v3/#environment) of
the `traefik` service. The dynamic configuration is set within the
[`labels` section](https://docs.docker.com/compose/compose-file/compose-file-v3/#labels) of services using the
reverse proxy.

## Setup

### Staging

If `DOMAIN` and `ACME_MAIL` are set in the `.env` file, simply run:

```bash
docker-compose up -d
```

If you prefer to pass these values explicitely to the `docker-compose` command:

```bash
DOMAIN=<your_domain> ACME_MAIL=<admin_email> docker-compose up -d
```

The [_Traefik Dashboard_](https://doc.traefik.io/traefik/operations/dashboard/) can be accessed on its subdomain
(e.g.: https://traefik.example.com). In staging, username and password are both set to "`traefik`".

### Production

For production you must override `docker-compose.yml` with `docker-compose.prod.yml` in order to use production ready
configurations.

When using `docker-compose.prod.yml`, credentials for the
[_Traefik Dashboard_](https://doc.traefik.io/traefik/operations/dashboard/) must be set explicitly. The `traefik`
service is configured with the [_DigestAuth_](https://doc.traefik.io/traefik/middlewares/http/digestauth/) middleware for
authentification. The digest token can be generated using 
[_htdigest_](https://httpd.apache.org/docs/2.4/programs/htdigest.html) and must be passed to `docker-compose` via the
`DASHBOARD_DIGESTAUTH_TOKEN` environment variable:

```bash
DASHBOARD_DIGESTAUTH_TOKEN=<digest_token> \
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

If `DOMAIN` and `ACME_MAIL` are not set in the `.env` file:

```bash
DOMAIN=<your_domain> ACME_MAIL=<admin_email>  DASHBOARD_DIGESTAUTH_TOKEN=<digest_token> \
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## Connect services

By default, containers running on the same _Docker Engine_ are neither connected to the same docker network as the
`traefik` service, nor are they discovered if they are. Containers must be connected to the `reverse_proxy` network and
be labeled with `traefik.enable=true` and `traefik.http.routers.<routername>.tls.certresolver=letsencrypt` in order to
be served trough the `traefik` service with a _Let's Encrypt_ certificate.

Here is an example of `docker-compose.yml` for a very simple webserver being served trough `traefik` using the
aforementioned `labels` and `networks` configuration:

```yaml
services:
  static-webserver:
    image: python
    volumes:
      - ./static-webserver:/static-webserver:ro
    working_dir: /static-webserver
    command: python -m http.server 443
    expose: [443]
    labels:
      - traefik.enable=true
      - traefik.http.routers.static-webserver.tls.certresolver=letsencrypt
networks:
  default:
    external: true
    name: reverse_proxy
```

`Traefik` is preconfigured with a [default rule](https://doc.traefik.io/traefik/providers/docker/#defaultrule) that will
use the application's service name to route the matching subdomain to the application.
(e.g.: `static-webserver` is available at https://static-webserver.example.com). This behaviour can be changed by
setting a [custom rule](https://doc.traefik.io/traefik/routing/routers/#rule) in the application's `labels`.
