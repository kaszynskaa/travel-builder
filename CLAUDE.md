# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Vite dev server at http://localhost:5173
npm run build     # Production build → dist/
npm run preview   # Serve the production build locally
```

No lint or test runner is configured.

## Environment Setup

Copy `.env.example` to `.env` and set:

```
VITE_N8N_WEBHOOK_URL=https://your-n8n-instance.app.n8n.cloud/webhook/your-webhook-id
```

The Vite dev server proxies all `/api/n8n/*` requests to the n8n origin to avoid CORS issues (see `vite.config.js`). Without this variable, requests will hit `/api/n8n/webhook/missing-webhook-url` and fail.

## Architecture

React 18 SPA, no router, no state library. Tailwind + inline styles (mixed throughout).

### Data flow

1. User types a query in `QueryInterface` → state lives in `App`
2. `App.handleSubmit` calls `useItinerary().submit(query, callbacks)`
3. `useItinerary` (`src/hooks/useItinerary.js`) POSTs `{ prompt: query }` to the n8n webhook with a 120-second timeout
4. n8n runs three specialised AI agents in sequence and returns the itinerary as plain text or `{ output }` JSON
5. On success, `App` stores the raw text and renders `ItineraryResult`

### Response parsing (`ItineraryResult` → `parseItinerary`)

The raw text string from n8n is parsed client-side by `parseItinerary()` in `src/components/ItineraryResult.jsx`:
- Splits on `Day N` / `DAY N` boundaries → array of `{ number, title, content }` objects → rendered as `DayCard` grid
- Regex-searches for budget/cost and packing-list sections → rendered as `BudgetCard` / `PackingCard`
- Falls back to a raw `<pre>` block when no `Day N` structure is found
- A collapsible `<details>` always shows the full raw text when day cards are present

### App state machine

`status` in `App` drives what is rendered:

| status | rendered |
|--------|----------|
| `idle` | `Hero` only |
| `loading` | `Hero` (dimmed) + `LoadingOverlay` |
| `error` | `Hero` (dimmed) + `ErrorState` modal |
| `success` | `Hero` (dimmed) + `ItineraryResult` |

### Styling conventions

- Design tokens are CSS custom properties in `src/index.css` (obsidian, gold, parchment, ash)
- Shared utility classes defined via `@layer components` in `index.css`: `.glass`, `.glass-strong`, `.text-gradient-gold`, `.shadow-gold`, `.card-hover`, `.textarea-glass`
- Most component-level styles are inline objects (not Tailwind classes) to allow dynamic values
