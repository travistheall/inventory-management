---
name: saas-sidebar-redesign
description: Guidelines for converting this app's top-nav layout into a modern SaaS-style interface with a vertical left sidebar. Use when redesigning the UI shell/navigation, or asked to make the app look "more like a SaaS product."
---

# SaaS Sidebar Redesign

This skill describes how to convert the Factory Inventory Management System's current top-nav layout into a modern SaaS-style shell: a vertical left sidebar for navigation, consistent spacing, and a more polished, professional look.

## Delegation Reminder

Per the root `CLAUDE.md`: **any creation or significant modification of a `.vue` file must be delegated to the `vue-expert` subagent.** This skill is the design spec — hand it to `vue-expert` and let it execute the changes; don't edit `.vue` files directly from the main agent.

## When to Use This Skill

- Redesigning the app shell or navigation (`App.vue`, nav components)
- Explicit requests for a "sidebar", "modern SaaS UI", or "more polished/professional" look
- Any restyle that changes how users move between Dashboard/Inventory/Orders/Demand/Spending/Reports

## Current → Target Shell Structure

**Current** (`client/src/App.vue`):
```
.top-nav (horizontal header: logo + .nav-tabs + LanguageSwitcher + ProfileMenu)
  ↓
<FilterBar/>
  ↓
.main-content > <router-view/>
```

**Target**: a two-column app shell.

- **Left**: a fixed-width vertical `Sidebar` component (new file: `client/src/components/Sidebar.vue`) containing:
  - Brand/logo at the top (reuse the existing `.logo` markup/copy from `App.vue`)
  - Nav links stacked vertically — same 6 routes and i18n keys as today (`nav.overview`, `nav.inventory`, `nav.orders`, `nav.finance`, `nav.demandForecast`, and "Reports" — see note below), same active-state logic (`$route.path` comparison)
  - A bottom-anchored footer area for `ProfileMenu` and `LanguageSwitcher`
- **Right**: a column with a slim top bar (page context + `<FilterBar/>`) above `<router-view/>`, which scrolls independently of the sidebar.

`App.vue`'s root becomes a CSS Grid: `grid-template-columns: <sidebar-width> 1fr;`. The sidebar is full-viewport-height and sticky/fixed; the right column scrolls on its own.

## Design Principles / Checklist

- **Sidebar width**: ~240–260px, full viewport height, `position: sticky` (or fixed) so it stays put while content scrolls.
- **Sidebar background**: visually distinct from the content area (e.g. white with a right border, or a subtle slate tint) — don't let it blend into `.main-content`.
- **Brand**: logo + company name at the top of the sidebar, matching existing copy.
- **Nav items**: vertical stack, one per row, generous click target (match existing `padding: 0.625rem 1.25rem` scale or larger). Active item uses the existing accent color (`#2563eb` text/background) with a left-edge accent bar instead of the current bottom-border trick. Hover state matches the existing `.nav-tabs a:hover` treatment (`background: #f1f5f9`, darker text).
- **All 6 routes preserved**: Overview, Inventory, Orders, Finance (Spending), Demand Forecast, Reports. Note: "Reports" is currently hardcoded in `App.vue` rather than pulled from `useI18n` — flag this instead of silently carrying the gap forward; add an `nav.reports` key to `locales/en.js` and `locales/ja.js` if you're touching this area.
- **ProfileMenu and LanguageSwitcher**: relocate into the sidebar footer (or the new top bar) — don't drop functionality or remove their modals (`ProfileDetailsModal`, `TasksModal`).
- **FilterBar**: moves into the new top bar above `<router-view/>`, not into the sidebar — it's page content, not navigation.
- **Spacing**: standardize on the existing 4/8px-multiple scale already used for cards/stats (`0.625rem`, `1.25rem`, `1.5rem`, `2rem`, etc.). While touching global styles, unify any stray one-off spacing values you find rather than adding new ones.
- **Color palette**: stay within the existing slate/gray + accent-blue system (`#0f172a`, `#64748b`, `#e2e8f0`, background `#f8fafc`, accent `#2563eb`). No new colors, no emojis.
- **Leave alone unless clashing**: existing card/table/badge styles (`.card`, `.stat-card`, `table`, `.badge`) are already consistent — don't rewrite them just because the shell changed.
- **Responsive (nice-to-have, not required for "done")**: consider collapsing the sidebar to icons-only or a drawer below a small-viewport breakpoint.

## Step-by-Step Process

1. Extract the nav markup (logo + `.nav-tabs` links) from `App.vue` into `client/src/components/Sidebar.vue`, keeping the same `router-link` / `$route.path` active-state logic and i18n calls.
2. Restructure `App.vue`'s template into the grid shell: `<Sidebar/>` on the left, a right column containing the new top bar (`<FilterBar/>` + relocated `ProfileMenu`/`LanguageSwitcher` if not placed in the sidebar) and `<router-view/>` below it. Keep prop/emit wiring for `ProfileMenu`, modals, and tasks exactly as it is today.
3. Rewrite the layout CSS in `App.vue`'s global `<style>` block: remove `.top-nav`, `.nav-container`, `.nav-tabs` rules; add `.app-shell` (grid), `.sidebar`, `.sidebar-nav`, `.topbar` rules. Leave card/table/badge rules untouched unless they visually clash with the new shell.
4. Verify with the dev server (`npm run dev`, port 3000) and Playwright MCP:
   - Sidebar renders on every route and stays visible while scrolling content
   - Active-link highlighting follows `$route.path` correctly on each of the 6 routes
   - `ProfileMenu`, its dropdown, `ProfileDetailsModal`, and `TasksModal` still open/close and function
   - `LanguageSwitcher` still toggles locale
   - `FilterBar` still filters data correctly from its new position
   - No console errors (`mcp__playwright__browser_console_messages`)

## Anti-Patterns to Avoid

- Don't hardcode colors outside the existing palette
- Don't break existing i18n keys, or add new hardcoded English strings where a `t()` key already exists elsewhere
- Don't use array index as the `:key` for nav items — use the route path
- Don't remove `FilterBar` functionality while relocating it — it must still call `resetFilters`/update the composable state
- Don't skip the loading/error states already present in views — the shell change shouldn't touch view-level data flow
- Don't modify `server/` or API contracts — this is a client-only visual/structural change
