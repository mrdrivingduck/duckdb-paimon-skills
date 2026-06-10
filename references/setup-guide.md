# Environment Setup Guide

## Platform Detection

| OS    | Arch    | Platform ID    |
|-------|---------|----------------|
| macOS | ARM64   | osx-arm64      |
| Linux | x86_64  | linux_amd64    |
| Linux | aarch64 | linux_arm64    |

Detect via:

```bash
uname -s   # Darwin or Linux
uname -m   # arm64, x86_64, aarch64
```

On macOS with Rosetta 2, `uname -m` may report `x86_64` on Apple Silicon. Verify with:

```bash
sysctl -n hw.optional.arm64   # returns 1 on Apple Silicon
```

## Step 1: Find Latest Extension Release

Scrape the GitHub releases page to find the latest duckdb-paimon release for the detected `{platform}`. This uses two lightweight HTML requests (no API, no rate-limit issues):

**Step 1a** — Get the latest release tag:

```bash
curl -sL https://github.com/polardb/duckdb-paimon/releases \
  | grep -oE 'releases/tag/[^"]+' \
  | head -1
```

This returns the most recent tag (e.g., `v0.0.6-variegata`).

**Step 1b** — Get assets for that tag:

```bash
curl -sL https://github.com/polardb/duckdb-paimon/releases/expanded_assets/{tag} \
  | grep -oE 'href="[^"]+"' \
  | grep '{platform}' \
  | grep '\.tar\.gz"'
```

The `expanded_assets` endpoint returns the asset list HTML fragment for a specific release, bypassing JavaScript rendering.

### Asset Naming Convention

Release tags follow: `{ext_version}-{codename}` (e.g., `v0.0.6-andium`).

Tarball names follow: `duckdb-paimon-{ext_version}-{codename}-v{duckdb_version}-{platform}.tar.gz`

For example, with `osx-arm64`, the matching assets might be:
- `duckdb-paimon-v0.0.6-variegata-v1.5.2-osx-arm64.tar.gz` (from tag `v0.0.6-variegata`)
- `duckdb-paimon-v0.0.6-andium-v1.4.4-osx-arm64.tar.gz` (from tag `v0.0.6-andium`)

From the matching assets, extract the available `{duckdb_version}` values (without the `v` prefix, e.g., `1.5.2`). These are the DuckDB versions supported by the extension. Prefer the highest version number.

If multiple extension releases exist, prefer the one with the latest release tag (highest `{ext_version}`).

### Fallback

If the releases page is unreachable, use `git ls-remote` to discover available release tags:

```bash
git ls-remote --tags https://github.com/polardb/duckdb-paimon.git
```

This returns tag names (e.g., `v0.0.6-variegata`) but not the full asset list. Construct candidate download URLs by combining the tag, known DuckDB versions, and platform, then probe with `curl -I` to verify they exist.

## Step 2: Check Local Installation

The default install location is `~/.duckdb-paimon/`. The tarball name from Step 1 (without the `.tar.gz` suffix) is the expected directory name. Check whether it already exists locally:

```bash
ls ~/.duckdb-paimon/{tarball_basename}/paimon.duckdb_extension
```

If the file exists, the latest version is already installed — note its absolute path and skip to extension loading (Phase 3 in AGENTS.md). Otherwise proceed to Step 3.

## Step 3: Download and Extract

The `{download_url}` from Step 1 is a relative path like `/polardb/duckdb-paimon/releases/download/...`. Prepend `https://github.com` to form the full URL.

### Download Acceleration

Detect whether a GitHub mirror proxy is needed by querying IP geolocation:

```bash
curl -s --connect-timeout 5 https://ifconfig.co/country-iso
```

If the result is `CN` (mainland China), GitHub downloads will be very slow. Prepend a mirror proxy to the download URL. Otherwise, download directly from GitHub.

| Priority | Proxy prefix |
|----------|-------------|
| 1 | `https://cors.isteed.cc/` |
| 2 | `https://gh-proxy.com/` |

Usage: prepend the proxy prefix directly before the full URL, e.g.:
- `https://cors.isteed.cc/https://github.com/polardb/duckdb-paimon/releases/download/...`

Try the first proxy; if it fails (`curl` returns non-zero), fall back to the next, then to direct download.

### Download

```bash
mkdir -p ~/.duckdb-paimon
curl -L -f -o /tmp/duckdb-paimon.tar.gz "{download_url}"
tar xzf /tmp/duckdb-paimon.tar.gz -C ~/.duckdb-paimon/
rm -f /tmp/duckdb-paimon.tar.gz
```

### Verify

After extraction, confirm the extension binary exists:

```bash
ls ~/.duckdb-paimon/*/paimon.duckdb_extension
```

The directory should also contain companion shared libraries (`libpaimon.dylib` on macOS, or `.so` on Linux). Do not move `paimon.duckdb_extension` out of its directory — it expects the companion libraries to be co-located.

## Step 4: Ensure DuckDB is Installed

`{duckdb_version}` is the preferred version determined in Step 1 (without the `v` prefix, e.g., `1.5.2`).

The official install script places the binary at `~/.duckdb/cli/{duckdb_version}/duckdb`, which is not on `PATH` by default. Always export the path first so that both an existing installation and a fresh one are discoverable:

```bash
export PATH="$HOME/.duckdb/cli/{duckdb_version}:$PATH"
```

Then check:

```bash
duckdb -version
```

- **DuckDB not found** — Install `{duckdb_version}`:

  ```bash
  curl https://install.duckdb.org | DUCKDB_VERSION={duckdb_version} sh
  ```

  See https://duckdb.org/install/ for more options. After installation, `duckdb -version` should work because `PATH` was already exported above.

- **Version matches** one of the supported versions from Step 1 — Done.
- **Version does not match** — Advise the user to install a supported version.
