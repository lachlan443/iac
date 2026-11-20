# iac

Personal homelab infrastructure managed via GitOps.

## Servers

| Host | Role |
|------|------|
| **sweet** | Home server — media, arr stack, home automation |
| **sour** | Oracle Always Free VPS — monitoring, DNS, reverse proxy |

## Routing (Traefik)

Routes are defined as Docker labels on each service in the compose files, so adding or changing a site is a PR.
Merging to `main` deploys to the affected host(s).

To publish a new service, add labels to its compose block (and put it on a network Traefik shares):

```yaml
    labels:
      - traefik.enable=true
      - traefik.docker.network=proxy            # or `media` for services on the media network
      - traefik.http.routers.myapp.rule=Host(`myapp.lbanks.dev`)
      - traefik.http.routers.myapp.entrypoints=websecure
      - traefik.http.routers.myapp.tls=true
      - traefik.http.routers.myapp.middlewares=auth@file   # omit for no SSO
      - traefik.http.services.myapp.loadbalancer.server.port=8080
```

- Shared middlewares (`auth`) live in `docker/<host>/traefik/dynamic/middlewares.yml`.
- `traefik.yml` (entrypoints, ACME/Cloudflare resolver, providers), `acme.json` and the Cloudflare token `.env` stay on the host under `/home/lach/configs/traefik/`.
- Traefik reads containers through a read-only `docker-socket-proxy`, never the raw Docker socket.
- DNS (CoreDNS `Corefile`) is host-only and still needs a record for new hostnames.
