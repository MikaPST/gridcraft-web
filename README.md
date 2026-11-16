<p align="center">
  <img src="https://raw.githubusercontent.com/storytold/gridcraft/main/assets/app-icon/hicolor/256x256/apps/ai.storyteller.gridcraft.png" width="128" height="128" alt="GridCraft application icon">
</p>

<h1 align="center">GridCraft Web — Docker Image</h1>
<p align="center">
  A community-maintained Docker image for
  <a href="https://github.com/storytold/gridcraft">GridCraft</a>,
  bringing its browser-based spreadsheet editor to a self-hosted web interface.
  <br>
  Automated official-release builds and publication to GitHub Container Registry (GHCR).
</p>

<p align="center">
  <a href="https://github.com/MikaPST/gridcraft-web/actions/workflows/publish-image.yml"><img src="https://github.com/MikaPST/gridcraft-web/actions/workflows/publish-image.yml/badge.svg" alt="Build and publish GridCraft Web"></a>
  <a href="https://github.com/MikaPST/gridcraft-web/releases/latest"><img src="https://img.shields.io/github/v/release/MikaPST/gridcraft-web?label=Release&logo=github&color=blue" alt="Latest Release"></a>
  <a href="https://github.com/MikaPST/gridcraft-web/pkgs/container/gridcraft-web"><img src="https://ghcr-badge.egpl.dev/mikapst/gridcraft-web/latest_tag?trim=major&label=GHCR" alt="GHCR image tag"></a>
  <a href="docker-compose.yml"><img src="https://img.shields.io/badge/Docker%20Compose-ready-2496ED?logo=docker&logoColor=white" alt="Docker Compose configuration"></a>
</p>

---

## Why this repository?

The official GridCraft project provides a WebAssembly-based spreadsheet editor that can be hosted as a static website. This repository packages the browser application into a Docker image for convenient self-hosting.

The goals are to:

- Build the official GridCraft web application from the exact upstream release tag.
- Publish ready-to-run Docker images to GHCR for both AMD64 and ARM64.
- Check for new official releases daily and publish corresponding versioned images.
- Create a matching GitHub Release **only after** the Docker image has been published.
- Offer a simple Docker Compose configuration and support Unraid deployments.
- Maintain Docker, NGINX and workflow configuration independently of upstream.
- Preserve upstream license, attribution and third-party asset notices in the image.

This is a packaging and distribution project, **not** a fork intended to develop a separate spreadsheet editor.

## How it works

GridCraft is an open-source spreadsheet editor built in Rust. Its browser application uses WebAssembly and serves a familiar spreadsheet interface with formulas, formatting, tables, charts and workbook import/export.

- **Browser-based spreadsheets:** open XLSX, CSV and TSV files through the browser, with additional format support depending on the upstream release.
- **Local workbook handling:** opening uses the browser's file picker or drag-and-drop; saving and exporting trigger browser downloads.
- **Static application:** NGINX serves the compiled HTML, JavaScript, WebAssembly and assets. It does not provide a server-side spreadsheet engine or user accounts.
- **Self-hosted:** access GridCraft on your own server, over a local network or a properly configured HTTPS reverse proxy.

The upstream desktop CLI, MCP server and control channel are not deployed as services by this image. Browser integration features, where available, are part of the upstream web application.

## Docker image

The image is published to GitHub Container Registry:

```text
ghcr.io/mikapst/gridcraft-web:latest
```

Each official upstream release is also published under its original version tag, for example:

```text
ghcr.io/mikapst/gridcraft-web:v0.15.1
```

### Option 1 — Docker Compose (recommended)

A `docker-compose.yml` file is included at the repository root:

```yaml
services:
  gridcraft-web:
    image: ghcr.io/mikapst/gridcraft-web:latest
    container_name: gridcraft-web
    ports:
      - "8080:8080"
    restart: unless-stopped
```

Start the application:

```bash
docker compose up -d
```

Open `http://YOUR_SERVER_IP:8080` (or `http://localhost:8080` when Docker runs on your own computer).

#### Useful Compose commands

```bash
# Follow application logs
docker compose logs -f

# Update to the newest published image
docker compose pull
docker compose up -d

# Stop and remove the container
docker compose down
```

No `.env` file or persistent volume is required. To change the host port, modify the left side of the mapping, for example `8081:8080`.

### Option 2 — Run directly with Docker

```bash
docker pull ghcr.io/mikapst/gridcraft-web:latest

docker run -d \
  --name gridcraft-web \
  --restart unless-stopped \
  -p 8080:8080 \
  ghcr.io/mikapst/gridcraft-web:latest
```

Open `http://YOUR_SERVER_IP:8080`. Optional CPU and memory limits may be configured with Docker if required.

### Network access and reverse proxies

The container listens on HTTP port `8080`. Use a reverse proxy such as Traefik for HTTPS and avoid exposing unencrypted HTTP directly to the public Internet.

**WebGPU and clipboard access require a secure context**, normally HTTPS or localhost. On non-secure origins GridCraft can fall back to WebGL2. The app does not require cross-origin isolation headers such as COOP/COEP. Its iframe host bridge has additional same-origin and sandbox requirements; refer to upstream documentation before embedding it in another site.

## Repository structure

The repository versions its own Docker packaging and retrieves the official application source only during CI builds.

```text
gridcraft-web/
├── .github/
│   └── workflows/
│       └── publish-image.yml
├── .dockerignore
├── Dockerfile
├── nginx.conf
├── docker-compose.yml
└── README.md
```

| File | Purpose |
|---|---|
| `Dockerfile` | Defines the unprivileged NGINX runtime image, OCI metadata and health check. |
| `nginx.conf` | Serves WebAssembly, compressed assets, hashed-file caching, security headers and `/healthz`. |
| `.dockerignore` | Restricts the image build context to the generated site, legal notices and configuration. |
| `docker-compose.yml` | Defines the ready-to-use Compose deployment. |
| `.github/workflows/publish-image.yml` | Detects upstream releases, builds WebAssembly, publishes multi-platform GHCR images and creates GitHub Releases. |
| `README.md` | Documents deployment, architecture, automation and attribution. |

### Docker build architecture

GridCraft's official web build uses **Rust, WebAssembly and Trunk**. The upstream `apps/gridcraft-web/Trunk.toml` defines `../../dist/web` as its output directory and uses relative URLs.

The build process follows these steps:

1. GitHub Actions resolves the latest published official GridCraft release or a manually selected upstream tag.
2. It checks out that exact upstream source release into `upstream/`.
3. Rust, the `wasm32-unknown-unknown` target and Trunk are installed.
4. The official application is compiled from `upstream/apps/gridcraft-web` using `trunk build --release --dist ../../dist/web --public-url ./`.
5. The workflow validates `upstream/dist/web/index.html`, generated JavaScript and WebAssembly files, and checks for root-absolute URLs.
6. The static site, the version-controlled `Dockerfile`, `nginx.conf` and `.dockerignore`, plus available upstream legal notices, are assembled in `image-context/`.
7. Docker Buildx publishes the NGINX runtime image to GHCR for `linux/amd64` and `linux/arm64`.
8. A matching GitHub Release is created or updated **only after** the multi-platform image is successfully published.

The application is compiled once and packaged into both container architectures. No database, server-side workbook volume, GPU passthrough or remote desktop is required.

The runtime uses an unprivileged NGINX user, exposes `/healthz`, serves `.wasm` with its required MIME type, and revalidates `index.html`. Content-hashed assets can be cached immutably.

> [!NOTE]
> Upstream Rust, Trunk, dependencies or build-path changes may require packaging adjustments. Static checks do not replace a successful GitHub Actions build.

## Automated builds

GitHub Actions checks official GridCraft releases every day.

| Trigger | Behaviour |
|---|---|
| Daily schedule | Checks the latest official release and builds it if it has not already been published here |
| Manual workflow dispatch | Builds the latest release or the specified official upstream tag |
| Force rebuild | Rebuilds an existing version and updates its image and GitHub Release |
| Packaging changes on `main` | Rebuilds the latest official release when version-controlled packaging or documentation changes |

### Published tags

Each successful build publishes:

- `latest` — most recently packaged official GridCraft release.
- `<upstream-version>` — the exact official release tag, for example `v0.15.1`.
- `upstream-<commit>` — the first 12 characters of the upstream source commit.

Images are published for `linux/amd64` and `linux/arm64`.

### GitHub Releases

After the Docker image is successfully pushed to GHCR, the workflow creates or updates a [GitHub Release](https://github.com/MikaPST/gridcraft-web/releases) matching the upstream version.

Each release includes the official upstream release link and notes, source revision, Docker tags, supported architectures, image digest and build provenance.

The **Release** badge at the top of this README displays the latest GitHub Release. Scheduled runs skip already published versions. To rebuild, open **Actions → Build and publish GridCraft Web → Run workflow** and enable `force_rebuild`.

> [!IMPORTANT]
> A GitHub Release is created **only after** a successful GHCR image push. `latest` is mutable; forced rebuilds can also update versioned tags. Pin an image digest when exact reproducibility is required.

## Unraid Community Applications

GridCraft Web is suitable for Unraid Docker deployments. The Community Applications template is maintained separately from this repository:

- **Unraid Community Applications repository:** [MikaPST/unraid-community-apps](https://github.com/MikaPST/unraid-community-apps)
- **GridCraft Web template:** [gridcraft-web.xml](https://github.com/MikaPST/unraid-community-apps/blob/main/templates/gridcraft-web.xml)
- **Docker image:** `ghcr.io/mikapst/gridcraft-web:latest`
- **Container port:** `8080`

The Docker image does not store workbooks in Unraid `appdata`; workbook files are handled in the browser.

## Upstream project

GridCraft is an open-source, clean-room spreadsheet application developed by the ArtCraft team and its contributors.

- **Source code:** [storytold/gridcraft](https://github.com/storytold/gridcraft)
- **Official website:** [getartcraft.com/apps/gridcraft](https://getartcraft.com/apps/gridcraft)
- **Official web hosting guide:** [Hosting GridCraft for the web](https://github.com/storytold/gridcraft/blob/main/packaging/web/README.md)
- **Official releases:** [GridCraft releases](https://github.com/storytold/gridcraft/releases)

This repository publishes independent Docker packaging built from the official upstream release, not an official GridCraft distribution.

## License and attribution

GridCraft is dual-licensed under the **MIT License or Apache License 2.0**, at the user's option. Upstream copyright and attribution notices must be preserved.

- [Upstream MIT License](https://github.com/storytold/gridcraft/blob/main/LICENSE-MIT)
- [Upstream Apache License 2.0](https://github.com/storytold/gridcraft/blob/main/LICENSE-APACHE)
- [Upstream NOTICE](https://github.com/storytold/gridcraft/blob/main/NOTICE)
- [Upstream asset attributions](https://github.com/storytold/gridcraft/blob/main/ATTRIBUTION.md)

The workflow copies upstream license and attribution files, into the published runtime image. The upstream software licenses do not automatically determine the license of independently maintained packaging files; consult this repository's own license if present.

GridCraft, ArtCraft, their logos and trademarks remain the property of their respective owners. This project is community-maintained and is **not affiliated with or endorsed by** the official maintainers.

## Disclaimer

GridCraft is under active development. Features, browser compatibility, import/export fidelity and behavior may change between releases. Keep backups of important workbooks and test new versions before relying on them for critical work.
