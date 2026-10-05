# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An Ansible project that provisions a single Ubuntu home server (`demiurge`) as a *arr media stack: Jellyfin, Plex, Sonarr, Radarr, Prowlarr, Bazarr, Seerr and qBittorrent, all run as Docker containers from one compose file.

## Commands

```sh
# Dry run (this is the form in the README; --check makes no changes)
ansible-playbook site.yaml --check --ask-become-pass

# Apply for real
ansible-playbook site.yaml --ask-become-pass

# Show what templated files would change
ansible-playbook site.yaml --check --diff --ask-become-pass

# Syntax check without touching the host
ansible-playbook site.yaml --syntax-check
```

`ansible.cfg` points at `inventory.yaml`, so no `-i` flag is needed. There are no tags defined, and no lint or test configuration in the repo (no `.ansible-lint`, `.yamllint`, or molecule).

Every run targets the real server over SSH (`ansible_host: demiurge`, user `tmod`) — there is no staging host or local test target. Use `--check` unless the user asks to apply.

## Architecture

`site.yaml` has two plays against the single host `arr-stack-server`, both with `become: true`:

1. `unattended-upgrades` — installs the package and copies a static `50unattended-upgrades` apt config.
2. `docker` then `arr-stack` — order matters: `docker` installs Docker Engine + the compose plugin from Docker's apt repo, which `arr-stack` depends on.

The `arr-stack` role is the core of the repo and works in three steps: create the directory tree, render `templates/docker-compose.yaml.j2` to `{{ arr_stack_dir }}/docker-compose.yaml`, then run `docker compose up -d --remove-orphans` in that directory. Adding or changing a service therefore usually means touching two places: the compose template, and the directory list in `roles/arr-stack/tasks/main.yaml` for any new bind-mount path (so it is created with the right ownership before Docker creates it as root).

All tunables live in `roles/arr-stack/defaults/main.yaml` with an `arr_` prefix (paths, PUID/PGID, timezone, qBittorrent ports). Container config dirs are relative bind mounts under `arr_stack_dir` (`/srv/arr`); media and downloads are absolute paths from `arr_media_dir` / `arr_downloads_dir`.

## Things to know before editing

- The compose step uses `ansible.builtin.command`, not a Docker module, so it is skipped under `--check`; a dry run validates the template and directories but not that the stack actually starts. Its `changed_when` is derived from matching `Started` / `Created` / `Recreate` in compose output.
- Role file names use the `.yaml` extension (`main.yaml`), not `.yml`. Keep that consistent.
- Role assumes Ubuntu: the Docker apt repo URL is hardcoded to `linux/ubuntu`, and the unattended-upgrades config uses Ubuntu origins.
- `arr_media_dir` and `arr_downloads_dir` default to paths under `/home/ubuntu`, while the inventory connects as `tmod`. This is a known leftover from how the server was first set up and is kept on purpose — do not "fix" it.
- `roles/unattended-upgrades/templates/50unattended-upgrades` is deployed with `copy`, not `template` — it is not Jinja-rendered, and its `${distro_id}` placeholders are apt's own syntax.
- qBittorrent and Prowlarr use `network_mode: "service:gluetun"`, so they have no network of their own: their ports are published on the `gluetun` service, other containers reach them at hostname `gluetun`, and they must be recreated whenever gluetun is. Do not give them `ports:` or move them off gluetun's network — that is the VPN kill switch.
- The WireGuard config (`wg0.conf` in the repo root, gitignored, contains a private key) is read from the control machine via `arr_wireguard_conf`. Never commit it or print its contents.
- All images are pinned to `latest` (or untagged); Plex runs with `network_mode: host` while the other services publish ports.
- Open TODO from the README: backups.
