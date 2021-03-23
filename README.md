## Requirements

- Docker Engine: https://docs.docker.com/engine/install/
- Docker Compose: https://docs.docker.com/compose/install/

## Configuration

Traefik has a [static configuration](https://doc.traefik.io/traefik/reference/static-configuration/overview/) (provided by us)
and a [dynamic configuration](https://doc.traefik.io/traefik/providers/docker/) (provided by docker).

In this project the [environment variable based static configuration](https://doc.traefik.io/traefik/reference/static-configuration/env/)
is set within the [`environment` section](https://docs.docker.com/compose/compose-file/compose-file-v3/#environment) of the
`traefik` service. The dynamic configuration is set within the [`label` section](https://docs.docker.com/compose/compose-file/compose-file-v3/#labels)
of the services using reverse proxy.

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

The [Traefik Dashboard](https://doc.traefik.io/traefik/operations/dashboard/) can be accessed on the `traefik` subdomain (e.g.: https://traefik.example.com).
In staging, username and password are both set to _`traefik`_.

### Production

For production you must override `docker-compose.yml` with `docker-compose.prod.yml` in order to use production ready settings.

When using `docker-compose.prod.yml`, credentials for the [Traefik Dashboard](https://doc.traefik.io/traefik/operations/dashboard/)
must be set explicitly.Traefik is configured with the [DigestAuth](https://doc.traefik.io/traefik/middlewares/digestauth/)
middleware for authentification. The digest token can be generated using [`htdigest`](https://httpd.apache.org/docs/2.4/programs/htdigest.html)
and must be passed to `docker-compose` via the `DASHBOARD_DIGESTAUTH_TOKEN` environment variable:

```bash
DASHBOARD_DIGESTAUTH_TOKEN=<digest_token> docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

If `DOMAIN` and `ACME_MAIL` are not set in the `.env` file:

```bash
DOMAIN=<your_domain> ACME_MAIL=<admin_email>  DASHBOARD_DIGESTAUTH_TOKEN=<digest_token> docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```
