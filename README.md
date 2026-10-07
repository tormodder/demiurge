Ansible playbooks and other config files for my home server.

# Arr-Stack
Automates the installing and configuration of a *arr stack server:
  - Jellyfin
  - Plex
  - Sonarr
  - Radarr
  - Bazarr
  - Seerr
  - qbittorrent
  - Healarr
  - Cleanuparr

I have decided to include both Jellyfin and Plex because I've had some issues using Jellyfin on smart TV devices.

Run the playbook:
`ansible-playbook site.yaml --ask-become-pass`

## VPN
qbittorrent and Prowlarr only reach the internet through a Proton VPN WireGuard tunnel (gluetun container). Download a WireGuard config from your VPN provider with NAT-PMP (port forwarding) enabled and save it as `wg0.conf` in the repo root before running the playbook. The file is gitignored.

In the qbittorrent web UI, enable "Bypass authentication for clients on localhost" so the forwarded port can be set automatically.

## Library maintenance
Healarr and Cleanuparr are deployed unconfigured; connect them to Sonarr, Radarr and the media servers in their web UIs. From Cleanuparr, qbittorrent is reached at `http://gluetun:8080`.

Healarr runs in dry-run mode (report only) until `arr_healarr_dry_run` is set to `false`.

## TODO
- backup

