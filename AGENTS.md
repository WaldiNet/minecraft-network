# AGENTS.md

## Project type

Docker Compose infrastructure for a Minecraft server network. No code, no build, no tests, no lint.

## Prerequisites

Only **Docker** is required. No language runtime.

## Setup and run

```bash
cp .env.dist .env   # required first step — .env is gitignored
docker compose up -d
docker compose down
```

The `.env.dist` is the template. The `.env` file must exist at runtime and is never committed.

## .env quirks

- `SERVER_CURSEFORGE_API_KEY` must be set if any `_CURSEFORGE_FILES` variables are used.
- Mod lists use `_MODRINTH_PROJECTS` (slug:version-id format) and `_CURSEFORGE_FILES` (project-id:file-id format), per the itzg docs.

## Architecture

- **1 proxy** (Velocity via [`itzg/bungeecord`](https://github.com/itzg/docker-mc-proxy)) + **3 backend servers** (Fabric via [`itzg/minecraft-server`](https://github.com/itzg/docker-minecraft-server)).
- **3 backup containers** ([`itzg/mc-backup`](https://github.com/itzg/docker-mc-backup)) — one per server instance.
- **1 code-server** for in-browser editing (exposes the whole repo at `/config/workspace`).

### Server instances

| Instance    | Container          | Game mode    | Memory |
|-------------|--------------------|--------------|--------|
| `waldicraft`  | `server-smp`       | survival     | 4G     |
| `expedition`  | `server-expedition`| survival     | 8G     |
| `testing`     | `server-testing`   | creative/flat| 2G     |

Each instance directory (`server/<name>/`) has three subdirs:
- `config/` — mounted read-only, committed to git
- `data/` — world data, mostly gitignored (only `ops.json`, `whitelist.json`, and `dynmap/` config are tracked)
- `mods/` — mod JARs, entirely gitignored (downloaded at container startup from Modrinth/CurseForge URLs in `.env`)

### Proxy/server forwarding

- Velocity uses `modern` forwarding mode with `forwarding.secret` (committed).
- Backend servers run `ONLINE_MODE=false` and use **FabricProxy-Lite** with hack settings for player info forwarding.
- Velocity `velocity.toml` maps server names to internal Docker hostnames:
  ```
  waldicraft = "server-smp:25565"
  expedition = "server-expedition:25565"
  testing    = "server-testing:25565"
  ```

## Adding or modifying mods

Mod JARs are **never committed**. They are downloaded at container startup. To change mods:

1. Edit the relevant `_MODRINTH_PROJECTS`, `_CURSEFORGE_FILES`, or `_DATAPACKS` variable in `.env.dist` (and update `.env`).
2. For mods not on Modrinth/CurseForge, place JARs manually in `server/<instance>/mods/`.
3. Mod config files go in `server/<instance>/config/` and **are** committed.

## Key files for configuration changes

- `.env.dist` — environment variables (ports, seeds, mods, memory, RCON passwords)
- `proxy/velocity/config/velocity.toml` — proxy routing and server list
- `server/<instance>/config/` — per-server mod configs
- `docker-compose.yaml` — service definitions

## Optional services

Four additional services exist in `docker-compose.yaml` but are **commented out**. To enable them, uncomment the service definition, verify paths/variables, and configure `.env` values. Only code-server is active by default.

### Nginx (`service-nginx`)

Serves the **LiveAtlas** web map UI plus dynmap tiles for all three servers. Required because all dynmap configs have `disable-webserver: true` and use `org.dynmap.JsonFileClientUpdateComponent` — the internal dynmap web server is off; data is written as JSON/tiles to the filesystem for an external web server to serve.

- Web root: `services/nginx/www/` — contains `index.html` (pre-configured LiveAtlas app) + `live-atlas/` (built JS/CSS assets)
- Mounts dynmap output from each server: `server/<name>/data/dynmap/web/standalone` and `…/tiles`
- LiveAtlas is pre-configured for all three servers in `index.html` using the filetree/JSON approach

### PHP-FPM (`service-php-fpm`)

Required by Nginx to handle PHP endpoints for LiveAtlas — chat sending, login, and register. The nginx config (`docker/nginx/conf.d/default.conf`) proxies `*.php` requests to `service-php-fpm:9000`. Must be enabled alongside Nginx.

### MySQL (`service-mysql`)

Alternative dynmap storage backend. The current dynmap configs use `type: filetree`; MySQL is the commented-out option (`type: mysql`) in each `configuration.txt`.

- Init script (`.docker/mysql/init/01-databases.sql`) creates a `dynmap` database and grants access to the `minecraft` user
- LiveAtlas `index.html` includes commented-out `MySQL_*.php` endpoint URLs for when MySQL is used
- Data volume `services/mysql/` is gitignored (see `services/.gitignore`)

### Redis (`service-redis`)

Available as a Redis 7.2 cache instance. No config in this repo references a specific Redis consumer — intended as infrastructure for a mod or plugin that needs it.

## CI

A single GitHub Actions workflow (`.github/workflows/opencode.yml`) triggers OpenCode AI on issue/PR comments containing `/oc` or `/opencode`.
