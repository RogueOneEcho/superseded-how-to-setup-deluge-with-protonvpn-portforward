# Deluge with Proton VPN port forwarding

This guide shows how to run Deluge via ProtonVPN with port forwarding.

But due to the flexibility of [Gluetun](https://github.com/qdm12/gluetun) it can be easily adapt it to work with any wireguard or OpenVPN VPN provider by following their [wiki documentation](https://github.com/qdm12/gluetun-wiki).

*Prior knowledge of Docker, Deluge, Prowlarr, and linuxserver.io containers are assumed.*

## Technologies
- [Docker Compose](https://docs.docker.com/compose/install/)
- [Deluge - linuxserver.io container](https://fleet.linuxserver.io/image?name=linuxserver/deluge)
- [Gluetun](https://github.com/qdm12/gluetun)
- [Proton VPN](https://protonvpn.com/)
- [WireGuard](https://www.wireguard.com/)
- [Caddy](https://caddyserver.com/)
- [Prowlarr - linuxserver.io container](https://fleet.linuxserver.io/image?name=linuxserver/prowlarr)

## How it works

### VPN

The `vpn` service runs  [Gluetun](https://github.com/qdm12/gluetun) which handles all the networking, connecting to the VPN and obtaining a forwarded port.

The other services use `vpn` as their network due to `network_mode: "service:vpn"`.

### Preflight

The `preflight` service runs a [bash script](preflight/preflight) to check:
- the mountpoint is working
- the external ip https://ipinfo.io/ip is reported as the VPN.

Only after `preflight` exits successfully do the `deluge` and `prowlarr` services start due to:

```yaml
depends_on:
  preflight:
    condition: service_completed_successfully
```

### Monitor

The `monitor` service runs a [bash script](monitor/monitor) that obtains the forwarded port from the gluetun API and sets the listening port of `deluge`.

If the monitor script fails it retries waits for a duration defined as `RETRY`, until it's successful. Then it waits a duration defined by `INTERVAL` before running again, therefore if the forwarded port happens to change it will be updated in `deluge`.

### Deluge

The `deluge` service runs a [Deluge](https://deluge-torrent.org/).

### Prowlarr

The `prowlarr` service runs a [Prowlarr](https://prowlarr.com/) connect with indexers.

### Caddy

The `caddy` service runs a [Caddy](https://caddyserver.com/) reverse proxy providing access to the deluge web client and prowlarr web client over HTTPS/TLS.

## Getting started

### 1. Create a WireGuard configuration

From the ProtonVPN website, [create a WireGuard configuration](
https://account.proton.me/u/3/vpn/WireGuard).

Use a memorable `device/certificate name` so you can easily revoke or extend it later.

Set the platform to `GNU/Linux`.

Set `Level for NetShield blocker filtering` to `No filter` - probably not required.

Set `Moderate NAT` to `Yes` - probably not required.

Set `NAT-PMP (Port Forwarding)` to `Yes` - **important**.

Set `VPN Accelerator` to `Yes` - probably not required.

Select a server location from `Standard server configs` that has the `⇆` icon for `P2P`. Avoid any TOR servers.

Save the file as you won't be able to read the private key again.

The file should look something like this:

```ini
[Interface]
# Key for deluge
# Bouncing = 14
# NetShield = 0
# Moderate NAT = on
# NAT-PMP (Port Forwarding) = on
# VPN Accelerator = on
PrivateKey = CIGiABCDEFGkNDgXCiyidFc61ybHJ1S5ufvUd2NNG3k=
Address = 10.2.0.2/32
DNS = 10.2.0.1

[Peer]
# CH#999
PublicKey = n+45suABCDEFGuZWtCnzGkXNBCgJB3wFZYIlBltpORM=
AllowedIPs = 0.0.0.0/0
Endpoint = 203.0.113.1:51820
```


### 2. Update the `vpn` environment variables

In `docker-compose.yml` find the `vpn` service and update the `environment` variables by copying the values from the WireGuard configuration file:

- `PrivateKey` to `WIREGUARD_PRIVATE_KEY`
- `Address` to `WIREGUARD_ADDRESSES`
- `PublicKey` to `WIREGUARD_PUBLIC_KEY`
- The IP address from `Endpoint` to `WIREGUARD_ENDPOINT` - `149.88.27.235` in this example.
- The port from `Endpoint` to `WIREGUARD_PORT` - `51820` in this example.

*For other VPN providers, you can follow their documentation to create a WireGuard configuration, or refer to the [Gluetun documentation](https://github.com/qdm12/gluetun-wiki).*

If you're using a VPN provider who use pre-shared keys then also:

- `PreSharedKey` to `WIREGUARD_PRESHARED_KEY`

### 3. Update the `preflight` environment variables

In `docker-compose.yml` find the `preflight` service and update the `environment` variables by copying the values from the WireGuard configuration file:

- Copy IP address from `Endpoint` to `EXPECTED_IP` - `149.88.27.235` in this example.

### 4. Update the `monitor` environment variables

The `monitor` service loads environment variables from both the `environment` section of `docker-compose.yml` and a `monitor/.env` file. This is better practice for storing sensitive data such as passwords, you can even go a step further and use [secrets](https://docs.docker.com/compose/how-tos/use-secrets/) but that's out of scope here.

Copy the `monitor/.env.example` file to `monitor/.env` and set the `DELUGE_PASSWORD` variable to your deluge web client password.

```bash
cp monitor/.env.example monitor/.env
```

You can also adjust the frequency and verbosity of the monitor: in `docker-compose.yml` find the `monitor` service and update the `environment` variables as follows:

While getting started and testing the following values work well:

```yml
LOG_LEVEL: info
INTERVAL: 30s
RETRY: 2s
```

But those are verbose and frequent, so for production use:

```yml
LOG_LEVEL: warn
INTERVAL: 5m
RETRY: 2s
```

`INTERVAL` is the time between successful runs of the monitor script, and `RETRY` is the time to wait before retrying when an error is encountered.

### 5. Set the volume paths in `docker-compose.yml`

Check the `volumes` section of each service in `docker-compose.yml` and update the paths to match your host system.

The default is to use `/media` for anything downloaded in `deluge` and `/data` for configuration and data volumes of each service. You can change these to suit your needs.

NOTE: By default deluge will download to `/downloads`, so this will need to be modified to `/media/deluge` (or similar) in the deluge web client settings.

If you're using a value other than `/media` then also update the `MOUNTPOINT` environment variable of `preflight`.

### 6. Configure Caddy

This example includes a configuration for proxying `https://deluge.example.com` and `https://prowlarr.example.com` using [Caddy](https://caddyserver.com/).

If you don't want this then simply remove the `caddy` service from `docker-compose.yml`.

This step requires knowledge of domains, DNS configuration, Cloudflare, and Caddy configuration so you may want to skip this step if you're not familiar with these technologies.

Copy the `caddy/.env.example` file to `caddy/.env` and set:

- `CLOUDFLARE_API_TOKEN` to your token [obtained with these steps] (https://github.com/caddy-dns/cloudflare/blob/master/README.md#configuration)
- `LETSENCRYPT_EMAIL` to your email address

```bash
cp caddy/.env.example caddy/.env
```

Edit `caddy/Caddyfile` replacing `example.com` with your domain.


### 7. Start the services


Start the services detached with:

```bash
docker compose up
```

### 8. Follow the logs

Follow the `vpn` logs to ensure the VPN connection is successful.

```bash
docker compose logs -f vpn
```

Follow the `preflight` logs to ensure the VPN connection is successful and the mountpoint is correct:

```bash
docker compose logs -f preflight
```

Follow the `monitor` logs to ensure `gluetun` was able to get a forwarded port and set it in `deluge`:

```bash
docker compose logs -f monitor
```

Follow all logs to ensure everything is running smoothly:

```bash
docker compose logs -f
```

### 9. Access the services

The services should now be running at the domains you defined.

Don't forget to set up your Deluge and Prowlarr clients with the correct settings, and go back to step 4 and revise the `monitor` settings for production use.

## Acknowledgements

This guide is based on the great work of [xitation](https://github.com/xitation/protonvpn-deluge-gluetun-portforward).
