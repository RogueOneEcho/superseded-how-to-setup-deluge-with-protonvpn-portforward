**This guide is a work in progress. Please check back later.**

[Add a reaction or comment to issue #1 if you'd like to see this completed sooner.](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward/issues/1)

# Automated cross seeding with Prowlarr, cross-seed, and fertilizer

This is part 2 of the series: [Deluge via Proton VPN with port forwarding](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward).

This part adds Prowlarr, cross-seed, and fertilizer to the stack for automatic cross seeding.

## Technologies
- [Prowlarr](https://prowlarr.com/) for managing indexers.
- [cross-seed v6](https://github.com/cross-seed/cross-seed) for cross seeding movie and TV torrents between indexers.
- [Fertilizer](https://github.com/moleculekayak/fertilizer) for cross seeding music torrents between indexers.

## How it works

### Caddy

The `caddy` service runs [Caddy](https://github.com/qdm12/gluetun) which:
- makes the Deluge web client accessible via a custom domain such as `deluge.example.com`
- automatically obtains and renews HTTPS/TLS certificates from Let's Encrypt using [Cloudflare DNS validation](https://github.com/caddy-dns/cloudflare).

## Getting started

### 1. Placeholder

## Troubleshooting

1. Check the logs
2. Re-read the guide
3. [Ask for help in GitHub Discussions](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward/discussions)
4. [Create an issue](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward/issues)
