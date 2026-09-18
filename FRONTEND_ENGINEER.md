# Frontend Engineer Guide

**Owns**: `frontend/`
**Deliverable**: an interactive, demo-able map application.

[← Back to main README](../../README.md)

---

## Objective

Give the project a visual, interactive surface — a map where a user picks a location, triggers inference, and sees the predicted segmentation mask overlaid on the satellite tile. This is the piece judges/reviewers/recruiters will actually engage with, so usability and polish matter here more than anywhere else in the stack.

## Scope

- Map interface and region selection
- Calling the backend API and handling async job state
- Rendering overlays (raw image vs. mask vs. side-by-side)
- Basic UX for loading, error, and empty states

## Responsibilities

### 1. Map interface
- Use Leaflet.js (lightweight, free) or Mapbox GL (nicer styling, needs an API key)
- Let the user click a point or draw a bounding box to select a region
- Show a date picker, defaulting to the most recent available imagery

### 2. API integration
- On selection, call `POST /predict`
- If the backend returns a job ID (async), poll `GET /jobs/{id}` with a loading indicator until the result is ready
- Handle and surface backend errors (no imagery available, region too cloudy, rate limited) clearly instead of failing silently

### 3. Visualization
- Overlay the returned mask on the map with adjustable opacity
- Provide a toggle: raw imagery / predicted mask / side-by-side split view
- Show metadata returned by the backend: tile date, cloud coverage %, inference latency

### 4. UX polish
- Loading states while inference runs (this can take a few seconds — don't leave the user guessing)
- Empty/error states with actionable messaging
- Responsive layout — this should work on a laptop screen at minimum, mobile is a stretch goal

## File structure

```
frontend/
├── src/
│   ├── components/
│   │   ├── MapView.jsx          # map + region selection
│   │   ├── OverlayToggle.jsx    # raw/mask/split toggle
│   │   ├── DatePicker.jsx
│   │   └── StatusBanner.jsx     # loading/error states
│   ├── api/
│   │   └── client.js            # backend API calls
│   └── App.jsx
├── public/
└── package.json
```

## Interfaces consumed

| Provider | Interface used |
|---|---|
| Backend Engineer | `POST /predict`, `GET /jobs/{id}` |

## Definition of done

- [ ] User can select a location and see a mask overlay within a reasonable time
- [ ] Loading and error states are handled, not left blank
- [ ] Toggle between raw/mask/split view works correctly
- [ ] App runs locally against the backend with a documented `npm run dev` flow

## Suggested tools

`React`, `Leaflet.js` (`react-leaflet`), `axios` or `fetch`, `Tailwind CSS` (optional, for fast styling)
