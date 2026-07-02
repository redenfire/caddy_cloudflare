# caddy_cloudflare

Immagine Docker di [Caddy](https://caddyserver.com) con il modulo [caddy-dns/cloudflare](https://github.com/caddy-dns/cloudflare) precompilato, per usare i DNS challenge di Cloudflare con ACME.

## Immagine

Pubblicata su GitHub Container Registry:

```
ghcr.io/redenfire/caddy_cloudflare:latest
```

## Build automatica

GitHub Actions builda e pubblica l'immagine in due occasioni:

- **A ogni push** sul branch `main`
- **Ogni giorno** alle 06:00 UTC (cron `0 6 * * *`), per raccogliere gli aggiornamenti a monte delle immagini base Caddy

## Come usarla

`docker run`:

```bash
docker run -p 80:80 -p 443:443 \
  -v ./Caddyfile:/etc/caddy/Caddyfile \
  -v ./data:/data \
  ghcr.io/redenfire/caddy_cloudflare:latest
```

`docker-compose.yml`:

```yaml
services:
  caddy:
    image: ghcr.io/redenfire/caddy_cloudflare:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./data:/data
```

### Caddyfile con DNS challenge Cloudflare

```
example.com {
    tls {
        dns cloudflare <api_token>
    }
    respond "Hello, world!"
}
```

È consigliabile passare il token via variabile d'ambiente:

```
tls {
    dns cloudflare {env.CLOUDFLARE_API_TOKEN}
}
```

```bash
docker run -e CLOUDFLARE_API_TOKEN=<token> ...
```

## Dockerfile

```dockerfile
FROM caddy:2-builder AS builder
RUN xcaddy build --with github.com/caddy-dns/cloudflare

FROM caddy:2-alpine
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```
