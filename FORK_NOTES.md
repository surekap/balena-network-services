# Fork Notes

This repository tracks the upstream `balena-pihole` project and adds extra services for remote access and dynamic DNS.

## Upstream Sync Status

As of 2026-03-16, `master` already contains `origin/master`. The fork-specific surface area is intentionally limited to:

- `docker-compose.yml`
- `ddclient-cloudflare/`
- `nginx/`

That smaller diff should remain the focus during future upstream merges.

## Fork-Specific Additions

The fork keeps the upstream Pi-hole and Unbound pairing, then layers on:

- `unifi` via `ryansch/unifi-rpi:5.10.19-arm32v7`
- `unms` via `oznu/unms:armhf`
- `nginx` as the TLS reverse proxy for `unifi.<DOMAIN>` and `unms.<DOMAIN>`
- `ddclient-cloudflare` to keep those hostnames updated in Cloudflare DNS

## Compatibility Notes

- Pi-hole stays on `network_mode: host` so DNS continues to behave like upstream. Because of that, it is not reverse-proxied through the `nginx` container.
- `nginx` can run with either Let's Encrypt certificates using the Cloudflare DNS challenge or locally generated self-signed certificates.
- `ddclient-cloudflare` assumes `DOMAIN` is the Cloudflare zone name, for example `example.com`.
- The `unifi` and `unms` images are pinned to older ARM32-compatible images. They are outside the scope of the upstream project and should be validated independently before major platform upgrades.

## Merge Guidance

When pulling future upstream changes:

1. Merge or rebase upstream into `master`.
2. Re-check `docker-compose.yml` for service wiring changes around Pi-hole and Unbound.
3. Re-check `nginx/setup-and-run.sh` and `ddclient-cloudflare/conf/ddclient.conf` because they contain the fork-specific routing and DNS assumptions.
4. Smoke-test Pi-hole DNS, `unifi.<DOMAIN>`, and `unms.<DOMAIN>` after the merge.
