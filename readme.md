# qbittorrent-tailscale

Docker compose setup for qBittorrent and Tailscale, plus a Firefox instance whose traffic also goes through the Tailscale exit node.

Tailscale Serve routes both apps on the same hostname over HTTPS:

- `https://<hostname>.<tailnet>.ts.net/` -> qBittorrent
- `https://<hostname>.<tailnet>.ts.net/firefox/` -> Firefox (note the trailing slash)

Tailscale's WireGuard port is pinned to UDP 41642 and published, so peers can reach the node directly instead of through a relay. Forwarding UDP 41642 on your router to the host makes direct connections even more reliable.

Firefox is served by [jlesage/firefox](https://github.com/jlesage/docker-firefox) (noVNC, no audio, small image). It has no auth by default; set `VNC_PASSWORD` on the service to require a password. Firefox runs in permanent private browsing mode, so history, cookies and site data are discarded on restart. Installed extensions and their settings persist in `firefox/config`. The dark theme is enabled, the Firefox Account sign-in UI is hidden, and welcome pages and sponsored new-tab content are disabled. All of this is done through `FF_PREF_*` variables on the service.

Create a `.env` file in the root directory and set the following variables:

```
TS_AUTHKEY=<your tailscale auth key>
TS_HOSTNAME=<hostname for the Tailscale service (e.g. qbt)>
TS_EXIT=<exit node IP address (e.g. 1.2.3.4)>
QBT_MEDIA=<path to media directory (e.g. /volume1/media)>
QBT_TZ=<your timezone (e.g. Europe/Oslo)>
```

To disable qBittorrent's auth, set `WebUI\LocalHostAuth=false` under `[Preferences]` in `qbittorrent/config/qBittorrent/qBittorrent.conf` while the container is stopped, but after having run `docker compose up -d` for the first time.
