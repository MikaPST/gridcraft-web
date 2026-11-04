<p align="center">
  <img src="https://raw.githubusercontent.com/storytold/gridcraft/main/assets/app-icon/hicolor/256x256/apps/ai.storyteller.gridcraft.png" width="128" height="128" alt="GridCraft icon">
</p>

<h1 align="center">GridCraft Web — Docker Image</h1>

<p align="center">
  A self-hosted browser build of GridCraft, an open-source spreadsheet application written in Rust.
  <br>
  Packaged as a static web application and published to GitHub Container Registry.
</p>

<p align="center">
  <a href="https://github.com/MikaPST/gridcraft-web/actions/workflows/publish-image.yml"><img src="https://github.com/MikaPST/gridcraft-web/actions/workflows/publish-image.yml/badge.svg" alt="Build and publish image"></a>
  <a href="https://ghcr.io/mikapst/gridcraft-web"><img src="https://img.shields.io/badge/GHCR-latest-blue?logo=docker" alt="GHCR"></a>
  <img src="https://img.shields.io/badge/Docker%20Compose-supported-blue?logo=docker" alt="Docker Compose">
</p>

---

## Why this repository?

This repository provides a Docker image for the official [GridCraft](https://github.com/storytold/gridcraft) web application.

The goal is to make GridCraft easy to deploy on a self-hosted server such as Unraid, without having to build the application manually.

- Build the web application directly from the official upstream repository.
- Automatically publish the resulting Docker image to GitHub Container Registry (GHCR).
- Rebuild periodically to follow upstream changes.
- Provide a simple Docker Compose configuration.
- Make the image suitable for Unraid and other Docker environments.
- Keep the Docker packaging separate from the upstream application.

This repository is a packaging project only. It is not a fork of GridCraft and does not modify the upstream source code.

## How it works

GridCraft is an open-source spreadsheet application written in Rust. Its web version runs in a modern browser through WebAssembly and uses the same application interface as the desktop version.

GridCraft is designed for spreadsheet workflows, with XLSX as its native format, alongside CSV and TSV support. It includes formulas, formatting, tables, charts, sorting, filtering and pivot tables. The project is in alpha, and compatibility and feature depth are still evolving.

The Docker image:

- Serves the prebuilt GridCraft web application as static files using NGINX.
- Does not run a desktop session or remote desktop.
- Does not add a server-side spreadsheet engine or processing backend.
- Does not provide server-side user accounts or centralized workbook storage.

Opening a workbook uses the browser's file picker (drag and drop is also supported). Saving a workbook triggers a browser download. Browser storage may be used by the application for preferences or other local state, but it is not a Docker-hosted workbook volume.

## Docker image

The image is published on GitHub Container Registry:

```text
ghcr.io/mikapst/gridcraft-web:latest
```

### Option 1 — Docker Compose (recommended)

Create a `docker-compose.yml` file:

```yaml
services:
  gridcraft-web:
    image: ghcr.io/mikapst/gridcraft-web:latest
    container_name: gridcraft-web
    ports:
      - "8080:8080"
    restart: unless-stopped
```

Start the container:

```bash
docker compose up -d
```

Open GridCraft in your browser:

```text
http://YOUR_SERVER_IP:8080
```

#### Useful Compose commands

View container logs:

```bash
docker compose logs -f
```

Update the image:

```bash
docker compose pull
docker compose up -d
```

Stop and remove the container:

```bash
docker compose down
```

The container listens on port `8080`. No volume or environment variable is required by this static web image.

### Option 2 — Run directly with Docker

Pull the image:

```bash
docker pull ghcr.io/mikapst/gridcraft-web:latest
```

Run the container:

```bash
docker run -d \
  --name gridcraft-web \
  --restart unless-stopped \
  -p 8080:8080 \
  ghcr.io/mikapst/gridcraft-web:latest
```

Then open:

```text
http://YOUR_SERVER_IP:8080
```

### Network access and reverse proxies

The container exposes an HTTP service on port `8080`.

You can place it behind a reverse proxy such as Traefik or NGINX Proxy Manager to access it through a domain name and HTTPS.

For public access, use HTTPS through a reverse proxy rather than exposing the unencrypted HTTP endpoint directly to the Internet.

The server must serve `.wasm` files with the `application/wasm` MIME type. The included NGINX configuration sets this through the standard NGINX MIME types.

If GridCraft is embedded in a sandboxed iframe, the hosting page must allow scripts, same-origin access and downloads; otherwise browser storage or workbook exports may be blocked.

## Automated builds

The GitHub Actions workflow builds the web application from the official `storytold/gridcraft` repository and publishes the resulting image to GHCR.

| Trigger | Behaviour |
|---|---|
| Weekly schedule | Checks for upstream changes and rebuilds the image |
| Manual execution | Allows a build to be started manually |
| Packaging changes | Rebuilds when the workflow, README or Compose file changes |

Published tags:

- `latest` — the most recent successful build.
- `upstream-<commit>` — the upstream commit used for that build.

Supported image architectures:

- `linux/amd64`
- `linux/arm64`

> **Important:** GridCraft is currently in alpha. The `latest` tag is mutable and may contain upstream changes or regressions. For a stable deployment, consider pinning a specific image digest.

The workflow has been prepared but must be validated by its first execution in GitHub Actions. Until that run succeeds, the build and publication process should be considered unverified.

## Unraid Community Applications

This image can be used with Unraid through Docker Compose or a Community Applications template.

The Unraid template is maintained separately in the [MikaPST/unraid-community-apps](https://github.com/MikaPST/unraid-community-apps) repository.

This repository contains the Docker image build and publication workflow; the Community Applications repository contains the Unraid-specific template.

## Upstream project

**GridCraft** is an open-source spreadsheet application written in Rust, designed as a modern alternative for spreadsheet workflows.

Its capabilities include:

- XLSX as the native workbook format, with CSV and TSV support.
- Formula calculation with a large worksheet function library.
- Cell formatting, conditional formatting, tables and data validation.
- Charts, sparklines, pictures, links and comments.
- Sorting, filtering, outlining, subtotals and pivot tables.
- Browser-based workbook opening and download-based saving.

The upstream project also provides native desktop applications, a command-line interface and MCP/control-channel integrations. These are not included in this Docker image, which packages only the browser-based WebAssembly application.

- **Source code:** [github.com/storytold/gridcraft](https://github.com/storytold/gridcraft)
- **Official website:** [getartcraft.com/apps/gridcraft](https://getartcraft.com/apps/gridcraft)
- **Releases:** [GitHub Releases](https://github.com/storytold/gridcraft/releases)
- **Web hosting guide:** [Hosting GridCraft for the web](https://github.com/storytold/gridcraft/blob/main/packaging/web/README.md)

This repository packages the upstream web application for Docker. It does not modify the upstream application.

## License and attribution

GridCraft is dual-licensed under **MIT** or **Apache-2.0**, at your option. Copyright and required notices are maintained by the upstream project in its `NOTICE` file.

Bundled assets, if any, may have their own licenses. Their authors, sources and licenses are documented in the upstream `ATTRIBUTION.md`.

The GridCraft and ArtCraft names, logos and related brand assets are trademarks of the upstream project and are not covered by the software license. Refer to the upstream brand guidelines before reusing them.

The Docker packaging files in this repository are provided separately from the upstream application. Refer to the upstream repository for the complete license texts, notices and asset attributions.

## Disclaimer

This is an independent community packaging project and is not affiliated with, endorsed by, or supported by the ArtCraft team.

All product names, trademarks and logos are the property of their respective owners.
