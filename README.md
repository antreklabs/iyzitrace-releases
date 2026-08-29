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
| `iyzitrace-bundle.tar.gz` (+ `.sha256`) | Platform bundle (config, compose templates) |
| `iyzitrace-<version>-<os>-<arch>` (+ `.sha256`) | The `iyzitrace` installer CLI |

## Installing the platform

1. Download the CLI binary for your OS/architecture from the latest release and
   put it on your `PATH`:

   ```bash
   chmod +x iyzitrace-<version>-linux-amd64
   sudo mv iyzitrace-<version>-linux-amd64 /usr/local/bin/iyzitrace
   ```

2. Fetch the bundle and prepare the install directory (this downloads
   `iyzitrace-bundle.tar.gz` from the latest release automatically):

   ```bash
   iyzitrace init --require-checksum
   ```

3. Review `iyzitrace.yaml`, then bring the platform up:

   ```bash
   iyzitrace apply
   ```

4. Open the console (`http://<host>/console`) and install your license token
   under **License**.

To pin an older suite instead of the latest, pass that release's bundle asset
link: `iyzitrace init --bundle-url <asset-url> --require-checksum`.

## Installing the plugin

**From the Grafana catalog (recommended):** search for *IyziTrace* in
Grafana's plugin catalog and install the same version as your platform suite.

**Manual install from this page:**

1. Download `antreklabs-iyzitrace-app-<version>.zip` and unzip it into
   Grafana's plugin directory (`/var/lib/grafana/plugins` by default).
2. The plugin is currently unsigned, so allow it in `grafana.ini`:

   ```ini
   [plugins]
   allow_loading_unsigned_plugins = antreklabs-iyzitrace-app
   ```

3. Restart Grafana, enable the IyziTrace app, and follow its setup wizard to
   connect your platform.

## Updating

Update the **platform first, then the plugin** (a newer platform serves older
plugins fine; a newer plugin against an older platform disables its newest
screens until the platform catches up). The plugin shows an update banner when
a newer suite is published here.

## Support

- Product & documentation: <https://iyzitrace.com>
- Book a demo: <https://iyzitrace.com/book-demo>
