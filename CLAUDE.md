# ✦ CLAUDE.md — Rival Client Context & Rules

This file provides context, commands, and rules for Claude Code and AI agents working in this repository.

---

## 🎯 Project Overview

**Rival Client** is an ultra-lightweight, open-source Minecraft optimization client and launcher built with Rust, Tauri v2, and React. Its mission is zero bloat, sub-second startup (< 0.4s), and sub-30MB idle RAM footprint while providing pre-packaged Fabric optimization mods (*Sodium, NVIDIUM, Lithium, Iris, FerriteCore, ModernFix*).

---

## 🛠️ Specialized Skills in `.agents/skills/`

Always check and follow the relevant skill when working on subsystems:
- [`.agents/skills/karpathy-guidelines/SKILL.md`](.agents/skills/karpathy-guidelines/SKILL.md): Behavioral coding discipline & minimalism.
- [`.agents/skills/frontend-design/SKILL.md`](.agents/skills/frontend-design/SKILL.md): High-contrast OLED dark styling & typography.
- [`.agents/skills/Rival-tauri-backend/SKILL.md`](.agents/skills/Rival-tauri-backend/SKILL.md): Tauri v2 backend commands & async Tokio rules.
- [`.agents/skills/Rival-react-frontend/SKILL.md`](.agents/skills/Rival-react-frontend/SKILL.md): React 18, Zustand state, Lucide icons, Framer Motion.
- [`.agents/skills/Rival-microsoft-auth/SKILL.md`](.agents/skills/Rival-microsoft-auth/SKILL.md): Local Microsoft PKCE loopback auth flow.

---

## 💻 Common Commands

### Development
```bash
pnpm install          # Install frontend dependencies
pnpm tauri dev        # Run React + Tauri Rust backend with hot-reload
```

### Build & Verification
```bash
pnpm tauri build      # Build production binary
pnpm tsc --noEmit     # Verify TypeScript types
```

### Rust Checks (from src-tauri/)
```bash
cargo check           # Fast Rust compiler check
cargo clippy          # Rust linter
cargo test            # Run backend test suite
```

---

## 🏗️ Architecture Summary
- Frontend (`src/`): React 18 + TypeScript + Vite + Tailwind CSS + Shadcn UI + Zustand.
- Backend (`src-tauri/src/`): Rust + Tauri v2 + Tokio async runtime.
  - `auth/`: Local Microsoft OAuth 2.0 PKCE loopback server (127.0.0.1).
  - `downloader/`: Multi-threaded async downloader for Fabric assets & mods.
  - `game/`: JVM process spawner and G1GC flag builder.
  - `config/`: Configuration & RAM state persistence.

---

## 📐 Coding Conventions & Rules

1. **Rust Backend:**
   - Never block the Tokio runtime — use async I/O and `tokio::spawn` for long tasks.
   - Spawn JVM processes detached so the launcher UI can minimize/close independently.
   - Keep OAuth token handling strictly local; never output credentials in logs.

2. **React Frontend:**
   - Components must use Functional Components with TypeScript interfaces.
   - Color palette is strictly OLED Dark (`#000000` / `#09090b`) with cyan/emerald/violet accents.
   - Typography uses `Space Grotesk` (UI) and `JetBrains Mono` (code/numbers).
   - State management uses Zustand stores (`src/store/`).

3. **Commits:**
   - Use Conventional Commits (`feat:`, `fix:`, `perf:`, `docs:`).

---

## 🚫 Strict Prohibitions
- ❌ NO Telemetry/Trackers: Never introduce analytics, tracking scripts, or external telemetry pingers.
- ❌ NO Monetization Fluff: Do not add store carousels, paid cosmetics, or ad banners.
- ❌ NO Blocking Calls: Never execute synchronous network or heavy file I/O operations on the main UI thread.
