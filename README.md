Automates the installing and configuration of a *arr stack server:
  - Jellyfin
  - Plex
  - Sonarr
  - Radarr
  - Bazarr
  - Seerr
  - qbittorrent

I have decided to include both Jellyfin and Plex because I've had some issues using Jellyfin on smart TV devices.

Run the playbook:
`ansible-playbook site.yaml --check --ask-become-pass`

## TODO
- VPN configuration for qbittorrent
- backup

