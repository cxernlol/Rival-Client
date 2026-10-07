# ✦ Contributing to Rival Client

First off, thank you for considering contributing to Rival Client! 
Rival is built on a strict **"No-BS" philosophy**: zero bloat, zero telemetry, maximum raw FPS, and sub-second performance. We welcome contributions that align with these principles.

---

## 📜 Core Guidelines

1. **Keep it Lean:** Every kilobyte and millisecond counts. Avoid adding heavy dependencies, analytics scripts, or unnecessary UI clutter.
2. **Performance First:** Frontend components must be optimized, and Rust backend code should avoid blocking async runtimes.
3. **Open & Transparent:** No hidden code, no tracking, and no proprietary lock-ins.

---

## 🛠️ How to Contribute

### 1. Reporting Bugs
Found a glitch or crash? Open an issue on GitHub:
* Check existing [Issues](https://github.com/cxernlol/RivalClient/issues) to avoid duplicates.
* Provide clear steps to reproduce the issue.
* Include system specs (OS, GPU, Java version, launcher logs).

### 2. Requesting Features
Have an idea that improves client speed or quality of life?
* Open a feature request under [Issues](https://github.com/cxernlol/RivalClient/issues).
* Keep suggestions focused on performance, UI usability, or essential optimization mods.

### 3. Submitting Code (Pull Requests)

1. **Fork & Clone:**
   ```bash
   git clone https://github.com/cxernlol/RivalClient.git
   cd RivalClient
   ```
2. **Create a Future Branch:**
   ```Bash
   git checkout -b feat/your-feature-name
   # or fix/your-bug-fix
   ```

3. **Install & Run Locally:**
   ```Bash
   pnpm install
   pnpm tauri dev
   ```

## Commit Guidelines

We follow Conventional Commits:
- feat: add new JVM memory preset selector
- fix: prevent loopback OAuth server hang on auth cancel
- perf: reduce frontend bundle size
- docs: update ARCHITECTURE.md

## Submit a Pull Request:
Push your branch and open a PR targeting main. Be sure to describe your changes and why they are necessary.

## 🏗️ Codebase Overview
- Frontend (src/): React, TypeScript, Vite, Tailwind CSS, Shadcn UI.
- Backend (src-tauri/): Rust, Tauri v2, Tokio async runtime.
- Engine Injection: Fabric Loader + Core Optimization Mods (Sodium, Lithium, Iris, etc.).

For full technical details, consult the [Architecture Guide.](ARCHITECTURE.md)

Thank you for helping keep Minecraft clients fast, free, and open! 🚀
