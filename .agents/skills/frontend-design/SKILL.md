---
name: frontend-design
description: >
  Guidance for distinctive, intentional visual design when building or reshaping
  the Rival Client UI. Helps with aesthetic direction, typography, motion, and
  making choices that don't read as templated defaults. Adapted from the
  Anthropic claude-code frontend-design skill for the Rival Client context.
license: See original at github.com/anthropics/claude-code — Complete terms in LICENSE.txt
---

# Frontend Design — Rival Client Edition

Approach this as the design lead at a studio known for giving every client a distinct visual identity. Rival's audience is **performance-obsessed PC gamers** who run OLED monitors, custom desk setups, and despise anything that feels bloated, corporate, or cheap. They have already rejected proposals that felt cliché or templated. Make deliberate, opinionated choices specific to this brief.

---

## Ground Every Design in Rival's World

Before designing any screen, confirm:
- **Who sees it?** Hardcore Minecraft players who care about FPS, not casual users.
- **What's its job?** Get them into the game as fast as possible, with zero friction.
- **What's the vernacular?** Dark terminal aesthetics, hardware RGB, OLED contrast, engineering precision.

The subject's industry (high-performance gaming software) is where distinctive visual choices come from. A Rival screen should feel like a gaming peripheral's companion app — not a SaaS dashboard or a consumer app store.

---

## Design Principles

### Hero / First Impression
The dashboard hero is the first thing the player sees. Open with the most characteristic thing in Rival's world: a **prominent, glowing PLAY button** with a pulsing halo ring — not a generic "welcome" banner, a stat card, or a news carousel. The centered CTA *is* the hero.

### Palette — Strict OLED Dark
| Token | Hex | Usage |
|---|---|---|
| `--bg-void` | `#000000` | True OLED black background |
| `--bg-surface` | `#09090b` | Card / panel surfaces |
| `--bg-elevated` | `#111113` | Hover states, dropdowns |
| `--border` | `#27272a` | Subtle dividers |
| `--text-primary` | `#fafafa` | Headlines, CTA labels |
| `--text-muted` | `#71717a` | Secondary text, metadata |
| `--accent-cyan` | `#06b6d4` | Primary interactive accent |
| `--accent-emerald` | `#10b981` | Success / active status |
| `--accent-violet` | `#8b5cf6` | Secondary accent |
| `--accent-glow` | `rgba(6,182,212,0.15)` | Glow halos, shadows |

Never use warm beige (#F4F1EA), warm whites, or light-mode palettes.

### Typography
Use **one or two** font families. The combination for Rival:
- **Display / UI:** `Space Grotesk` — geometric sans, feels engineered and precise.
- **Mono / Code:** `JetBrains Mono` — for version strings, JVM flags, memory readouts.

Do NOT reach for Inter, Roboto, or system-ui as primary display fonts — they are the AI-generated default and will make the UI look generic.

**Avoid these typographic tells of generated UIs:**
- Accenting a single word in a headline with a different color.
- ALL CAPS labels everywhere (use sparingly, only for status badges like `● ONLINE`).
- Unnecessary eyebrow labels above every section heading.
- Generic `font-size: 14px` body text — establish a deliberate type scale.

**Type scale:**
```
--text-xs:   11px / 1.4  (metadata, version strings)
--text-sm:   13px / 1.5  (sidebar nav, secondary labels)
--text-base: 15px / 1.6  (body)
--text-lg:   18px / 1.4  (card titles)
--text-xl:   24px / 1.2  (section headers)
--text-2xl:  32px / 1.1  (PLAY button label, hero text)
--text-3xl:  48px / 1.0  (branding / splash logo)
```

### Motion
**One orchestrated launch sequence.** When the dashboard loads, run a single composed animation:
1. Background fades in (150ms)
2. Status ribbon slides up from bottom (200ms, ease-out)
3. Sidebar ghost fades in (250ms)
4. PLAY button scales up from 0.9 with glow bloom (300ms, spring)

**After that, motion only answers user actions:**
- Button hover → subtle glow intensity increase + 2px upward translate
- Sidebar hover → smooth expand (240ms ease-out)
- Version dropdown open → height expand with fade
- Active status badge → slow pulsing halo ring (3s loop, not distracting)

**Do NOT:**
- Fade-and-slide-up each section independently on scroll.
- Add hover transitions to every card.
- Use `animate-bounce` or `animate-spin` anywhere.
- Add parallax scrolling effects.

### Visual Structure is Information
- Dividers = meaningful section breaks, not decoration.
- Borders on cards = only when the card needs to be distinguished from the surface.
- Numbered markers (01/02/03) = only if the content is a genuine sequence.
- Status dots (`●`) = use to encode live state (online, downloading, active mod).

---

## Rival-Specific UI Patterns

### 3D Depth Carousel & Shadow Avatar Switching
```text
        ╭── Steve_PvP ──╮    ← Floating nametag above head (active account only)
        │   ● Online    │
        ╰───────────────╯
         ┌─────────────┐        ░░░░░░░░░
         │  [ ACTIVE ] │        ░ SHADOW░  ← Dimmed shadow avatar on right
         │  [ 3D SKIN] │        ░ AVATAR░    (Secondary account, name hidden)
         └─────────────┘        ░░░░░░░░░
          ─ ─ ─ ─ ─ ─ ─          ─ ─ ─ ─   ← Pedestal shadow glow
```
- Floating nametag sits above the active avatar's head in true Minecraft visual style
- Secondary accounts stand in depth as sleek, dimmed shadow/silhouette avatars with nametags hidden
- Clicking or sliding toward the shadow avatar initiates a 3D horizontal glide:
  - The shadow avatar glides into center spotlight, blooming into full color/texture with nametag revealed
  - The previous avatar glides into depth shadow on the other side
- Eliminates artificial dock widgets between avatar and the Play button for maximum visual purity

### PLAY Button
```
┌─────────────────────────────────┐
│   ▶   P L A Y   G A M E        │   ← Centered, large, glowing cyan border
└─────────────────────────────────┘
```
- `border: 1px solid var(--accent-cyan)`
- `box-shadow: 0 0 20px var(--accent-glow), inset 0 0 20px var(--accent-glow)`
- On hover: glow intensifies, `translateY(-2px)`
- Label uses letter-spacing for the "gaming terminal" feel

### Version Switcher
```
[ ⚡ 1.21.4 • Fabric v0.16.0  ▾ ]
```
- Inline dropdown, monospace version strings
- Sits directly below PLAY button
- Compact — never expands the dashboard layout

### Status Ribbon (bottom bar)
```text
● Ready                                               RAM 4.0 / 16.0 GB
```
- Fixed to bottom of window, ultra-slim 28px height
- Monospace `JetBrains Mono` at 11px on `--bg-surface` with subtle top border
- Left: quiet engine status dot (`● Ready` in emerald)
- Right: technical memory telemetry (`RAM 4.0 / 16.0 GB`)
- Completely free of emoji clutter and redundant text

### Sleek Top Minimalist Navbar
```
┌────────────────────────────────────────────────────────────────────────┐
│ ✦ Rival Client    [ Home ]   [ Mods ]   [ Settings ]   [ Console ]   [● Steve] ─ □ ✕  │
└────────────────────────────────────────────────────────────────────────┘
```
- Pinned to the top of the window, acts as both draggable header and navigation bar
- Left: `✦ Rival Client` branding with subtle cyan glow on `✦`
- Center: Minimalist tab navigation pills with subtle active indicator
- Right: Account pill (`[●] Steve`) and frameless window controls (`─`, `□`, `✕`)

---

## Process: Plan → Review → Build → Critique

Before writing a single line of component code:
1. **Name the screen** and its single job.
2. **Confirm the data** it needs (what does Zustand store provide?).
3. **Sketch the layout** in comments — what is centered, what is pinned.
4. **Review against the brief:** Does it match the OLED dark palette? Does it have zero bloat?
5. **Build** the component.
6. **Critique against AI-generated tells:** Remove any cream backgrounds, gradient accents on numbers, scattered hover animations, or ALL CAPS label eyebrows.
