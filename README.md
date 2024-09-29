# Use a custom domain and HTTPS/TLS with Caddy

This is part 3 of the series: [Deluge via Proton VPN with port forwarding](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward).

This part adds Caddy to the stack making the Deluge web client publicly accessible via a custom domain with HTTPS/TLS encryption.

*It is assumed you are using Cloudflare for DNS. If not then refer to the Caddy documentation for your provider:*
- [How to use DNS provider modules in Caddy 2](https://caddy.community/t/how-to-use-dns-provider-modules-in-caddy-2/8148)
- [Automatic HTTPS](https://caddyserver.com/docs/automatic-https)
- [HTTPS quick-start](https://caddyserver.com/docs/quick-starts/https)
- [acme_dns reference](https://caddyserver.com/docs/caddyfile/options#acme-dns)

## Technologies
- [Caddy](https://caddyserver.com/) as a reverse proxy.
- [Cloudflare](https://www.cloudflare.com/) for DNS validation.

## How it works

### Caddy

The `caddy` service runs [Caddy](https://github.com/qdm12/gluetun) which:
- makes the Deluge web client accessible via a custom domain such as `deluge.example.com`
- automatically obtains and renews HTTPS/TLS certificates from Let's Encrypt using [Cloudflare DNS validation](https://github.com/caddy-dns/cloudflare).

## Getting started

### 1. Set the DNS records

Add an `A` record for your domain pointing to your server's IP address.

Be aware that this is publicly exposing your IP address and that it's NOT going via your VPN Provider. For extra security there are a few options, but these are outside the scope of this guide:

- use Cloudflare to hide your server's IP address by enabling the Cloudflare proxy.
- [create a personal VPN](https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-ubuntu-20-04) on your server and bind Docker to only use the local VPN IP address. That way your services are connectable whenever you're connected to your personal VPN.

### 2. Obtain a Cloudflare API token

[By following these steps](https://github.com/caddy-dns/cloudflare/blob/master/README.md#configuration)

### 3. Set the Caddy environment variables

Copy the `caddy/.env.example` file to `caddy/.env` and set:

- `CLOUDFLARE_API_TOKEN` to your Cloudflare API token
- `LETSENCRYPT_EMAIL` to your email address

### 4. Update the `Caddyfile`

Edit `caddy/Caddyfile` replacing `example.com` with your domain.

### 5. Start the services

In part 1 we only edited `docker-compose.yml` and `.env` files.

However, in this part we've modified the `Caddyfile` which is copied into the `caddy` image during the build stage therefore any changes to the `Caddyfile` require the `caddy` service to be rebuilt with the `--build` flag.

Re-build and start up the docker compose services:

```bash
docker compose up -d --build
```

Follow the `caddy` logs to ensure Caddy is able to complete the DNS challenge for a Let's Encrypt certificate:

```bash
docker compose logs -f caddy
```

## Troubleshooting

1. Check the logs
2. Re-read the guide
3. [Ask for help in GitHub Discussions](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward/discussions)
4. [Create an issue](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward/issues)

## Next steps

[Part 3 of this guide adds Prowlarr, cross-seed, and fertilizer](tree/part-3-prowlarr-cross-seed-fertilizer) for a completely automated setup with cross seeding.
