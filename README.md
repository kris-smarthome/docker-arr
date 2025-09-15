# ARR Stack (Prowlarr, Radarr, Sonarr, qBittorrent, Recyclarr)

Self‑hosted media automation stack using Docker Compose. Includes:

- Prowlarr: indexer aggregator/proxy
- Radarr: movies automation
- Sonarr: TV automation
- qBittorrent: torrent client
- Recyclarr: sync Radarr/Sonarr with TRaSH guides
- FlareSolverr: optional CF bypass helper

Main stack definition: `compose.yaml:1`

## Prerequisites

- Docker and Docker Compose V2 installed
- A host path for media and downloads
- Optional external reverse‑proxy network named `traefik` (used by this stack)

Create the external network if it doesn’t exist:

```bash
docker network create traefik || true
```

## Configure

Edit `.env:1` and set values for your environment:

- `PUID`: user id that owns media files (e.g., 10000)
- `PGID`: group id that owns media files (e.g., 10000)
- `TZ`: timezone (e.g., America/Los_Angeles)
- `MEDIA`: absolute path to your media root (must contain `downloads/`)

Example:

```env
PUID=10000
PGID=10000
TZ=America/Los_Angeles
MEDIA=/srv/media
```

Recommended host layout under `MEDIA`:

```text
/srv/media/
  downloads/        # qBittorrent completed/incomplete
  movies/           # Radarr library
  tv/               # Sonarr library
```

Create Recyclarr config directory (mounted at `/config` in the container):

```bash
mkdir -p recyclarr-app
```

## Create a media user (optional but recommended)

Run services as a dedicated user/group to avoid permission issues. Match `PUID`/`PGID` in `.env`.

```bash
sudo groupadd -g 10000 media || true
sudo useradd -u 10000 -g 10000 -s /bin/bash media || true
sudo usermod -aG media "$USER"
```

## Launch

```bash
docker compose up -d
```

Web UIs (on the Docker host):

- Prowlarr: http://localhost:9696
- Radarr: http://localhost:7878
- Sonarr: http://localhost:8989
- qBittorrent: http://localhost:8080 (default user `admin`, password `adminadmin` if unchanged)
- FlareSolverr: 8191 (not published by default)

## Volumes and Paths

- App configs persist in named volumes: `prowlarr`, `radarr`, `sonarr`, `qbittorrent`.
- Bind mounts from `MEDIA` are used for libraries and downloads.
- Service mounts summary:
  - Prowlarr: `prowlarr:/config` only
  - Radarr: `radarr:/config`, `${MEDIA}/downloads:/data/downloads`, `${MEDIA}:/media`
  - Sonarr: `sonarr:/config`, `${MEDIA}/downloads:/data/downloads`, `${MEDIA}:/media`
  - qBittorrent: `qbittorrent:/config`, `${MEDIA}/downloads:/data/downloads`
  - Recyclarr: `./recyclarr-app:/config`

## Notes on Traefik

- Services attached to the external `traefik` network: Prowlarr, Radarr, Sonarr, qBittorrent.
- FlareSolverr uses only the internal `media` network; Recyclarr does not require external networking for basic use.
- If you don’t use Traefik, remove the `traefik` network block and those service attachments in `compose.yaml:1`; direct ports remain available.

## Recyclarr

- Place your Recyclarr YAML files in `recyclarr-app/` and run Recyclarr commands as needed, for example:

```bash
docker compose run --rm recyclarr list
docker compose run --rm recyclarr sync radarr
docker compose run --rm recyclarr sync sonarr
```

## Troubleshooting

- Permissions: if files are created as `root`, set correct `PUID`/`PGID` and ensure the media path is writable by that user.
- Network not found: create `traefik` via `docker network create traefik` or remove it from `compose.yaml:1`.
- Port conflicts: adjust published ports in `compose.yaml:1` if 9696/7878/8989/8080 are in use.
- Folder structure: ensure `${MEDIA}/downloads` exists; Radarr/Sonarr expect consistent paths across apps.

## Repository Usage

```bash
git clone https://github.com/kris-smarthome/docker-arr.git
cd docker-arr
cp .env .env.example 2>/dev/null || true  # optional backup
${EDITOR:-nano} .env
docker compose up -d
```

If you want, I can also validate the Compose file and address any issues before you deploy.
