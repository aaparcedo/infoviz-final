# AI Now

A scrollytelling piece on the measurable consequences of the AI boom. Five charts: capital, geography, FDA medical-device authorizations, public perception, and teen use of AI companions.

Built with Svelte 5 + Vite + D3.

## Run

```
npm install
npm run dev
```

Open http://localhost:5173.

## Build

```
npm run build
```

Outputs to `dist/`.

## Structure

- `src/App.svelte` — page composition: hero, five `<Beat>` sections, close
- `src/lib/Beat.svelte` — scrollytelling scaffolding (sticky chart on the right, scrolling text on the left). Tracks `activeStep` (which step the reader is on) and `scrollProgress` (continuous 0–1 through the beat) and exposes both to the chart inside.
- `src/lib/charts/Chart{1..5}*.svelte` — one D3 chart per beat
- `src/app.css` — typography, color tokens, base styles
- `public/data/*.csv` — chart-ready datasets, fetched at runtime
