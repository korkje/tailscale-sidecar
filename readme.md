# qbittorrent-tailscale

Docker compose setup for qBittorrent and Tailscale, plus a Firefox instance whose traffic also goes through the Tailscale exit node.

Tailscale Serve routes both apps on the same hostname over HTTPS:

- `https://<hostname>.<tailnet>.ts.net/` -> qBittorrent
- `https://<hostname>.<tailnet>.ts.net/firefox/` -> Firefox (note the trailing slash)

Firefox is served by [linuxserver/firefox](https://github.com/linuxserver/docker-firefox). It has no auth by default; set `CUSTOM_USER` and `PASSWORD` on the service to enable HTTP basic auth.

Create a `.env` file in the root directory and set the following variables:

```
TS_AUTHKEY=<your tailscale auth key>
TS_HOSTNAME=<hostname for the Tailscale service (e.g. qbt)>
TS_EXIT=<exit node IP address (e.g. 1.2.3.4)>
QBT_MEDIA=<path to media directory (e.g. /volume1/media)>
QBT_TZ=<your timezone (e.g. Europe/Oslo)>
```

To disable qBittorrent's auth, set `WebUI\LocalHostAuth=false` under `[Preferences]` in `qbittorrent/config/qBittorrent/qBittorrent.conf` while the container is stopped, but after having run `docker compose up -d` for the first time.
