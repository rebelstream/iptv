<div align="center">

# IPTV

**Self-hosted IPTV server for live TV and sports**

[![Docker Pulls](https://img.shields.io/docker/pulls/rebelstream/iptv?style=for-the-badge&color=2496ED)](https://hub.docker.com/r/rebelstream/iptv)
[![Docker Image Size](https://img.shields.io/docker/image-size/rebelstream/iptv/latest?style=for-the-badge&color=1D63ED)](https://hub.docker.com/r/rebelstream/iptv)
[![GitHub Release](https://img.shields.io/github/v/release/rebelstream/iptv?style=for-the-badge&color=22C55E)](https://github.com/rebelstream/iptv/releases/latest)
[![License](https://img.shields.io/badge/license-proprietary-8B5CF6?style=for-the-badge)](LICENSE)

Aggregates live TV channels and sports events into a single M3U playlist with full EPG guide data.<br>
Streams are fully proxied so upstream sources are never exposed to clients.<br>
Designed for **Jellyfin**, **Plex**, **Emby**, and other IPTV apps.

</div>

---

## Screenshots

<details>
<summary><strong>Dashboard</strong> — system overview with live sports</summary>
<br>

![Dashboard](screenshots/dashboard.png)
</details>

<details>
<summary><strong>Channels</strong> — searchable channel list with categories</summary>
<br>

![Channels](screenshots/channels.png)
</details>

<details>
<summary><strong>Sports</strong> — live events sorted by start time</summary>
<br>

![Sports](screenshots/sports.png)
</details>

<details>
<summary><strong>Channel Detail</strong> — video player with program guide</summary>
<br>

![Channel Detail](screenshots/channel-detail.png)
</details>

<details>
<summary><strong>Sport Event</strong> — live scores from ESPN</summary>
<br>

![Sport Event](screenshots/event-detail.png)
</details>

<details>
<summary><strong>Guide</strong> — EPG program schedule</summary>
<br>

![Guide](screenshots/guide.png)
</details>

<details>
<summary><strong>Settings</strong> — API key management and server info</summary>
<br>

![Settings](screenshots/settings.png)
</details>

## Features

- **176+ TV channels** with logos and persistent channel numbers
- **Live sports events** (NHL, NFL, NBA, MLB, Soccer, PPV, and more) with accurate start times
- **Channel categories** -- Sports, News, Local, Kids, Movies, Entertainment, Lifestyle, Documentary, and more
- **Local station detection** -- identifies call signs (KABC, WCBS, etc.) with network affiliate and market info
- **Stream health monitoring** -- automatically detects offline channels and recovers them
- **Live sports scores** -- real-time scores and game status from ESPN in the web dashboard
- 3-day XMLTV EPG guide with per-channel schedules
- Continuous MPEG-TS stream proxy (works like a real TV tuner for Jellyfin/ffmpeg)
- Web dashboard with channel detail pages, program guide, and built-in video player
- Sport events sorted by start time with team logos and pregame/postgame EPG entries
- API key protection for playlist and EPG endpoints
- Automatic hourly refresh with offline channel recovery every 5 minutes
- Multi-architecture Docker support (amd64, arm64)

## Quick Start

### Docker Compose (recommended)

Create a `docker-compose.yml`:

```yaml
services:
    iptv:
        image: rebelstream/iptv:latest
        container_name: iptv
        ports:
            - "8080:8080"
        environment:
            TZ: "America/New_York"
        volumes:
            - ./data:/app/data
        restart: unless-stopped
```

```bash
docker compose up -d
```

### Docker Run

```bash
docker run -d \
  --name iptv \
  -p 8080:8080 \
  -e TZ=America/New_York \
  -v ./data:/app/data \
  --restart unless-stopped \
  rebelstream/iptv:latest
```

## Setting Up an API Key

Playlist and EPG endpoints can optionally be protected with an API key.

1. Open the web dashboard at `http://<server-ip>:8080`
2. Go to **Settings**
3. Click **Generate Key**
4. Use the key in your client URLs: `http://<server-ip>:8080/playlist?key=YOUR_KEY`

Without a key, playlist and EPG are accessible to anyone on your network.

## Usage with IPTV Clients

### Jellyfin

1. Go to **Dashboard > Live TV > Tuner Devices > Add**
2. Select **M3U Tuner**
3. Enter the M3U URL: `http://<server-ip>:8080/playlist?key=YOUR_KEY`
4. The EPG guide URL is auto-discovered from the playlist header
5. Save and refresh guide data

**Recommended tuner settings:**
- Allow stream sharing: **enabled**
- Auto-loop live streams: **enabled**
- Read input at native frame rate: **enabled**

### Plex

1. Install the **IPTV** plugin or use **xTeVe** as middleware
2. M3U URL: `http://<server-ip>:8080/playlist?key=YOUR_KEY`
3. EPG URL: `http://<server-ip>:8080/epg?key=YOUR_KEY`

### General IPTV Clients

| File | URL |
|------|-----|
| M3U Playlist | `http://<server-ip>:8080/playlist?key=YOUR_KEY` |
| EPG Guide (XML) | `http://<server-ip>:8080/epg?key=YOUR_KEY` |
| EPG Guide (Gzip) | `http://<server-ip>:8080/epg.gz?key=YOUR_KEY` |

> **Note:** The `?key=` parameter is only required if an API key has been generated in the web dashboard under **Settings**.

## Web Dashboard

Access the dashboard at `http://<server-ip>:8080`.

### Pages

- **Dashboard** -- channel/event counts, online/offline stats, live sports with start times and scores, playlist copy buttons
- **Channels** -- searchable and filterable channel list with category badges and online/offline status. Click a channel for its detail page with video player and program guide.
- **Sports** -- live and upcoming events sorted by start time with team logos. Click an event for its detail page with live scoreboard and video player.
- **Guide** -- per-channel program schedule with current show highlighted, filtered to current and upcoming
- **Settings** -- API key management, server info, version update check, manual refresh

### Channel Detail

Click any channel to see:
- Channel logo, categories, and status
- Embedded video player (click to watch)
- Now playing info with up-next preview
- Full program guide

### Sport Event Detail

Click any sport event to see:
- Team logos with live scores from ESPN (updates every 30 seconds)
- Period-by-period linescore
- Game status, venue
- Embedded video player

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `8080` | Server port |
| `HOST` | `0.0.0.0` | Bind address |
| `STREAM_QUALITY` | `hd` | Stream quality (`hd` or `sd`) |
| `CRON_SCHEDULE` | `0 * * * *` | Data refresh schedule (cron) |
| `SPORTS_EVENTS` | `true` | Set to `false` to disable sports events |
| `CACHE_DIR` | `./data` | Directory for cached data |
| `TZ` | `Etc/UTC` | Timezone |

### Cron Schedule Examples

| Schedule | Meaning |
|----------|---------|
| `0 * * * *` | Every hour (default) |
| `*/30 * * * *` | Every 30 minutes |
| `0 */3 * * *` | Every 3 hours |
| `0 0 * * *` | Once daily at midnight |

## How It Works

### Channels

On startup and every hour, the server scrapes channel data, normalizes names, detects categories, and checks stream health. Channels are sorted alphabetically and assigned persistent numbers. Offline channels are rechecked every 5 minutes.

Local broadcast stations are identified by their FCC call sign. For example, "ABC (KABC) Los Angeles" becomes **KABC (Los Angeles, CA)** with affiliate **ABC**.

Channels are grouped by category in the M3U playlist using `group-title`. Multiple categories are supported (e.g., a local station gets both "Local" and "News").

### Sports Events

Sport events are scraped, then cross-referenced with ESPN's scoreboard API for accurate start times and canonical team names. Events are sorted by start time across all leagues.

Each event gets EPG entries with pregame, game, and postgame blocks:

| Sport | Pregame | Game | Postgame |
|-------|---------|------|----------|
| NFL / NCAAF / CFL | 30 min | 3.5-4 hrs | 30 min |
| NHL | 30 min | 3 hrs | 30 min |
| NBA / NCAAB | 30 min | 2.5 hrs | 30 min |
| MLB | 30 min | 3.5 hrs | 30 min |
| Soccer / MLS | 15-30 min | 2 hrs | 15-30 min |
| UFC / Boxing / PPV | 30 min | 3 hrs | 30 min |

Events are removed from the lineup 30 minutes after the postgame block ends.

## Persistent Data

All persistent data is stored in the `/app/data` volume:

| File | Purpose |
|------|---------|
| `cache.json` | Channels, events, EPG data |
| `channel-map.json` | Master channel registry with names, categories, and sources |
| `instance.json` | Instance ID and API key |

## Reverse Proxy

The server auto-detects the correct base URL from `X-Forwarded-Proto` and `X-Forwarded-Host` headers.

### Nginx

```nginx
location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host $host;
}
```

### Traefik

```yaml
labels:
    - "traefik.enable=true"
    - "traefik.http.routers.iptv.rule=Host(`iptv.example.com`)"
    - "traefik.http.services.iptv.loadbalancer.server.port=8080"
```

## Reporting Issues

Found a bug or want a feature? [Open an issue](https://github.com/rebelstream/iptv/issues/new/choose) using one of the templates:

- **Bug Report** -- something isn't working
- **Feature Request** -- suggest an improvement
- **Channel Request** -- missing or broken channel

## License

Proprietary -- free for personal, non-commercial use. See [LICENSE](LICENSE) for details.
