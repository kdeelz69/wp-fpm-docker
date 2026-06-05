# WordPress Docker Stack

Docker Compose setup for:

- WordPress with PHP-FPM
- MariaDB
- Nginx
- Purchased TLS certificates or Let's Encrypt with Certbot

## Requirements

- Docker Engine and Docker Compose v2
- Domain DNS records pointing to the server
- Ports `80` and `443` open

## Configure

Create the local environment file:

```bash
cp .env.example .env
```

Update `.env` with:

- `DOMAIN` and `WWW_DOMAIN`
- WordPress site and administrator details
- MariaDB and WordPress database credentials
- `LETSENCRYPT_EMAIL` only when using Let's Encrypt

Use hostnames only for `DOMAIN` and `WWW_DOMAIN`, without `https://`.

## Purchased Certificate

The certificate must be valid for both `DOMAIN` and `WWW_DOMAIN` when both
hostnames are configured.

By default, place the files here:

```text
certificates/fullchain.pem
certificates/privkey.pem
```

- `fullchain.pem` contains the domain certificate followed by any intermediate
  CA certificates.
- `privkey.pem` contains the matching unencrypted private key.

The original filenames and extensions do not matter. If you keep names such as
`domain.pem` and `domain.key`, update `.env`:

```env
TLS_CERTIFICATE_PATH=/etc/nginx/certificates/domain.pem
TLS_CERTIFICATE_KEY_PATH=/etc/nginx/certificates/domain.key
```

The files remain inside the host `certificates/` directory because Docker
mounts that directory at `/etc/nginx/certificates` inside the Nginx container.

Never commit or share the private key. Certificate files in `certificates/`
are ignored by Git.

Start the stack:

```bash
docker compose up -d
docker compose up -d --force-recreate nginx
```

Nginx automatically uses the purchased certificate when both configured files
exist.

Validate the configuration:

```bash
docker compose exec nginx nginx -t
docker compose logs --tail=100 nginx
```

When installing or replacing a purchased certificate, recreate Nginx so it
regenerates its configuration and mounts the current certificate files:

```bash
docker compose up -d --force-recreate nginx
```

## Let's Encrypt

Use this option only when you do not have a purchased certificate.

Set `LETSENCRYPT_EMAIL` in `.env`, then run:

```bash
sh bootstrap-https.sh
```

This command:

1. Starts the Docker services.
2. Starts Nginx over HTTP.
3. Requests a certificate for `DOMAIN` and `WWW_DOMAIN`.
4. Stores it under `certbot/conf/`.
5. Restarts Nginx with HTTPS.

Certbot is included in the Compose stack, but `docker compose up -d` alone does
not request a certificate.

To renew later:

```bash
sh certbot/run-certbot.sh
docker compose restart nginx
```

Nginx prefers a purchased certificate from `certificates/` over a Let's Encrypt
certificate.

## Initial WordPress Setup

On the first startup, `wpcli` waits for MariaDB and WordPress, creates
`wp-config.php` when needed, and installs WordPress using the values from
`.env`. It does not reinstall an existing site.

## Useful Commands

```bash
docker compose ps
docker compose logs --tail=100 nginx
docker compose logs --tail=100 wordpress
docker compose exec nginx nginx -t
docker compose restart nginx
docker compose down
```

## Persistent Data

- WordPress files: `html/`
- MariaDB data: Docker volume `db_data`
- Purchased certificates: `certificates/`
- Let's Encrypt certificates: `certbot/conf/`

MariaDB is exposed only on `127.0.0.1` by default. Use an SSH tunnel for remote
database access rather than exposing port `3306` publicly.

## Troubleshooting

- Confirm `DOMAIN` and `WWW_DOMAIN` point to the server.
- Confirm ports `80` and `443` are open.
- Confirm the certificate and private key match.
- Confirm the certificate covers both configured hostnames.
- Run `docker compose exec nginx nginx -t`.
- Review `docker compose logs --tail=200 nginx`.
