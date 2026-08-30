# IyziTrace Releases

Official release downloads for [IyziTrace](https://iyzitrace.com) — the
observability platform and its Grafana app plugin.

Each release on this repo is a **compatibility suite**: one page carrying a
plugin build and the platform build it was released with. Install both sides
from the same release page and they are guaranteed to work together — the
release list is the compatibility matrix.

| Asset | What it is |
|---|---|
| `antreklabs-iyzitrace-app-<version>.zip` (+ `.sha1`) | The Grafana app plugin |
| `iyzitrace-bundle.tar.gz` (+ `.sha256`) | Platform bundle (compose file, config templates) — always the latest suite |
| `iyzitrace-bundle-<version>.tar.gz` (+ `.sha256`) | The same bundle, version-pinned |
| `iyzitrace-<version>-<os>-<arch>` (+ `.sha256`) | The `iyzitrace` installer CLI |

CLI binaries are published for `linux` and `darwin`, `amd64` and `arm64`.

---

## Installing the platform (CLI)

The platform is installed and operated with the **`iyzitrace` CLI**. It fetches
the bundle from this repo, generates secrets, renders configuration, and drives
Docker Compose for you — there is no manual `git clone` or compose editing.

### Requirements

- Linux or macOS host (Linux for production)
- **Docker Engine + Compose v2** (`docker compose version` must work)
- Ports `80` and `443` free, or different ones set in `iyzitrace.yaml`
- ~4 GB RAM to start; sized by retention and ingest volume

### 1. Install the CLI

```bash
OS=$(uname -s | tr '[:upper:]' '[:lower:]')
ARCH=$(uname -m); case "$ARCH" in x86_64) ARCH=amd64 ;; aarch64) ARCH=arm64 ;; esac

URL=$(curl -fsSL https://api.github.com/repos/antreklabs/iyzitrace-releases/releases/latest \
  | grep -o "https://[^\"]*iyzitrace-[0-9.]*-$OS-$ARCH" | head -1)

curl -fsSL -o iyzitrace "$URL"
curl -fsSL -o iyzitrace.sha256 "$URL.sha256"
sha256sum -c iyzitrace.sha256 2>/dev/null || shasum -a 256 -c iyzitrace.sha256

chmod +x iyzitrace
sudo mv iyzitrace /usr/local/bin/iyzitrace
iyzitrace version
```

Prefer clicking? Download the `iyzitrace-<version>-<os>-<arch>` asset for your
machine from the latest release, `chmod +x` it, and move it onto your `PATH`.

### 2. Initialize the install

```bash
sudo iyzitrace init --require-checksum
```

This downloads `iyzitrace-bundle.tar.gz` from the **latest suite on this repo**,
verifies its `.sha256`, extracts it into the install directory, writes
`iyzitrace.yaml` from the shipped default, and generates a secrets vault.

Where things land depends on the user you run as:

| | Install dir (compose, config, `.env`) | Secrets vault |
|---|---|---|
| root | `/etc/iyzitrace` | `/var/lib/iyzitrace/secrets` |
| non-root | `~/.iyzitrace` | `~/.iyzitrace/secrets` |

Override either with `--install-dir <path>` (or `IYZITRACE_INSTALL_DIR`) on any
command. Telemetry data lives wherever `deployment.data_dir` in
`iyzitrace.yaml` points — `/var/lib/iyzitrace/data` out of the box.

### 3. Review the configuration

```bash
sudo iyzitrace config show      # print the effective config
sudo iyzitrace config edit      # open it in $EDITOR
sudo iyzitrace config validate  # check it before applying
```

The things worth setting before the first start:

```yaml
deployment:
  domain: observability.example.com   # localhost by default
  http_port: 80
  https_port: 443
  data_dir: /var/lib/iyzitrace/data   # put this on your big disk
```

Component images (Tempo, Loki, Prometheus, Thanos, OTel Collector, nginx and the
IyziTrace services) are pinned under `services:` — leave them alone unless you
are deliberately overriding one.

### 4. Apply and start

```bash
sudo iyzitrace apply   # render .env, .secrets.env and config/*.template, create data dirs
sudo iyzitrace up      # docker compose up -d
sudo iyzitrace status  # per-container state
```

`apply` never starts anything, `up` never rewrites configuration — after any
config change the order is always `apply` then `up` (or `restart`).

### 5. Install the license

Open the console at `http://<host>/console` and paste your token under
**License**, or do it from the CLI:

```bash
sudo iyzitrace license apply --from-file license.jwt
sudo iyzitrace license status
```

Point either command at a non-default address with `--url http://host:port`.

### Pinning a specific suite

`init` follows the latest release. To install an older, pinned suite, pass that
release's versioned bundle asset:

```bash
sudo iyzitrace init --require-checksum \
  --bundle-url https://github.com/antreklabs/iyzitrace-releases/releases/download/v1.0.16/iyzitrace-bundle-0.0.7.tar.gz
```

`IYZITRACE_BUNDLE_URL` works as an environment-variable equivalent, and a
`file://` URL installs from a bundle you downloaded ahead of time — useful for
air-gapped hosts.

### Upgrading

Upgrade the **platform first, then the plugin**. A newer platform serves older
plugins fine; a newer plugin against an older platform simply hides its newest
screens until the platform catches up.

```bash
sudo iyzitrace upgrade --require-checksum \
  --bundle-url https://github.com/antreklabs/iyzitrace-releases/releases/download/<tag>/iyzitrace-bundle-<version>.tar.gz
```

`upgrade` extracts the new bundle over the install dir, keeps your
`iyzitrace.yaml` and secrets, re-applies configuration, and restarts the stack.
Use `--dry-run` to see what would change, `--no-up` to stop after applying, and
`-y` to skip the confirmation prompt.

### Command reference

| Command | What it does |
|---|---|
| `iyzitrace init` | Fetch the bundle, write `iyzitrace.yaml`, generate secrets |
| `iyzitrace config show \| edit \| validate` | Inspect and change configuration |
| `iyzitrace apply` | Render `.env`, `.secrets.env`, config templates; prepare data dirs |
| `iyzitrace up [service...]` | Start the platform (or named services) |
| `iyzitrace down [--purge]` | Stop the platform |
| `iyzitrace restart [service...]` | Restart the platform (or named services) |
| `iyzitrace status` | Per-container state |
| `iyzitrace logs [service]` | Stream logs |
| `iyzitrace upgrade` | Move an install to a newer bundle |
| `iyzitrace license apply \| status` | Install or inspect the license |
| `iyzitrace secrets list \| show \| set \| rotate` | Manage the secrets vault |
| `iyzitrace fix-perms` | Repair data-directory ownership |
| `iyzitrace version` | CLI version and the installed bundle version |

---

## Installing the plugin

**From the Grafana catalog (recommended):** search for *IyziTrace* in Grafana's
plugin catalog and install the version matching your platform suite.

**Manual install from this page:**

1. Download `antreklabs-iyzitrace-app-<version>.zip` from the same release you
   installed the platform from and unzip it into Grafana's plugin directory
   (`/var/lib/grafana/plugins` by default):

   ```bash
   unzip antreklabs-iyzitrace-app-<version>.zip -d /var/lib/grafana/plugins
   ```

2. Restart Grafana, then enable the **IyziTrace** app under
   *Administration → Plugins* and follow its setup wizard to point it at your
   platform URL.

The plugin is signed, so no `allow_loading_unsigned_plugins` entry is needed.

---

## Support

- Product & documentation: <https://iyzitrace.com>
- Book a demo: <https://iyzitrace.com/book-demo>
