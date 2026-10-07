---
name: rival-react-frontend
description: >
  Skill for building Rival Client React/TypeScript components and Zustand stores.
  Use when working in src/: creating components, setting up routing, managing state,
  integrating Tauri IPC, or wiring up Framer Motion animations.
---

# Rival Client — React Frontend Skill

## Rules (from AGENTS.md)
- Components must be **Functional Components** with TypeScript interfaces — no class components.
- Color palette is **strictly** OLED Dark (`#000000` / `#09090b`) with cyan/emerald/violet accents.
- State management uses **Zustand** stores in `src/store/` — no prop-drilling, no Context for global state.
- Icons: **Lucide React** exclusively — no other icon libraries.
- Animations: **Framer Motion** — one orchestrated launch sequence, then action-response only.
- Never introduce analytics, tracking scripts, or external telemetry.

---

## Key Dependencies (package.json)
```json
{
  "dependencies": {
    "react": "^18",
    "react-dom": "^18",
    "@tauri-apps/api": "^2",
    "framer-motion": "^11",
    "lucide-react": "^0.400",
    "zustand": "^4",
    "tailwindcss": "^3",
    "@radix-ui/react-dropdown-menu": "^2",
    "@radix-ui/react-slider": "^1",
    "@radix-ui/react-tooltip": "^1"
  }
}
```

---

## Directory Structure
```
src/
├── App.tsx                  # Root: routing between splash / auth / dashboard
├── main.tsx                 # Vite entry
├── index.css                # Global styles, CSS custom properties (design tokens)
├── components/
│   ├── splash/
│   │   └── SplashScreen.tsx # Fast startup state / skeleton
│   ├── auth/
│   │   └── AuthView.tsx     # "Sign in with Microsoft" button
│   ├── dashboard/
│   │   ├── Dashboard.tsx    # Layout wrapper
│   │   ├── PlayButton.tsx   # Centered hero CTA with glow + pulsing ring
│   │   └── VersionSwitcher.tsx # Dropdown: profile/version selector
│   ├── layout/
│   │   ├── Navbar.tsx       # Sleek top minimalist navbar (✦ Rival Client)
│   │   └── StatusRibbon.tsx # Bottom engine status & RAM ribbon
│   ├── library/
│   │   └── Library.tsx      # Installed versions & profiles
│   ├── mods/
│   │   └── ModBrowser.tsx   # Modrinth API integration
│   └── settings/
│       ├── Settings.tsx     # Settings page wrapper
│       ├── RamSlider.tsx    # RAM allocation slider (Radix Slider)
│       └── JvmFlags.tsx     # JVM flags display/override
└── store/
    ├── useAuthStore.ts      # { account, isAuthenticated, login, logout }
    ├── useGameStore.ts      # { isRunning, launch, activeProfile, profiles }
    └── useSettingsStore.ts  # { ramMb, jvmFlags, setRam, saveSettings }
```

---

## Component Template
```tsx
import { type FC } from 'react'
import { motion } from 'framer-motion'
import { Play } from 'lucide-react'
import { useGameStore } from '@/store/useGameStore'

interface PlayButtonProps {
  disabled?: boolean
}

const PlayButton: FC<PlayButtonProps> = ({ disabled = false }) => {
  const { launch, isRunning } = useGameStore()

  return (
    <motion.button
      className="play-button"
      onClick={launch}
      disabled={disabled || isRunning}
      whileHover={{ y: -2, boxShadow: '0 0 30px rgba(6,182,212,0.3)' }}
      whileTap={{ scale: 0.98 }}
      transition={{ type: 'spring', stiffness: 400, damping: 25 }}
    >
      <Play size={18} />
      <span>P L A Y &nbsp; G A M E</span>
    </motion.button>
  )
}

export default PlayButton
```

---

## Zustand Store Template
```ts
// src/store/useGameStore.ts
import { create } from 'zustand'
import { invoke } from '@tauri-apps/api/core'

interface GameStore {
  isRunning: boolean
  activeProfile: string
  profiles: string[]
  launch: () => Promise<void>
  setProfile: (profile: string) => void
}

export const useGameStore = create<GameStore>((set, get) => ({
  isRunning: false,
  activeProfile: '1.21.4-fabric',
  profiles: [],

  launch: async () => {
    set({ isRunning: true })
    try {
      await invoke('launch_game', { profile: get().activeProfile })
    } catch (err) {
      console.error('Launch failed:', err)
      set({ isRunning: false })
    }
  },

  setProfile: (profile) => set({ activeProfile: profile }),
}))
```

---

## Tauri IPC Integration
```ts
import { invoke } from '@tauri-apps/api/core'
import { listen } from '@tauri-apps/api/event'

// Command call (request/response)
const result = await invoke<string>('command_name', { param: value })

// Event listener (backend → frontend push)
const unlisten = await listen<ProgressPayload>('download_progress', (event) => {
  console.log(event.payload.percent)
})
// Call unlisten() in useEffect cleanup
```

---

## CSS Design Tokens (index.css)
```css
@import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap');

:root {
  --bg-void:        #000000;
  --bg-surface:     #09090b;
  --bg-elevated:    #111113;
  --border:         #27272a;
  --text-primary:   #fafafa;
  --text-muted:     #71717a;
  --accent-cyan:    #06b6d4;
  --accent-emerald: #10b981;
  --accent-violet:  #8b5cf6;
  --accent-glow:    rgba(6, 182, 212, 0.15);
  
  font-family: 'Space Grotesk', sans-serif;
}

* { box-sizing: border-box; margin: 0; padding: 0; }

body {
  background: var(--bg-void);
  color: var(--text-primary);
  -webkit-font-smoothing: antialiased;
}

.play-button {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 16px 48px;
  border: 1px solid var(--accent-cyan);
  background: transparent;
  color: var(--accent-cyan);
  font-family: 'Space Grotesk', sans-serif;
  font-size: 15px;
  font-weight: 600;
  letter-spacing: 0.15em;
  border-radius: 4px;
  cursor: pointer;
  box-shadow: 0 0 20px var(--accent-glow), inset 0 0 20px var(--accent-glow);
  transition: box-shadow 0.2s ease;
}
```

---

## Vite + TypeScript Config
Ensure `tsconfig.json` has path aliases:
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": { "@/*": ["./src/*"] }
  }
}
```
And `vite.config.ts`:
```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: { alias: { '@': path.resolve(__dirname, './src') } },
})
```

---

## Commands
```bash
pnpm install          # Install all dependencies
pnpm tauri dev        # Hot-reload dev mode (frontend + Rust IPC)
pnpm tsc --noEmit     # TypeScript type check
```
