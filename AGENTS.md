# ✦ Agent Guidelines — Rival Client

This file provides system instructions, design constraints, and developer workflows for AI coding agents working on the **Rival Client** repository.

---

## 🎯 Project Identity & Philosophy

**Rival Client** is an open-source, hyper-lightweight Minecraft launcher and optimization client.

* **Zero-Bloat Rule:** Never introduce telemetry, tracking scripts, ad components, or daily promotion carousels.
* **Performance Constraints:** 
  * Launcher idle RAM usage must remain **< 30 MB** (via detached launch and Webview suspension).
  * Launcher startup time target is **< 0.4 seconds** (< 100ms for frameless splash).
  * 0ms extra render overhead in-game.

---

## 🛠️ Specialized Agent Skills

Before performing work on specific subsystems, consult the relevant skill in `.agents/skills/`:

| Skill | Path | Description |
|---|---|---|
| `karpathy-guidelines` | [`.agents/skills/karpathy-guidelines/SKILL.md`](.agents/skills/karpathy-guidelines/SKILL.md) | Think before coding, surgical diffs, verify success criteria. |
| `frontend-design` | [`.agents/skills/frontend-design/SKILL.md`](.agents/skills/frontend-design/SKILL.md) | High-contrast OLED dark styling, typography, avoiding AI tropes. |
| `Rival-tauri-backend` | [`.agents/skills/Rival-tauri-backend/SKILL.md`](.agents/skills/Rival-tauri-backend/SKILL.md) | Tauri v2 commands, Tokio async runtimes, G1GC tuning. |
| `Rival-react-frontend` | [`.agents/skills/Rival-react-frontend/SKILL.md`](.agents/skills/Rival-react-frontend/SKILL.md) | React 18, Zustand state, Lucide icons, Framer Motion rules. |
| `Rival-microsoft-auth` | [`.agents/skills/Rival-microsoft-auth/SKILL.md`](.agents/skills/Rival-microsoft-auth/SKILL.md) | Local PKCE loopback server, Mojang token exchange & keyring. |

---

## 🛠️ Project Tech Stack

* **Frontend (`src/`):** React 18, TypeScript, Vite, Tailwind CSS, Shadcn UI, Framer Motion, Lucide Icons, Zustand (State Management).
* **Backend (`src-tauri/`):** Rust 2021, Tauri v2, Tokio async runtime.
* **Authentication:** Local Microsoft OAuth 2.0 PKCE via a `127.0.0.1` Rust HTTP loopback server.
* **Game Engine:** Java JVM + Fabric Loader + Sodium / NVIDIUM / Iris / Lithium / FerriteCore / ModernFix.

---

## 📂 Codebase Map

```text
RivalClient/
├── .agents/skills/              # Specialized domain skills for AI agents
├── src-tauri/                   # Rust Backend (Tauri Native Layer)
│   ├── Cargo.toml
│   ├── tauri.conf.json
│   └── src/
│       ├── main.rs              # Tauri entry point & IPC handlers
│       ├── auth/                # OAuth 2.0 PKCE loopback server
│       ├── downloader/          # Async asset & Fabric downloader (Tokio)
│       ├── game/                # JVM process spawner & G1GC flag builder
│       └── config/              # Local settings & RAM persistence
└── src/                         # React Frontend (Vite UI)
    ├── App.tsx
    ├── components/
    │   ├── splash/              # Fast startup state / skeleton
    │   ├── dashboard/           # 3D avatar carousel (shadow avatar depth) & PLAY CTA
    │   ├── layout/              # Sleek top minimalist navbar & status ribbon
    │   ├── library/             # Profile & version manager
    │   ├── mods/                # Modrinth browser & optimization stack toggles
    │   └── settings/            # RAM allocation slider & JVM flags
    └── store/                   # Zustand global stores
```

---

## 💻 Agent Command Cheatsheet

Always use these standard commands when testing, linting, or building changes:

```bash
# Install frontend dependencies
pnpm install

# Start development mode (Frontend + Tauri Rust IPC hot reload)
pnpm tauri dev

# Build production bundle & executables
pnpm tauri build

# Rust linting & checking
cd src-tauri && cargo check
cd src-tauri && cargo clippy

# Frontend TypeScript checking
pnpm tsc --noEmit
```

---

## 📐 Design & Coding Standards for Agents

1. **UI/UX Rules (Frontend)**
   - **Theme Palette:** Strictly adhere to OLED Black (`#000000`) and Industrial Zinc (`#09090b`), accented by glowing neon cyan (`#06b6d4`), emerald (`#10b981`), or violet (`#8b5cf6`).
   - **Sleek Top Minimalist Navbar:** Pinned top bar featuring `✦ Rival Client` branding on the top left, minimalist tab switches (`Home`, `Mods`, `Settings`, `Console`), and top-right window controls.
   - **3D Avatar Depth Carousel & Shadow Avatar:** Centerpiece features the active 3D player avatar with a floating nametag above its head. Secondary accounts stand in depth as dimmed shadow avatars (nametag hidden). Clicking/sliding smoothly transitions between up to 3 accounts with native 3D horizontal slide motion.
   - **Centered Hero CTA Layout:** The main dashboard features the glowing centered PLAY GAME CTA directly beneath the 3D avatar stage, with the version switcher dropdown attached directly beneath it.
   - **Icons:** Use `lucide-react` icons exclusively.
   - **Animations:** Keep Framer Motion micro-animations subtle and performant (e.g., 3D depth glide transition, pulsing halo rings for active status badges, single orchestrated entrance).

2. **Backend Rules (Rust)**
   - **Async Non-Blocking:** All disk, network, and process operations must run asynchronously using Tokio.
   - **Security:** Keep OAuth token processing strictly local inside `src-tauri/src/auth/`. Never log credentials or tokens.
   - **Process Spawning:** JVM process spawning should run detached so the launcher can safely minimize or terminate to free system RAM.

3. **Commit Message Convention**
   When generating git commits, follow Conventional Commits:
   - `feat:` new launcher or UI feature
   - `fix:` bug fix or error handling adjustment
   - `perf:` RAM or startup speed optimization
   - `docs:` documentation updates (`ARCHITECTURE.md`, `README.md`)

---

## 🔗 Related Documentation
- Read the full technical specifications in [Architecture Guide](ARCHITECTURE.md).
- Review contribution guidelines in [Contributing](CONTRIBUTING.md).
- View general overview in [README](README.md).
