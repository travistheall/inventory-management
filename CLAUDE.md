# CLAUDE.md

Factory Inventory Management System Demo with GitHub integration - Full-stack application with Vue 3 frontend, Python FastAPI backend, and in-memory mock data (no database).

> ⚠️ **This repository and any fork you create are PUBLIC.** Do not commit credentials, internal hostnames, or private registry URLs. `client/.npmrc` pins the public npm registry and `client/package-lock.json` is gitignored to prevent locally-configured registries from leaking into commits — leave both in place.

## Critical Tool Usage Rules

### Subagents
Use the Task tool with these specialized subagents for appropriate tasks:

- **vue-expert**: Use for Vue 3 frontend features, UI components, styling, and client-side functionality
  - Examples: Creating components, fixing reactivity issues, performance optimization, complex state management
  - **MANDATORY RULE: ANY time you need to create or significantly modify a .vue file, you MUST delegate to vue-expert**
- **code-reviewer**: Use after writing significant code to review quality and best practices
- **security-auditor**: Use for a focused security pass on changed files (critical vulnerabilities only)
- **Explore**: Use for understanding codebase structure, searching for patterns, or answering questions about how components work
- **general-purpose**: Use for complex multi-step tasks or when other agents don't fit

### Skills
- **backend-api-test** skill: Use when writing or modifying tests in `tests/backend` directory with pytest and FastAPI TestClient

### Slash Commands
Repo-specific commands in `.claude/commands/`: `/start` and `/stop` (dev servers on ports 3000/8001), `/test` (run full test suite), `/demo-branch` (new demo branch, auto-numbered), `/reset-branch` (discard branch and reset to main — destructive), `/optimize` (dead code/dependency cleanup pass).

### MCP Tools
- **ALWAYS use GitHub MCP tools** (`mcp__github__*`) for ALL GitHub operations
  - Exception: Local branches only - use `git checkout -b` instead of `mcp__github__create_branch`
- **ALWAYS use Playwright MCP tools** (`mcp__playwright__*`) for browser testing
  - Test against: `http://localhost:3000` (frontend), `http://localhost:8001` (API)

## Stack
- **Frontend**: Vue 3 + Composition API + Vite (port 3000)
- **Backend**: Python FastAPI (port 8001)
- **Data**: JSON files in `server/data/` loaded via `server/mock_data.py`

## Quick Start

```bash
# One-command (macOS/Linux): ./scripts/start.sh — or use the /start slash command
./scripts/start.sh

# Manual
cd server && uv run python main.py   # http://localhost:8001, docs at /docs
cd client && npm install && npm run dev  # http://localhost:3000
```

## Testing

```bash
cd tests
uv run pytest -v                              # all backend tests
uv run pytest backend/test_inventory.py -v    # single file
```
No frontend test suite exists yet.

## Key Patterns

**Filter System**: 4 filters (Time Period, Warehouse, Category, Order Status) apply to all data via query params
**Data Flow**: Vue filters → `client/src/api.js` → FastAPI → In-memory filtering → Pydantic validation → Computed properties
**Reactivity**: Raw data in refs (`allOrders`, `inventoryItems`), derived data in computed properties
**i18n**: `useI18n` composable (`client/src/composables/useI18n.js`) supports `en`/`ja` via `client/src/locales/*.js`; locale is persisted in `localStorage` and drives currency (USD/JPY) — see `client/src/utils/currency.js`
**Auth**: `useAuth` composable is fully mocked (no real backend auth/session) — user data is hardcoded and just switches by locale

## API Endpoints
- `GET /api/inventory`, `/api/inventory/{id}` - Filters: warehouse, category
- `GET /api/orders`, `/api/orders/{id}` - Filters: warehouse, category, status, month (also accepts quarters like `Q1-2025`)
- `GET /api/dashboard/summary` - All filters
- `GET /api/demand`, `/api/backlog` - No filters
- `GET /api/spending/*` - Summary, monthly, categories, transactions
- `GET /api/reports/quarterly`, `/api/reports/monthly-trends` - Computed from `orders`, no filters
- Note: `CreatePurchaseOrderRequest` is defined in `server/main.py` but there is no `POST` route wired up for it yet — `purchase_orders` data is currently read-only (used to flag `has_purchase_order` on backlog items).

## Code Style
- Always document non-obvious logic changes with comments

## Common Issues
1. Use unique keys in v-for (not `index`) - use `sku`, `month`, etc.
2. Validate dates before `.getMonth()` calls
3. Update Pydantic models when changing JSON data structure
4. Inventory filters don't support month (no time dimension)
5. Revenue goals: $800K/month single, $9.6M YTD all months

## File Locations
- Views: `client/src/views/*.vue`
- Components (modals, filter bar, etc.): `client/src/components/*.vue`
- Composables: `client/src/composables/` (`useFilters`, `useI18n`, `useAuth`)
- API Client: `client/src/api.js`
- Backend: `server/main.py`, `server/mock_data.py`
- Data: `server/data/*.json`
- Styles: `client/src/App.vue`

## Design System
- Colors: Slate/gray (#0f172a, #64748b, #e2e8f0)
- Status: green/blue/yellow/red
- Charts: Custom SVG, CSS Grid for layouts
- No emojis in UI
