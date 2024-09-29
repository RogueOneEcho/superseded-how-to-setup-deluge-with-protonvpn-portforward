# Automated cross seeding with Prowlarr, cross-seed, and fertilizer

This is part 2 of the series: [Deluge via Proton VPN with port forwarding](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward).

This part adds Prowlarr, cross-seed, and fertilizer to the stack for automatic cross seeding.

**This part is a work in progress**

[Add a reaction or comment to issue #1 if you'd like to see this completed sooner.](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward/issues/1)

## Technologies
- [Prowlarr](https://prowlarr.com/) for managing indexers.
- [cross-seed v6](https://github.com/cross-seed/cross-seed) for cross seeding movie and TV torrents between indexers.
- [Fertilizer](https://github.com/moleculekayak/fertilizer) for cross seeding music torrents between indexers.

## How it works

### Prowlarr

The `prowlarr` service runs [Prowlarr](https://prowlarr.com/) to manage indexers.

### cross-seed

The `cross-seed` service runs [cross-seed v6](https://www.cross-seed.org/docs/v6-migration) to cross seed movie and TV torrents between indexers.

## Getting started

### 1. Set the volume paths in `docker-compose.yml`

Check the `volumes` section of each service in `docker-compose.yml` and update the paths to match your host system.

The default is to use `/media` for anything downloaded in `deluge` and `/data` for configuration and data volumes of each service. You can change these to suit your needs.

### 2. Set the user and group IDs in `docker-compose.yml`

Check the `user`, `group`, `PUID`, `PGID` settings in the `environment` section of each service in `docker-compose.yml` and update the IDs to match your host system. This ensures that the services run with the correct file permissions.

*Typically, your user ID is `1000` and your group ID is `1000`, but this can vary depending on your system.*

### 3. Configure `cross-seed`

`cross-seed` is highly configurable and they have great documentation. Refer to:
- [Getting Started](https://www.cross-seed.org/docs/basics/getting-started)
- [Options](https://www.cross-seed.org/docs/basics/options#all-options)
- [v6 migration guide](https://www.cross-seed.org/docs/v6-migration)

[Ask for help in GitHub Discussions](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward/discussions) if you get stuck.

### 4. Start the services

Start up the docker compose services:

```bash
docker compose up -d
```

Check the status of the services:

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
```

Continuously watch the statuses (updated every 2 seconds):

```bash
watch --interval 2 docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
```

Follow the `prowlarr` logs to ensure Prowlarr is running:

```bash
docker compose logs -f prowlarr
```

Follow the `cross-seed` logs to ensure your cross-seed config is valid:

```bash
docker compose logs -f cross-seed
```

Follow all logs to ensure everything is running smoothly, although this will be incredibly verbose:

```bash
docker compose logs -f
```

To stop all the services:

```bash
docker compose down
```

## Troubleshooting

1. Check the logs
2. Re-read the guide
3. [Ask for help in GitHub Discussions](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward/discussions)
4. [Create an issue](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward/issues)
