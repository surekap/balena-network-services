# balena-pihole

This repository started as a fork of the upstream `balena-pihole` project and keeps the original Pi-hole plus Unbound stack intact while adding a few self-hosted network services around it.

The current stack includes:

- [Pi-hole](https://hub.docker.com/r/pihole/pihole/) with [PADD](https://github.com/jpmck/PADD)
- [Unbound](https://nlnetlabs.nl/projects/unbound/about/)
- `nginx` for TLS termination
- `ddclient` for Cloudflare-backed dynamic DNS updates
- `unifi`
- `unms`

Fork-specific rationale and merge guidance live in [FORK_NOTES.md](./FORK_NOTES.md).

## Architecture

- Pi-hole forwards DNS requests to Unbound on `127.0.0.1:1053`, matching the upstream design.
- Pi-hole stays on host networking so DNS remains directly available on the device IP.
- `nginx` publishes HTTPS for `unifi.<DOMAIN>` and `unms.<DOMAIN>`.
- `ddclient` updates those public DNS records in Cloudflare when the WAN address changes.

Pi-hole is intentionally not reverse-proxied through `nginx`; access it directly on the device IP, typically `http://<device-ip>/admin/`.

## Getting Started

Sign up for a free [balenaCloud](https://www.balena.io/cloud) account, provision the device, then deploy this repository with Git or the balena CLI.

Reference: <https://www.balena.io/docs/learn/getting-started>

## Configuration

### Application Variables

| Name | Example | Purpose |
| --- | --- | --- |
| `TZ` | `Asia/Kolkata` | Timezone shared across services. |
| `DOMAIN` | `example.com` | Base domain used for `unifi.<DOMAIN>` and `unms.<DOMAIN>`. |
| `CLOUDFLARE_LOGIN` | `user@example.com` | Cloudflare account email used by `ddclient` and optional Let's Encrypt DNS validation. |
| `CLOUDFLARE_APIKEY` | `...` | Cloudflare global API key used by `ddclient` and optional Let's Encrypt DNS validation. |
| `LETSENCRYPT` | `1` | If set, `nginx` requests Let's Encrypt certificates via the Cloudflare DNS challenge. If unset, self-signed certificates are generated instead. |

### Optional Variables

| Name | Default | Purpose |
| --- | --- | --- |
| `DDCLIENT_UPDATE_SECONDS` | `600` | How often `ddclient` checks for a public IP change. |
| `DDCLIENT_EXTRAARGS` | empty | Extra flags passed to `ddclient`. |
| `NODDNS` | empty | If set, disables the `ddclient` service startup. |
| `CERT_COUNTRY` | `UK` | Country code used when generating self-signed certificates. |

### Pi-hole Variables

| Service | Name | Value | Purpose |
| --- | --- | --- | --- |
| `pihole` | `DNS1` | `127.0.0.1#1053` | Primary upstream DNS target, pointing Pi-hole at Unbound. |
| `pihole` | `DNS2` | `127.0.0.1#1053` | Secondary upstream DNS target. |
| `pihole` | `DNSMASQ_LISTENING` | `eth0` | Use `wlan0` instead when the device is attached by Wi-Fi. |
| `pihole` | `INTERFACE` | `eth0` | Interface Pi-hole should bind to. |
| `pihole` | `IPv6` | `False` | Disables IPv6 inside this stack unless you explicitly want it. |
| `pihole` | `ServerIP` | `<device LAN IP>` | Required for full Pi-hole blocking behavior. |
| `pihole` | `WEBPASSWORD` | `mysecretpassword` | Optional password for the Pi-hole admin UI. |

## Access Patterns

- Pi-hole admin: `http://<device-ip>/admin/`
- Unifi: `https://unifi.<DOMAIN>/`
- UNMS: `https://unms.<DOMAIN>/`

Direct container ports are still exposed for `unifi` and `unms`, but the intended public path is through `nginx`.

## Compatibility Notes

- Upstream sync is currently clean: local `master` already contains `origin/master`.
- The fork-specific changes are concentrated in `docker-compose.yml`, `nginx/`, and `ddclient-cloudflare/`, which keeps future upstream merges manageable.
- `unifi` and `unms` rely on older ARM-compatible images and should be validated before any broader base-image or device-architecture change.

## Usage

- Pi-hole with Unbound: <https://docs.pi-hole.net/guides/unbound/>
- balena forums: <https://forums.balena.io>

## Author

Upstream project: Kyle Harding <kylemharding@gmail.com>

## Acknowledgments

- <https://github.com/pi-hole/docker-pi-hole/>
- <https://docs.pi-hole.net/guides/unbound/>
- <https://github.com/folhabranca/docker-unbound>
- <https://github.com/MatthewVance/unbound-docker>
- <https://nlnetlabs.nl/documentation/unbound>
- <https://firebog.net/>

## License

[MIT License](./LICENSE)
