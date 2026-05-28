# ADR-001: Pixel Art Dashboard — Decision Record

**Date:** 2026-05-27
**Status:** Implemented
**Agent:** Goop
**Context:** Alex directed Goop + Lil Claw to build "moving characters as agents" dashboard per the original `agent-dashboard-vision.md` spec. Goop made all implementation decisions autonomously per Alex's "U decide, dont ask me question" directive.

---

## Decision 1: New route `/pixel` rather than replacing `/`

**Decision:** Created a new `/pixel` route alongside the existing dark ops `/` dashboard.

**Why:**
- The dark ops terminal dashboard (Fira Code, #00ff88 accent) is already deployed and verified working. Replacing it risked introducing regressions with no rollback path.
- Alex's original vision in `agent-dashboard-vision.md` describes pixel art / Stardew Valley aesthetic — fundamentally different from the terminal design. Building as a separate route preserves both.
- Pixel art is explicitly marked as "v2" in the original spec. A separate route matches that intent.

**Alternative considered:** Replace `/`. Rejected — too risky with no approval workflow for a live deployment.

---

## Decision 2: CSS-generated sprites (no external sprite assets)

**Decision:** Agent sprites are constructed from CSS `div` elements (colored rectangles for head, body, legs) rather than PNG/JPG pixel art assets.

**Why:**
- No new npm packages, no image assets to manage or host
- CSS sprites are resolution-independent and load instantly
- Animation via CSS (`sprite-idle`, `sprite-working` keyframes) is lightweight
- Future enhancement: swap in actual pixel art PNG sprites without changing component structure

**Alternative considered:** Pixel art PNG sprites. Rejected — would require asset pipeline, hosting, and could break on HiDPI screens without careful scaling.

---

## Decision 3: Press Start 2P font (Google Fonts)

**Decision:** Used `Press Start 2P` from Google Fonts as the primary pixel font.

**Why:**
- Authentic 8-bit pixel aesthetic — the spec explicitly referenced retro game style
- No self-hosted font files needed — loaded from Google Fonts CDN
- Works in static export (GitHub Pages) without additional configuration
- Slightly more readable than `VT323` at small sizes (7-10px body text)

**Alternative considered:** VT323. Rejected — larger visual footprint at the sizes we need (body text 7-10px), less authentic 8-bit feel.

---

## Decision 4: Stamina = inbox pressure, Gold = outbox count

**Decision:** Mapped game-style stats to agent data as follows:
- **Stamina bar** = `(inbox_count / 10) * 100` — represents how busy/pressured the agent is
- **Gold counter** = `outbox_count` — represents work completed

**Why:**
- Inbox count directly reflects pending workload — high inbox = high stamina drain (makes thematic sense)
- Outbox count reflects historical work output — more tasks completed = more gold earned (matches RPG logic)
- Both metrics are live from the existing `/state` API — no new data sources needed
- Visual is a proxy/proxy — real token usage and budget data exist but the router endpoint exposes `total_cost_usd` which is displayed in the header gold counter separately

**Alternative considered:** Real token stamina from model context window usage. Rejected — would require model-specific context window sizes and instrumentation we don't have in the current API.

---

## Decision 5: Static export (`output: "export"`) + GitHub Pages

**Decision:** Built using Next.js `output: "export"` which generates static HTML/CSS/JS in `/out`.

**Why:**
- GitHub Pages (the current hosting setup for `zeimhahnu.github.io`) only supports static sites
- FastAPI dashboard API (port 8443) serves the data — the React app is purely a read-only frontend
- VPS Nginx serves static files from `/var/www/openclaw-dashboard-web/out/` — this is already the established deployment pattern
- Static export means the pixel route works identically on both GitHub Pages and VPS Nginx once `/out/` is synced

**Alternative considered:** Next.js server-side rendering with Nginx proxying to a Node.js process. Rejected — adds complexity and a Node dependency on the VPS.

---

## Decision 6: A2A particle animation (CSS-only, no canvas)

**Decision:** A2A communication between agents is visualized as gold particle dots that fly across the screen using CSS animations + `requestAnimationFrame`.

**Why:**
- No Canvas API needed — pure CSS + React state is sufficient
- Particles are purely decorative/ambient — they enhance the "lived world" feel per the original spec without adding functional value
- Implementation is self-contained in `components.tsx` as `A2AParticles` + `ParticleDot`
- Triggered randomly when agents have tasks (70% chance every 3s if 2+ agents have data)
- Max 5 particles on screen at once to avoid performance issues

**Alternative considered:** WebSocket-driven real A2A events. Rejected — the dashboard is read-only observability, not a control panel. Random simulation is sufficient for the ambient feel.

---

## Decision 7: Build isolation — clone, build, push

**Decision:** When the `.next` build directory became root-owned (blocked by `npm run build`), I cloned the repo to `/tmp/px-build`, built there, and pushed from there.

**Why:**
- The permission issue was caused by a previous root-run build leaving root-owned files in `.next/`
- Waiting for Alex to run `sudo rm -rf .next` would have delayed the feature
- The `/tmp` clone approach was fast and worked reliably

**Lesson:** Document that future builds on the workspace repo should be done as `openclaw` user, not root. If root builds happen, `.next` becomes inconsistent with workspace ownership.

---

## Decision 8: Poll existing `/state` endpoint (no new API)

**Decision:** Pixel dashboard uses the same `https://dashboard.vpszeimhahnu.uk/state` endpoint as the terminal dashboard.

**Why:**
- No new backend API needed — the pixel dashboard is purely a different frontend visualization of the same data
- Polling interval: 7000ms (same as terminal dashboard)
- Data shape used: `agents[name].inbox_count`, `agents[name].working_count`, `router.decisions_count`, `router.total_cost_usd`

---

## Pending Decisions (for next iteration)

1. **Real sprite assets** — when pixel art PNG sprites are available, swap `AgentSprite` CSS divs for actual sprite images
2. **Context window stamina** — instrument real token usage per agent to show genuine stamina bars
3. **Actual A2A events** — replace simulated particles with real `sessions_send` event captures (would need a WebSocket or polling side-channel)
4. **Quest log from actual tasks** — pull active tasks from `/tasks/active` endpoint instead of hardcoded Sprint 10 list

---

*ADR written by Goop on 2026-05-27. Part of: `SPECS/adr-pixel-dashboard-2026-05-27.md`*