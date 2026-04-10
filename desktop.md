# Desktop App

The OpenFang Desktop App is a native desktop wrapper built with [Tauri 2.0](https://v2.tauri.app/) that packages the entire OpenFang Agent OS into a single, installable application. Instead of running a CLI daemon and opening a browser, users get a native window with system tray integration, OS notifications, and single-instance enforcement -- all powered by the same kernel and API server that the headless deployment uses.

**Crate:** `openfang-desktop`
**Identifier:** `ai.openfang.desktop`

---

## Architecture

The desktop app follows an embedded-server pattern:

```
+-------------------------------------------+
|  Tauri 2.0 Process                        |
|                                           |
|  +-----------+    +--------------------+  |
|  |  Main     |    | Background Thread  |  |
|  |  Thread   |    | ("openfang-server")|  |
|  |           |    |                    |  |
|  | WebView   |    | tokio runtime      |  |
|  | Window    |--->| axum API server    |  |
|  | (main)    |    | channel bridges    |  |
|  |           |    | background agents  |  |
|  | System    |    |                    |  |
|  | Tray      |    | OpenFang Kernel    |  |
|  +-----------+    +--------------------+  |
|       |                    |              |
|       |   http://127.0.0.1:{port}        |
|       +------------------------------------
+-------------------------------------------+
```

### Startup Sequence

1. **Tracing init** -- `tracing_subscriber` is configured with `RUST_LOG` env, defaulting to `openfang=info,tauri=info`.
2. **Kernel boot** -- `OpenFangKernel::boot(None)` loads the default configuration from `config.toml`, wrapped in `Arc`.
3. **Port binding** -- A `std::net::TcpListener` binds to `127.0.0.1:0` on the main thread, letting the OS assign a random free port. The port number is known before any window is created.
4. **Server thread** -- A dedicated OS thread named `"openfang-server"` is spawned. It creates its own `tokio::runtime::Builder::new_multi_thread()` runtime and runs the axum router via `openfang_api::server::build_router()` with graceful shutdown.
5. **Tauri app** -- The Tauri builder is assembled with plugins, managed state, IPC commands, system tray, and a WebView window pointing at `http://127.0.0.1:{port}`.
6. **Event loop** -- Tauri runs its native event loop. On exit, `server_handle.shutdown()` stops the embedded server and kernel.

### Graceful Shutdown

The axum server uses `with_graceful_shutdown()` wired to a `tokio::sync::watch` channel. When the Tauri app exits, the shutdown signal is sent, the background thread is joined, and the kernel is stopped. Channel bridges (Telegram, Slack, etc.) are stopped via `bridge.stop().await`.

---

## System Tray

The system tray provides quick access without bringing up the main window:

| Menu Item | Behavior |
|-----------|----------|
| **Show Window** | Shows, unminimizes, and focuses the main WebView window |
| **Open in Browser** | Opens `http://127.0.0.1:{port}` in the default browser |
| **Agents: N running** | Disabled info item showing current agent count |
| **Status: Running (uptime)** | Disabled info item showing uptime in human-readable format |
| **Launch at Login** | Checkbox toggling OS-level auto-start via `tauri-plugin-autostart` |
| **Check for Updates...** | Checks, downloads, installs, and restarts if an update is available |
| **Open Config Directory** | Opens `~/.openfang/` in the OS file manager |
| **Quit OpenFang** | Logs the quit event and exits the app |

The tray tooltip reads **"OpenFang Agent OS"**.

**Left-click on the tray icon** shows the main window (same as "Show Window").

---

## Single-Instance Enforcement

On desktop platforms, `tauri-plugin-single-instance` prevents multiple copies of OpenFang from running simultaneously. When a second instance attempts to launch, the existing instance's main window is shown, unminimized, and focused.

---

## Hide-to-Tray on Close

Closing the window does not quit the application. Instead, the window is hidden and the close event is suppressed. To actually quit, use the **"Quit OpenFang"** option in the system tray menu.

---

## Native OS Notifications

The app subscribes to the kernel's event bus and forwards critical events as native desktop notifications using `tauri-plugin-notification`:

| Event | Notification Title | Body |
|-------|-------------------|------|
| `LifecycleEvent::Crashed` | "Agent Crashed" | `Agent {id} crashed: {error}` |
| `LifecycleEvent::Spawned` | "Agent Started" | `Agent "{name}" is now running` |
| `SystemEvent::HealthCheckFailed` | "Health Check Failed" | `Agent {id} unresponsive for {secs}s` |

All other events are silently skipped.

---

## IPC Commands

Eleven Tauri IPC commands are registered, callable from the WebView frontend via `invoke()`:

| Command | Returns | Description |
|---------|---------|-------------|
| `get_port` | `u16` | Port the embedded server is listening on |
| `get_status` | JSON object | Status, port, agent count, uptime in seconds |
| `get_agent_count` | `usize` | Number of registered agents |
| `import_agent_toml` | agent name | Opens a native file picker for `.toml` files, validates, copies to `~/.openfang/agents/`, and spawns the agent |
| `import_skill_file` | -- | Opens a native file picker for skill files (`.md`, `.toml`, `.py`, `.js`, `.wasm`), copies to `~/.openfang/skills/`, triggers hot-reload |
| `get_autostart` | `bool` | Whether OpenFang launches at OS login |
| `set_autostart` | -- | Toggle launch at login |
| `check_for_updates` | `UpdateInfo` | Check for available updates without installing |
| `install_update` | -- | Download and install update, then restart (does not return on success) |
| `open_config_dir` | -- | Opens `~/.openfang/` in the OS file manager |
| `open_logs_dir` | -- | Opens `~/.openfang/logs/` in the OS file manager |

---

## Window Configuration

The main window is created programmatically in the Tauri `setup` closure:

| Property | Value |
|----------|-------|
| Window label | `"main"` |
| Title | `"OpenFang"` |
| URL | `http://127.0.0.1:{port}` (external) |
| Inner size | 1280 x 800 |
| Minimum inner size | 800 x 600 |
| Position | Centered |

The window uses `WebviewUrl::External(...)` rather than a bundled frontend, because the WebView renders the axum-served UI.

---

## CSP (Content Security Policy)

The `tauri.conf.json` configures a Content Security Policy that allows connections to the local embedded server, Google Fonts, and media/blob sources:

```
default-src 'self' http://127.0.0.1:* ws://127.0.0.1:*
         https://fonts.googleapis.com https://fonts.gstatic.com;
img-src 'self' data: blob: http://127.0.0.1:*;
style-src 'self' 'unsafe-inline'
         https://fonts.googleapis.com https://fonts.gstatic.com;
script-src 'self' 'unsafe-inline' 'unsafe-eval';
font-src 'self' https://fonts.gstatic.com;
connect-src 'self' http://127.0.0.1:* ws://127.0.0.1:*;
media-src 'self' blob: http://127.0.0.1:*;
frame-src 'self' blob: http://127.0.0.1:*;
object-src 'none';
base-uri 'self';
form-action 'self'
```

This permits the WebView to load content from the localhost API server and Google Fonts while blocking other external resource loading.

---

## Auto-Updater

The app checks for updates 10 seconds after startup. If an update is available, it is downloaded, installed, and the app restarts automatically. Users can also trigger a manual check via the system tray.

**Flow:**
1. Startup check (10s delay) -- if available -- notify user -- download and install -- app restarts
2. Tray "Check for Updates" -- same flow, with failure notification if install fails

**Configuration** (in `tauri.conf.json`):
- `plugins.updater.pubkey` -- Ed25519 public key (must match the signing private key)
- `plugins.updater.endpoints` -- URL to `latest.json` (hosted on GitHub Releases)
- `plugins.updater.windows.installMode` -- `"passive"` (install without full UI)

Every release bundle is signed with `TAURI_SIGNING_PRIVATE_KEY` (GitHub Secret). The `tauri-action` generates `latest.json` containing download URLs and signatures for each platform.

---

## Plugins

| Plugin | Purpose |
|--------|---------|
| `tauri-plugin-notification` | Native OS notifications for kernel events and update progress |
| `tauri-plugin-shell` | Shell/process access from the WebView |
| `tauri-plugin-dialog` | Native file picker for agent/skill import |
| `tauri-plugin-single-instance` | Prevents multiple instances (desktop only) |
| `tauri-plugin-autostart` | Launch at OS login (desktop only) |
| `tauri-plugin-updater` | Signed auto-updates from GitHub Releases (desktop only) |
| `tauri-plugin-global-shortcut` | Ctrl+Shift+O/N/C shortcuts (desktop only) |

### Capabilities

The default capability set grants permissions only to the `"main"` window:

```json
{
  "identifier": "default",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "notification:default",
    "shell:default",
    "dialog:default",
    "global-shortcut:allow-register",
    "global-shortcut:allow-unregister",
    "global-shortcut:allow-is-registered",
    "autostart:default",
    "updater:default"
  ]
}
```

---

## Building

### Prerequisites

- **Rust** (stable toolchain)
- **Tauri CLI v2**: `cargo install tauri-cli --version "^2"`
- **Platform-specific dependencies**:
  - **Windows**: WebView2 (included in Windows 10/11), Visual Studio Build Tools
  - **macOS**: Xcode Command Line Tools
  - **Linux**: `libwebkit2gtk-4.1-dev`, `libappindicator3-dev`, `librsvg2-dev`, `libssl-dev`, `build-essential`

### Development

```bash
cd crates/openfang-desktop
cargo tauri dev
```

This launches the app with hot-reload support. The console window is visible in debug builds for tracing output.

### Production Build

```bash
cd crates/openfang-desktop
cargo tauri build
```

This produces platform-specific installers:

- **Windows**: `.msi` and `.exe` (NSIS) installers
- **macOS**: `.dmg` and `.app` bundle
- **Linux**: `.deb`, `.rpm`, and `.AppImage`

The release binary suppresses the console window on Windows.

---

## Mobile Ready

The codebase includes conditional compilation guards for mobile platform support:

- The `run()` function is annotated with `#[cfg_attr(mobile, tauri::mobile_entry_point)]`.
- Desktop-only features (system tray, single-instance, hide-to-tray) are gated behind `#[cfg(desktop)]` so they compile out on mobile targets.
- iOS and Android builds are structurally supported by the Tauri 2.0 framework.

---

## File Structure

```
crates/openfang-desktop/
  build.rs                 # tauri_build::build()
  Cargo.toml               # Crate dependencies and metadata
  tauri.conf.json           # Tauri app configuration
  capabilities/
    default.json            # Permission grants for the main window
  gen/
    schemas/                # Auto-generated Tauri schemas
  icons/
    icon.png                # Source icon (1024x1024)
    icon.ico                # Windows icon
    32x32.png               # Small icon
    128x128.png             # Standard icon
    128x128@2x.png          # HiDPI icon
  src/
    main.rs                 # Binary entry point (calls lib::run())
    lib.rs                  # Tauri app builder, state types, event listener
    commands.rs             # IPC command handlers
    server.rs               # ServerHandle, kernel boot, embedded axum server
    tray.rs                 # System tray menu and event handlers
```

---

## Environment Variables

| Variable | Effect |
|----------|--------|
| `RUST_LOG` | Controls tracing verbosity. Defaults to `openfang=info,tauri=info` if unset. |

All other OpenFang environment variables (API keys, configuration) apply as normal since the desktop app boots the same kernel as the headless daemon.
