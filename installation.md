# Installation

Validated against OpenFang release `v0.5.7`.

This page only documents install paths that are supported by the released source tree or release assets. Unverified package-manager claims were removed.

## Requirements

| Requirement | Notes |
|-------------|-------|
| OS | Linux, macOS, or Windows |
| Architecture | `x86_64` and `aarch64` are handled by the release installers |
| LLM provider access | At least one configured provider, unless you are using a local backend such as Ollama |
| Rust 1.75+ | Only required for building from source |

## Recommended Installers

### Linux and macOS

```bash
curl -fsSL https://openfang.sh/install | sh
```

The released shell installer in `scripts/install.sh`:
- resolves the latest GitHub release unless `OPENFANG_VERSION` is set
- downloads the platform archive for the current OS and CPU
- verifies the SHA256 checksum when a checksum file is available
- installs to `~/.openfang/bin` by default
- updates the detected shell startup file when possible
- ad-hoc signs the binary on macOS after stripping quarantine attributes

Supported installer environment variables:

| Variable | Meaning |
|----------|---------|
| `OPENFANG_INSTALL_DIR` | Override the install directory |
| `OPENFANG_VERSION` | Install a specific tag such as `v0.5.7` |

Example:

```bash
OPENFANG_VERSION=v0.5.7 curl -fsSL https://openfang.sh/install | sh
```

### Windows

```powershell
irm https://openfang.sh/install.ps1 | iex
```

The released PowerShell installer in `scripts/install.ps1`:
- detects `x86_64` vs `aarch64`
- resolves the latest GitHub release unless `OPENFANG_VERSION` is set
- downloads the Windows archive
- verifies the SHA256 checksum when available
- installs `openfang.exe` into `%USERPROFILE%\.openfang\bin`
- appends that directory to the user `PATH`

## Release Assets

The upstream release page is the safest manual install source:

`https://github.com/RightNow-AI/openfang/releases/tag/v0.5.7`

The docs and source tree reference these platform asset types:

| Platform | Expected asset type |
|----------|---------------------|
| Windows | `.msi` and CLI archive |
| macOS | `.dmg` and CLI archive |
| Linux | `.AppImage`, `.deb`, and CLI archive |

## Build From Source

OpenFang is not documented here as a crates.io install. The released source tree supports Git-based Cargo builds.

### Build the CLI from Git

```bash
cargo install --git https://github.com/RightNow-AI/openfang openfang-cli
```

### Build from a local checkout

```bash
git clone https://github.com/RightNow-AI/openfang.git
cd openfang
cargo install --path crates/openfang-cli
```

Useful release-oriented build commands from the source tree:

```bash
# Faster optimized build
cargo build --profile release-fast -p openfang-cli

# Full release build
cargo build --release -p openfang-cli

# Test the workspace
cargo test --workspace
```

## Docker

The released source tree ships a `Dockerfile` and `docker-compose.yml`. The safest documented container path is to run from a checked-out release tree:

```bash
git clone https://github.com/RightNow-AI/openfang.git
cd openfang
git checkout v0.5.7
docker compose up -d
```

If you are using registry images, verify the tag and asset naming against the GitHub release page before pinning production deployments.

## systemd

The source tree includes `deploy/openfang.service`:

```bash
sudo cp deploy/openfang.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable openfang
sudo systemctl start openfang
```

## First Run

```bash
openfang init
openfang doctor
openfang start
```

Expected default dashboard URL:

```text
http://127.0.0.1:4200
```

## Notes

- No Homebrew workflow was kept in this docs repo because it was not validated from the `v0.5.7` release sources.
- No crates.io publication claim was kept for the same reason.
- Earlier docs in this repo used stale installer examples such as `OPENFANG_VERSION=v0.4.4`; those were removed.
