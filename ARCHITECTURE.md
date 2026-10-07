# ✦ Rival Client — Technical Specification & Project Architecture

**Rival Client** is an open-source, hyper-lightweight Minecraft launcher and optimization distribution. Built with **Rust**, **Tauri v2**, and **React**, it enforces a strict zero-bloat philosophy: maximum raw FPS, zero background telemetry, sub-second startup, and an OLED-focused desktop interface.

---

## 1. Project Manifesto & Guiding Principles

1. **Zero Bloat & Telemetry:** Absolutely zero launcher advertisements, daily store banners, promotional popups, or external tracking scripts. Network activity is limited strictly to Mojang, Fabric, and Modrinth APIs.
2. **Resource Discipline:** Target idle RAM footprint under **30 MB** (achieved via detached process launching and Webview suspension). Target startup time under **0.4 seconds**.
3. **100% Open & Auditable:** All code, build scripts, and workflows are public, verifiable, and free of proprietary lock-ins.
4. **Phased Focus (Simplicity First):**
   - **Phase 1 (Alpha MVP):** High-performance desktop launcher, local Microsoft OAuth, asset/Fabric downloader, and detached JVM spawner.
   - **Phase 2 (Game Enhancement):** Native Fabric Mixin menu and optional in-game lightweight HUD mods.

---

## 2. Technical Stack Overview

```text
┌────────────────────────────────────────────────────────┐
│             UI Layer: Vite + React 18                  │
│    TypeScript • Tailwind CSS • Shadcn UI • Zustand     │
└───────────────────────────┬────────────────────────────┘
                            │ Tauri IPC (Async)
┌───────────────────────────┴────────────────────────────┐
│          Backend Engine: Rust + Tauri v2               │
│   Tokio Runtime • Local PKCE Loopback • Process Spawner │
└───────────────────────────┬────────────────────────────┘
                            │ Spawns Detached
┌───────────────────────────┴────────────────────────────┐
│               Game Runtime: Java JVM                   │
│   Fabric Loader • Sodium • Lithium • FerriteCore       │
└────────────────────────────────────────────────────────┘
```

- **Frontend (`src/`):** React 18, TypeScript, Tailwind CSS, Shadcn UI primitives, Zustand state management, Lucide React icons, Framer Motion (subtle UI feedback).
- **Backend (`src-tauri/`):** Rust 2021 edition, Tauri v2, Tokio multi-threaded async runtime, `reqwest` (HTTP), `keyring` (secure credential vault).
- **Game Engine:** Java JVM (Java 17/21) running Fabric Loader with pre-configured optimization mods:
  - **Rendering:** Sodium, Iris (Shaders), NVIDIUM (NVIDIA Turing+ mesh shader accelerator; auto-detected).
  - **Memory & Logic:** FerriteCore, Lithium, ModernFix.

---

## 3. Directory Structure

```text
RivalClient/
├── .agents/                     # AI Agent skills & guidelines
│   └── skills/
│       ├── frontend-design/     # Distinctive visual design rules
│       ├── karpathy-guidelines/ # Behavioral coding discipline
│       ├── Rival-tauri-backend/# Rust / Tauri v2 IPC patterns
│       ├── Rival-react-frontend/# React components & Zustand stores
│       └── Rival-microsoft-auth/# OAuth 2.0 PKCE loopback specs
├── src-tauri/                   # Rust Backend (Tauri v2 Native Layer)
│   ├── Cargo.toml
│   ├── tauri.conf.json
│   └── src/
│       ├── main.rs              # Tauri entry point, window init & IPC handlers
│       ├── auth/                # Microsoft OAuth 2.0 PKCE loopback server & keyring
│       ├── downloader/          # Async asset, library & Fabric downloader (Tokio)
│       ├── game/                # JVM command builder, G1GC tuning & process management
│       └── config/              # Local settings persistence (~/.Rival/settings.json)
└── src/                         # React Frontend (Vite)
    ├── index.html
    ├── App.tsx
    ├── main.tsx
    ├── index.css                # OLED theme tokens & Space Grotesk / JetBrains Mono fonts
    ├── components/
    │   ├── splash/              # Fast startup state / skeleton
    │   ├── auth/                # Microsoft login view
    │   ├── dashboard/           # Hero launch view
    │   │   ├── AvatarCarousel.tsx# 3D player skin stage with shadow avatar depth & slide animation
    │   │   ├── Nametag.tsx      # Floating Minecraft nametag with live online status
    │   │   ├── PlayButton.tsx   # Glowing hero PLAY CTA
    │   │   └── VersionPicker.tsx# Attached profile/version selector
    │   ├── layout/              # Sleek top minimalist navbar & bottom status ribbon
    │   ├── library/             # Profile & version manager
    │   ├── mods/                # Modrinth browser & optimization stack toggles
    │   └── settings/            # RAM allocation slider & JVM flags
    ├── store/                   # Zustand stores (auth, game, settings)
    └── types/                   # Shared TypeScript definitions
```

---

## 4. UI / UX Design Blueprint

Adhering strictly to [`frontend-design`](.agents/skills/frontend-design/SKILL.md) and [`karpathy-guidelines`](.agents/skills/karpathy-guidelines/SKILL.md):

### A. Design System Tokens & Typography
- **Palette (High-Contrast OLED Dark):**
  - `--bg-void`: `#000000` — Pure OLED black canvas (zero power waste on OLED displays).
  - `--bg-surface`: `#09090b` — Deep industrial zinc surface for navigation bars and docks.
  - `--bg-card`: `#111114` — Elevated interactive card background.
  - `--border-subtle`: `#222226` — Minimal, crisp boundary lines.
  - `--border-focus`: `#06b6d4` — Neon cyan accent border for active states.
  - `--text-primary`: `#fafafa` — Crisp high-legibility headings and labels.
  - `--text-muted`: `#71717a` — Secondary metadata and version annotations.
  - `--accent-cyan`: `#06b6d4` — Primary tactile interactive glow and highlight color.
  - `--accent-emerald`: `#10b981` — Live online status and engine verification dot.
- **Typography:**
  - **Display / UI:** `Space Grotesk` — Geometric, engineered, and distinctive. Eliminates generic AI-generated sans-serif look.
  - **Monospace:** `JetBrains Mono` — For version strings (`1.21.4 • Fabric`), memory readouts (`4.0 GB / 16 GB`), and JVM arguments.
- **Motion Philosophy:**
  - **One Orchestrated Boot Sequence:** Top bar slides down (150ms) $\to$ Nametag and 3D avatar fade in (200ms) $\to$ Hero CTA glow blooms (250ms).
  - **Action-Response Only:** Motion thereafter is strictly reactive to user intent (button elevation, account slide glide, dropdown accordion unfold). Zero decorative bouncy animations.

---

### B. Dashboard Layout & Top Minimalist Nav Bar

```text
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ ✦ Rival Client    [ Home ]   [ Mods ]   [ Settings ]   [ Console ]               ─ □ ✕│ <- Sleek Top Minimalist Nav Bar
├────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                        │
│                                      ╭── Steve_PvP ──╮                                 │ <- Floating Nametag (Active Account Only)
│                                      │   ● Online    │                                 │
│                                      ╰───────────────╯                                 │
│                                       ┌─────────────┐        ░░░░░░░░░                 │
│                                       │             │        ░ SHADOW░                 │ <- Dimmed Shadow Avatar on the Right
│                                       │  [ ACTIVE ] │        ░ AVATAR░                 │    (Secondary Account in Depth, Name Hidden)
│                                       │  [ 3D SKIN] │        ░░░░░░░░░                 │
│                                       │             │                                  │
│                                       └─────────────┘        ─ ─ ─ ─ ─                 │
│                                        ─ ─ ─ ─ ─ ─ ─                                   │ <- Pedestal Glow
│                                                                                        │
│                           ┌──────────────────────────────┐                             │
│                           │     ▶   P L A Y   G A M E    │                             │ <- Centered Hero CTA (Cyan Glow)
│                           └──────────────────────────────┘                             │
│                           [ ⚡ 1.21.4 • Fabric v0.16.10 ▾ ]                            │ <- Attached Version Selector
│                                                                                        │
│                                                                                        │
├────────────────────────────────────────────────────────────────────────────────────────┤
│  ● Ready                                                         RAM 4.0 / 16.0 GB │ <- Minimal Bottom Ribbon
└────────────────────────────────────────────────────────────────────────────────────────┘
```

1. **Sleek Top Minimalist Navigation Bar:**
   - **Top-Left Identity:** `✦ Rival Client` branding pinned to the top-left with a glowing neon cyan accent on the `✦` star glyph. Draggable window region.
   - **Integrated Navigation Pills:** Minimalist tab switches positioned directly in the top bar: `[ Home ]`, `[ Mods ]`, `[ Settings ]`, `[ Console ]`.
   - **Top-Right Quick Actions:** Frameless window controls (`─`, `□`, `✕`).
2. **Minecraft Floating Nametag & Active 3D Avatar:**
   - Floating nametag sits directly **above** the active avatar's head in true Minecraft style, displaying the active player username (`Steve_PvP`) and live status badge (`● Online`).
   - Centerpiece features an interactive 3D rendered player skin model in full color and lighting with subtle idle breathing/floating motion.
3. **3D Depth Carousel & Shadow Avatar Switching (Max 3 Accounts):**
   - Secondary accounts stand to the right (and left) in depth as sleek, dimmed **shadow/silhouette avatars** with their nametags hidden to keep the canvas clean.
   - Clicking or swiping toward the shadow avatar initiates a fluid horizontal **slide animation**:
     - The shadow avatar glides into center spotlight, blooms into full color & texture, and its floating nametag dynamically appears above its head.
     - The previously active avatar glides to the left, gracefully receding into shadow depth.
   - Supports up to 3 accounts seamlessly in native 3D space without needing artificial dock widgets.
4. **Hero PLAY Action State Machine:**
   - **Idle:** Glowing cyan neon border with subtle hover elevation (`translateY(-2px)`) and letter-spaced `▶  P L A Y   G A M E`.
   - **Verifying / Downloading:** Replaces text with animated inline progress bar and status: `VERIFYING FABRIC ASSETS (42%)`.
   - **Launching:** Pulse glow animation with label: `SPAWNING DETACHED JVM...`.
   - **Running:** Emerald status dot with label: `IN GAME (DETACHED)` alongside quick "Open Console" and "Stop" actions.
5. **Attached Version Selector:**
   - Monospace dropdown selector positioned directly beneath the CTA: `[ ⚡ 1.21.4 • Fabric v0.16.10 ▾ ]`.
   - Expands a smooth accordion flyout listing installed profiles and version releases.
6. **Minimal Bottom Ribbon:**
   - Ultra-slim (28px height) status bar in monospace `JetBrains Mono` at 11px with a soft 1px zinc top border.
   - Left: Understated engine readiness dot (`● Ready` in emerald).
   - Right: Clean technical memory telemetry (`RAM 4.0 / 16.0 GB`). Eliminates duplicate version text and emoji clutter.

---

### C. Secondary Views Blueprint

#### 1. Mods View (Performance Stack Manager)
Inspired by Feather Client's clean mod organization, but strictly zero-bloat (no cosmetic stores or sponsored placements):
- **Search & Filter Header:** Instant search field + category filters (`Core Optimization`, `Shader Engine`, `Memory Fixes`).
- **Mod Cards Grid:**
  - Card Title: Mod name + installed version (e.g. `Sodium 0.6.1`).
  - Metric Badge: Performance contribution tag (`+65% FPS`, `-40% RAM Overhead`, `Mesh Shaders - RTX/GTX Only`).
  - Hardware Guard: NVIDIUM auto-displays a warning or disables toggle if a non-Turing GPU is detected.
  - Interactive Toggle Switch: Clean tactile switch with cyan active glow.

#### 2. Settings View (Hardware & Engine Tuning)
- **Interactive RAM Allocation Slider:**
  - Graduated visual slider from 2 GB to 16 GB with numerical input.
  - Real-time color-coded capacity meter: `Allocated to Game (4.0 GB)` / `System Reserved (12.0 GB)`.
- **Launcher Behavior on Launch:**
  - `Suspend WebView2 to Tray` (Preserves system resources; idle RAM `< 30 MB`).
  - `Close Launcher Completely` (Detached game process continues independently).
  - `Keep Open` (For live development and debugging).
- **G1GC Flag Presets:**
  - Quick-select presets: `Balanced (Default)`, `Low Latency / High FPS`, `High Render Distance (Heavy World Gen)`.
  - Advanced manual flag override field with syntax validation.

#### 3. Console View (Live Streaming Log Terminal)
- Monospace real-time log viewer streaming JVM stdout/stderr via Tauri IPC events.
- Log level filter chips: `[ All ]`, `[ Info ]`, `[ Warn ]`, `[ Error ]`.
- Utility buttons: `Copy Logs to Clipboard` and `Open .minecraft/logs`.

---

## 5. Process Lifecycle & Resource Strategy

To honor the **< 30 MB idle RAM** rule:

1. **Detached JVM Spawning:**
   - The game process (`java.exe`) is spawned completely detached from the launcher process tree.
   - When the game starts, Tauri can automatically **minimize to tray and suspend the WebView2 process**, or terminate entirely depending on user settings.
2. **G1GC Tuning Flags:**
   Launcher automatically constructs optimized JVM execution flags based on allocated RAM:
   ```text
   -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200
   -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch
   -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M
   -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4
   -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90
   -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32
   ```

---

## 6. Authentication Architecture

- **Microsoft OAuth 2.0 with PKCE:**
  1. Generate cryptographically random `code_verifier` and derived `code_challenge` (S256).
  2. Spawn temporary local loopback HTTP listener on `127.0.0.1:7878`.
  3. Open user's default browser to Microsoft OAuth authorization endpoint.
  4. Capture redirect callback `GET /callback?code=...`.
  5. Exchange auth code for Microsoft access and refresh tokens.
  6. Perform Xbox Live (XBL) authentication and obtain User Token.
  7. Authorize with XSTS (Xbox Security Token Service).
  8. Exchange XSTS token with Mojang authentication services for `mc_token`.
  9. Fetch Minecraft player profile (UUID and username).
  10. Store tokens securely in OS credential manager (`keyring`), never in plain files or frontend memory.
  11. Terminate loopback server immediately upon token capture.
- **Offline / Development Fallback:** Provides offline username profile launch for local testing without network dependency.

---

## 7. Discord Rich Presence

- Connects directly to the local Discord IPC pipe socket (`\\.\pipe\discord-ipc-0` on Windows, `/tmp/discord-ipc-0` on Unix).
- **Zero Overlay / DLL Injections:** Does NOT inject graphics hooks into the game process, eliminating OpenGL/Vulkan driver conflicts with Sodium and NVIDIUM.
- Emits lightweight JSON activity updates (current version, elapsed play time).

---

## 8. Roadmap & Phased Milestones

### Phase 1: Core Launcher MVP (Active)
- [ ] Initialize Vite + React 18 + Tailwind CSS + TypeScript frontend scaffold.
- [ ] Initialize Tauri v2 Rust backend with clean module layout (`auth`, `downloader`, `game`, `config`).
- [ ] Implement local Microsoft OAuth 2.0 PKCE loopback authentication engine.
- [ ] Build Tokio parallel asset and Fabric Loader downloader.
- [ ] Implement detached JVM spawner with pre-tuned G1GC optimization flags.
- [ ] Implement OLED dark UI: Centered Hero CTA, Version Switcher, Auto-hiding Sidebar, Settings.

### Phase 2: In-Game Integration & Distribution
- [ ] In-game Discord Rich Presence client.
- [ ] Automated Modrinth dependency updater.
- [ ] CI/CD automated cross-platform release pipeline on GitHub Actions.
- [ ] Optional lightweight native Fabric Mixin main menu.
