# ✦ Rival Client

> Hyper-lightweight, 100% open-source Minecraft launcher and optimization client. Zero bloat, zero tracking, maximum raw FPS.

---

## ⚡ The Solution to Client Bloat

Tired of traditional clients pushing daily cosmetic shops, battle passes, and consuming 1GB+ of system memory just idling in the background?

**Rival Client** rethinks the Minecraft launcher from the ground up using **Rust**, **Tauri v2**, and **React 18**:

| The Industry Standard | Rival Client |
| :--- | :--- |
| ❌ 500 MB – 1.2 GB idle RAM footprint | ✅ **< 30 MB idle RAM footprint** (suspends on game launch) |
| ❌ Intrusive store banners & paid cosmetics | ✅ **Zero ads, zero monetization, zero bloat** |
| ❌ 5+ second sluggish launcher startup | ✅ **Sub-second instant startup (< 0.4s)** |
| ❌ Closed-source binaries with telemetry | ✅ **100% Open Source & verifiable (0% trackers)** |

---

## 🚀 Key Features

* **Pre-Tuned Optimization Stack:** Built-in support for the leading modern performance mods: **Sodium**, **Lithium**, **FerriteCore**, **ModernFix**, **Iris**, and **NVIDIUM** (Mesh Shaders on NVIDIA Turing+).
* **Pure Local Authentication:** Authenticates directly via official Microsoft OAuth 2.0 PKCE over a local `127.0.0.1` Rust loopback server. Tokens are stored in your OS keyring and never leave your machine.
* **Focused OLED Interface:** Clean, high-contrast dark design featuring a sleek top minimalist navigation bar with `✦ Rival Client` branding, a centered **PLAY** action, and an attached version switcher without distracting news carousels or store banners.
* **Detached Game Process:** The Java JVM process is launched completely detached, allowing the launcher to minimize to tray or close to preserve all system resources for gameplay.

---

## 🛠️ Technology Stack

* **Frontend:** React 18 • TypeScript • Vite • Tailwind CSS • Shadcn UI • Zustand • Lucide Icons
* **Backend:** Rust 2021 • Tauri v2 • Tokio Async Runtime • Reqwest • Keyring
* **Game Runtime:** Java JVM + Fabric Loader

---

## 💻 Development Quickstart

### Prerequisites
- [Node.js](https://nodejs.org/) (v18+) and [pnpm](https://pnpm.io/)
- [Rust](https://rustup.rs/) (stable toolchain)
- [Tauri v2 Prerequisites](https://v2.tauri.app/start/prerequisites/)

### Setup
```bash
# Clone the repository
git clone https://github.com/cxernlol/RivalClient.git
cd RivalClient

# Install frontend dependencies
pnpm install

# Run frontend + Tauri native layer with hot-reload
pnpm tauri dev
```

### Verification & Linting
```bash
# Check TypeScript types
pnpm tsc --noEmit

# Check Rust backend
cd src-tauri && cargo check
```

---

## 📖 Documentation
- [Technical Architecture Guide](ARCHITECTURE.md)
- [Agent Guidelines & Coding Standards](AGENTS.md)
- [Claude Coding Rules](CLAUDE.md)
- [Contributing Guide](CONTRIBUTING.md)

---

## 📄 License
This project is licensed under the terms described in the [LICENSE](LICENSE) file.
