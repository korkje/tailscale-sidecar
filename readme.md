# tailscale-sidecar

Docker compose setup where a Tailscale container acts as the network sidecar for other services, so all their traffic goes through a Tailscale exit node. Currently runs qBittorrent and Firefox.

Tailscale Serve routes both apps on the same hostname over HTTPS:

- `https://<hostname>.<tailnet>.ts.net/` -> qBittorrent
- `https://<hostname>.<tailnet>.ts.net/firefox/` -> Firefox (note the trailing slash)

Tailscale's WireGuard port is pinned to UDP 41642 and published, so peers can reach the node directly instead of through a relay. Forwarding UDP 41642 on your router to the host makes direct connections even more reliable.

Firefox is served by [jlesage/firefox](https://github.com/jlesage/docker-firefox) (noVNC, no audio, small image). It has no auth by default; set `VNC_PASSWORD` on the service to require a password. Firefox runs in permanent private browsing mode, so history, cookies and site data are discarded on restart. Installed extensions and their settings persist in `firefox/config`. The dark theme is enabled, the Firefox Account sign-in UI is hidden, and welcome pages and sponsored new-tab content are disabled. All of this is done through `FF_PREF_*` variables on the service.

Create a `.env` file in the root directory and set the following variables:

```
TS_AUTHKEY=<your tailscale auth key>
TS_HOSTNAME=<hostname for the Tailscale service (e.g. sidecar)>
TS_EXIT=<exit node IP address (e.g. 1.2.3.4)>
QBT_MEDIA=<path to media directory (e.g. /volume1/media)>
QBT_TZ=<your timezone (e.g. Europe/Oslo)>
```

To disable qBittorrent's auth, set `WebUI\LocalHostAuth=false` under `[Preferences]` in `qbittorrent/config/qBittorrent/qBittorrent.conf` while the container is stopped, but after having run `docker compose up -d` for the first time.

## Surviving NAS reboots (Synology)

`restart: unless-stopped` is not enough for the two sidecar'd services. On boot, Docker restores
all containers in parallel, and if `qbittorrent`/`firefox` are started before `tailscale` is up
they fail with `cannot join network of a non running container` (exit code 128). Docker does not
retry a failed start, so they stay down until someone runs `docker compose up -d`.

Fix: add a boot-triggered task in DSM that brings the project up once Docker is ready.

Control Panel -> Task Scheduler -> Create -> Triggered Task -> User-defined script:

- General: User `root`, Event `Boot-up`
- Task Settings -> Run command:

```sh
#!/bin/sh
# Wait up to 5 minutes for the Docker daemon, then bring the stack up.
for i in $(seq 1 60); do
  /usr/local/bin/docker info >/dev/null 2>&1 && break
  sleep 5
done
cd /volume1/docker/tailscale-sidecar && /usr/local/bin/docker compose up -d
```

`docker compose up -d` is idempotent, so it is safe to run every boot. Compose already orders the
start correctly because `network_mode: service:tailscale` is an implicit dependency.
