# Linkding for TerraMaster TOS 7

TOS 7 App Center package for [Linkding](https://github.com/sissbruecker/linkding) — a self-hosted bookmark manager.

一个把 [Linkding](https://github.com/sissbruecker/linkding) 打包上架到 TOS 7 应用中心的 Docker 应用（单容器、单数据卷、SQLite，零外部依赖）。

---

## What it does / 这是什么

Linkding is a privacy-friendly bookmark manager that runs entirely inside a Docker container on your TNAS.

- Save links from any browser, tag and search them
- Full-text search plus automatically fetched page metadata (title, description, preview)
- Import / export bookmarks, REST API, browser extension support
- Single SQLite file — trivial to back up
- Web UI on host port `10002` → container port `9090`
- No external database, no root, no privileged mode

## Package contents / 包内文件

```
linkding.tar.gz
├── config.ini            # TOS application metadata
├── linkding.lang         # 14 languages (zh-cn, zh-hk, en-us, ... pt-pt)
├── linkding.svg          # application icon (SVG, transparent background)
└── docker-compose.yml    # container definition
```

## docker-compose.yml

```yaml
version: "3.8"

services:
  linkding:
    image: sissbruecker/linkding:1.47.0
    restart: unless-stopped
    user: "1000:1000"
    ports:
      - "10002:9090"
    volumes:
      - ./data:/etc/linkding/data
    environment:
      - TZ=Asia/Shanghai
      - LD_DISABLE_BACKGROUND_TASKS=True
      - LD_SUPERUSER_NAME=admin
```

`x-app-meta` declares `web.port: 10002` / `web.protocol: http`, as required for Docker applications with a Web UI.

## Design decisions

**Non-root execution.** The upstream image is built to run as root: `bootstrap.sh` performs
`chown -R www-data`, `uwsgi.ini` hardcodes `uid = www-data` / `gid = www-data`, and the default
background-task supervisor drops privileges as well. TOS requires the container to run as a
non-root user, so the container starts as `1000:1000` and the command performs the same bootstrap
steps as that user, starting uwsgi from a copy of `uwsgi.ini` with the privilege-dropping lines
removed. As a consequence background tasks (Wayback Machine snapshots, auto-tagging, link
previews) are disabled — see `LD_DISABLE_BACKGROUND_TASKS=True`. All other features, including
automatic metadata fetching for saved links, are available.

**Credentials.** No password is hardcoded anywhere. On first start a random administrator password
is generated inside the data volume at `data/.admin_password` (mode `600`, never written to logs).
The user name is `admin`. The path is documented in the `important` note of `linkding.lang`.

**Container name.** `container_name` is intentionally omitted, so Compose derives a globally
unique name (`<project>_<service>_1`). The platform only appends a numeric suffix to a conflicting
*project* name — it does not rewrite `container_name` — so hard-coding a container name would make
installation fail on a device that already runs a container called `linkding`.

## Data location / 数据位置

The host side of `./data` is resolved by the platform to the application data root:

```
/Volume<N>/DockerAppData/linkding/data
```

The database (`db.sqlite3`), the generated secret key and the initial password file all live there.
Data survives uninstall and reinstall as long as "delete data" is not selected.

## Build

```bash
./build.sh            # produces linkding.tar.gz + linkding.tar.gz.sha256
```

The archive must contain the four files at its root:

```bash
tar -czf linkding.tar.gz config.ini linkding.lang linkding.svg docker-compose.yml
sha256sum linkding.tar.gz > linkding.tar.gz.sha256
```

## Release

The package is published as a Release asset (the TOS Developer Platform downloads the package from
the Release, not from the repository root):

- Tag: `1.0.0`
- Asset: `linkding.tar.gz`
- Checksum: `linkding.tar.gz.sha256`

## Compliance notes / 合规说明

- Image source: Docker Hub only (`sissbruecker/linkding`), version tag pinned, never `:latest`.
- Runs as a non-root user (`user: "1000:1000"`); no `privileged`, no `network_mode: host`.
- No hardcoded credentials, tokens or secrets in the image or the compose file.
- No secrets are written to container logs.
- The icon in this repository is original artwork created for this package; it is **not** the
  upstream project logo, to avoid trademark issues.
- This repository contains packaging metadata only. The application itself is developed by the
  Linkding project and distributed under its own license (MIT). All credit belongs to the upstream
  authors.
- Ports 22, 80, 443, 445, 3306, 5050, 5432, 6379, 8181 and 8443 are not used.

## Links

- Upstream project: https://github.com/sissbruecker/linkding
- Docker Hub image: https://hub.docker.com/r/sissbruecker/linkding
- TOS 7 development guide: https://github.com/terramaster-tos/tos-app-pkg-tools
