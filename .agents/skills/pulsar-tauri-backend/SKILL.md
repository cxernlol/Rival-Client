---
name: rival-tauri-backend
description: >
  Skill for working on the Rival Client Rust/Tauri v2 backend.
  Use when implementing or modifying anything inside src-tauri/src/:
  IPC commands, Tokio async tasks, app lifecycle, window management,
  auth, downloader, game spawner, or config persistence.
---

# Rival Client — Tauri v2 Backend Skill

## Non-Negotiable Rules (from AGENTS.md)
- All disk, network, and process operations MUST be `async` via Tokio.
- Never block the Tokio runtime with synchronous I/O or `std::thread::sleep`.
- Never log OAuth tokens, `mc_token`, `access_token`, or `refresh_token`.
- JVM processes MUST be spawned **detached** so the launcher can minimize/exit freely.
- Idle RAM target: **< 30 MB**. Do not load large data structures at startup.

---

## Crate Versions (src-tauri/Cargo.toml)
```toml
[dependencies]
tauri          = { version = "2", features = [] }
tauri-build    = { version = "2", features = [] }      # build-dependency
tokio          = { version = "1", features = ["full"] }
serde          = { version = "1", features = ["derive"] }
serde_json     = "1"
reqwest        = { version = "0.12", default-features = false, features = ["json", "rustls-tls"] }
keyring        = "2"
open           = "5"
uuid           = { version = "1", features = ["v4"] }
base64         = "0.22"
sha2           = "0.10"
```

> **⚠ Tauri v2 breaking changes from v1:**
> - `#[tauri::command]` is unchanged, but builder API changed.
> - `tauri::Manager` import path changed.
> - Window API: use `tauri::WebviewWindow` not `tauri::Window`.
> - Always verify IPC call syntax against existing code before adding new commands.

---

## Directory Structure
```
src-tauri/src/
├── main.rs              # Entry point, register all IPC commands, setup windows
├── auth/
│   ├── mod.rs           # pub use oauth, token
│   ├── oauth.rs         # PKCE flow: generate verifier/challenge, loopback server, token exchange
│   └── token.rs         # Load/save/refresh tokens via keyring (never log values)
├── downloader/
│   ├── mod.rs
│   ├── manifest.rs      # Fetch Mojang version manifest JSON
│   ├── assets.rs        # Download game assets (parallel Tokio tasks with progress events)
│   └── fabric.rs        # Download & install Fabric Loader JAR
├── game/
│   ├── mod.rs
│   ├── jvm.rs           # Locate java binary, build G1GC JVM args from RAM setting
│   └── process.rs       # Spawn detached game process, emit `game_started` event
└── config/
    ├── mod.rs
    └── settings.rs      # Read/write ~/.Rival/settings.json (RAM MB, JVM flags, last profile)
```

---

## IPC Command Pattern
```rust
// src-tauri/src/main.rs
use tauri::Manager;

#[tauri::command]
async fn launch_game(
    app: tauri::AppHandle,
    profile: String,
) -> Result<(), String> {
    game::process::spawn_detached(&app, &profile)
        .await
        .map_err(|e| e.to_string())
}

fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![
            launch_game,
            // auth_start, auth_status, get_settings, set_settings, ...
        ])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

Frontend invokes with:
```ts
import { invoke } from '@tauri-apps/api/core';
await invoke('launch_game', { profile: '1.21.4-fabric' });
```

---

## Startup Window Sequence
1. `main.rs` → create **splash window** at 340×180px, frameless, always-on-top.
2. Async check: `auth::token::load_token()` (reads from OS keyring).
3. If valid token → resize to 840×520px, navigate to `/dashboard`.
4. If missing/expired → navigate to `/auth` (Microsoft login view).

Resize must use logical pixels:
```rust
window.set_size(tauri::Size::Logical(tauri::LogicalSize { width: 840.0, height: 520.0 }))?;
```

---

## G1GC JVM Args Pattern
```rust
pub fn build_jvm_args(ram_mb: u32) -> Vec<String> {
    vec![
        format!("-Xms{}M", ram_mb / 2),
        format!("-Xmx{}M", ram_mb),
        "-XX:+UseG1GC".into(),
        "-XX:+ParallelRefProcEnabled".into(),
        "-XX:MaxGCPauseMillis=200".into(),
        "-XX:+UnlockExperimentalVMOptions".into(),
        "-XX:+DisableExplicitGC".into(),
        "-XX:+AlwaysPreTouch".into(),
        "-XX:G1NewSizePercent=30".into(),
        "-XX:G1MaxNewSizePercent=40".into(),
        "-XX:G1HeapRegionSize=8M".into(),
        "-XX:G1ReservePercent=20".into(),
        "-XX:G1HeapWastePercent=5".into(),
        "-XX:G1MixedGCCountTarget=4".into(),
        "-XX:InitiatingHeapOccupancyPercent=15".into(),
        "-XX:G1MixedGCLiveThresholdPercent=90".into(),
        "-XX:G1RSetUpdatingPauseTimePercent=5".into(),
        "-XX:SurvivorRatio=32".into(),
        "-XX:+PerfDisableSharedMem".into(),
        "-XX:MaxTenuringThreshold=1".into(),
    ]
}
```

---

## Lint & Check Commands
```bash
cd src-tauri
cargo check          # Fast compile check
cargo clippy         # Fix ALL warnings before committing
cargo test           # Run test suite
```
