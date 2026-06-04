# Carousel + Configurable Events Design

**Date:** 2026-06-04
**Project:** lorena.github.io — Dia dos Namorados page

## Purpose

The current `index.html` shows the three-day programação (June 12, 13, 14 of 2026)
as a static 3-column timeline. This design replaces that timeline with:

1. A **carousel** that shows one day at a time with arrow navigation.
2. A **JSON configuration** that drives all day/event content (no hand-edited
   markup per event).
3. **Clickable events** that expand inline to reveal richer details.
4. An **embedded OpenStreetMap** view for events that include coordinates.

The site remains a single static HTML file with no build step and no runtime
dependencies, suitable for GitHub Pages.

## Data model

Event data lives in an inline `<script type="application/json" id="schedule-data">`
block inside `index.html`. The renderer reads it on `DOMContentLoaded`.

### Schema

```json
{
  "days": [
    {
      "date": "2026-06-12",
      "weekday": "Sexta-feira",
      "events": [
        {
          "icon": "🌷",
          "label": "Primeiro",
          "title": "Pegar flores da Gazebo",
          "time": "10:00",
          "description": "Buquê especial reservado com antecedência.",
          "address": "R. Comendador Araújo, 731 - Centro, Curitiba",
          "coords": [-25.4350, -49.2750],
          "image": "images/gazebo.jpg",
          "link": "https://gazebo.com.br"
        }
      ]
    }
  ]
}
```

### Field reference

| Field | Required | Notes |
|-------|----------|-------|
| `days[].date` | yes | ISO `YYYY-MM-DD`. Used for the date strong label and for auto-day-selection. |
| `days[].weekday` | yes | Display string under the date (e.g. `Sexta-feira`). |
| `days[].events[]` | yes | Ordered list; rendering preserves order. |
| `event.icon` | yes | Single emoji or short string shown on the row. |
| `event.label` | yes | Short bold prefix (e.g. `Jantar`, `De Manhã`). |
| `event.title` | yes | Main text (e.g. `Âmbar`). |
| `event.time` | no | Optional time string. Rendered after `title` in parentheses when present. |
| `event.description` | no | Long-form description shown in the expanded panel. |
| `event.address` | no | Human-readable address shown in the expanded panel. |
| `event.coords` | no | `[lat, lng]` numeric pair. Triggers the embedded OSM iframe. |
| `event.image` | no | Path to a photo (e.g. `images/foo.jpg`). Rendered above the description. |
| `event.link` | no | External URL. Rendered as a button/link in the expanded panel (opens in new tab). |

An event is **expandable** when it has at least one of: `description`,
`address`, `coords`, `image`, `link`. Events with none of these render as
plain rows without a chevron and do not respond to clicks.

## Carousel

### Behavior

- Replaces the existing `.timeline` grid with a single-viewport carousel that
  shows one `.day-card` at a time.
- Left arrow (`‹`) and right arrow (`›`) buttons are positioned outside the
  card on desktop and inside the top corners on small screens.
- Below the card, dot indicators (one per day) show position and are
  themselves clickable to jump to a specific day.
- The carousel wraps: from day 3 → arrow right → day 1, and vice-versa.
  *(Wrapping is enabled because the trip is short; user is unlikely to
  mistake direction.)*
- Mobile: horizontal swipe gesture (touchstart / touchend, > 40 px horizontal
  delta wins over vertical) advances to the next/previous day.
- Transition: a 250 ms slide animation between days (CSS `transform:
  translateX`). Single-card width; the off-screen days are positioned
  absolutely to the side.

### Initial day

On load, the renderer picks a starting day using this rule:

1. Compute today's date in local time as `YYYY-MM-DD`.
2. If today matches one of the configured `days[].date`, start on that day.
3. Otherwise, start on the first configured day (`days[0]`).

### Keyboard / a11y

- Arrow buttons are real `<button>` elements with `aria-label="Dia anterior"`
  / `aria-label="Próximo dia"`.
- Left/Right arrow keys, while focus is inside the carousel, also navigate.
- The active day region has `aria-live="polite"` so screen readers announce
  the new day after navigation.

## Expandable events

### Interaction

- Each event row is rendered as a `<button>` (when expandable) or `<div>`
  (when not). The button takes the full row width and is keyboard-focusable.
- Clicking an expandable row toggles an inline accordion panel that drops
  below the row. A chevron icon (`▾` collapsed / `▴` expanded) sits at the
  right edge of the row.
- **Only one event may be expanded per day.** Expanding event B
  auto-collapses event A within the same day. Across days no coordination
  is needed — switching days does not preserve expansion state.
- The expand/collapse animation is a 200 ms height transition. The panel
  uses `max-height` with overflow hidden to avoid layout jumps.

### Expanded-panel layout

When expanded, the panel renders the following sub-sections in this order
(each shown only when its source field is present):

1. **Image** — full-width rounded image (`object-fit: cover`, max-height
   180 px).
2. **Description** — paragraph in `--text` color.
3. **Address** — single line prefixed by `📍`.
4. **Map** — embedded OSM iframe (see next section).
5. **Link** — small pill button labeled `Abrir site ↗`, opens in a new tab
   with `rel="noopener noreferrer"`.

## OpenStreetMap embed

When an event has `coords: [lat, lng]`, the expanded panel includes an
iframe pointing to OSM's official embed endpoint:

```
https://www.openstreetmap.org/export/embed.html?bbox=<minLon>,<minLat>,<maxLon>,<maxLat>&layer=mapnik&marker=<lat>,<lng>
```

The bbox is derived from the coordinate ±0.004° in both axes (≈ a 400 m
window centered on the point). The iframe is:

- Full width of the expanded panel.
- 180 px tall.
- `border: 0`, rounded corners matching the card design (`border-radius:
  14px`).
- Lazy-loaded: the iframe's `src` attribute is **not** set during initial
  render. It is assigned on the first time the user expands that event.
  This guarantees no network requests until needed (`loading="lazy"` alone
  is not reliable for CSS-collapsed elements).
- `referrerpolicy="no-referrer-when-downgrade"` and `title="Mapa do
  local"`.

No JavaScript map library (Leaflet, Mapbox) is used — the OSM iframe is
self-contained and consistent with the no-build-step constraint.

## Rendering pipeline

A single `renderSchedule()` function executes on `DOMContentLoaded`:

1. Parse the JSON from `#schedule-data`.
2. Compute `initialDayIndex` per the auto-selection rule above.
3. For each day, build a `.day-card` DOM tree.
4. Insert all day cards into a `.carousel-track` container; absolutely
   position the carousel for the slide transition.
5. Wire arrow / dot / keyboard / swipe handlers to a `goToDay(index)`
   function that updates `track.style.transform` and the `aria-live`
   region.
6. Wire per-event click handlers that toggle the expansion panel.

The music player and hearts background code remain unchanged.

## CSS additions

Reuses existing tokens (`--bg-1`, `--card`, `--accent`, etc.). New classes:

- `.carousel` — viewport (overflow hidden, position relative).
- `.carousel-track` — flex row that translates horizontally.
- `.carousel-nav` — wrapper for prev/next buttons.
- `.carousel-nav button` — circular pill, glass-style, matches existing
  button language.
- `.day-indicators` — flex row of dots.
- `.day-indicator` / `.day-indicator.is-active` — dot states.
- `.plan-list li.is-expandable` — adds cursor pointer + hover state.
- `.plan-list li .chevron` — chevron icon.
- `.event-expand-panel` — accordion panel.
- `.event-expand-panel.is-open` — open state.
- `.event-image`, `.event-description`, `.event-address`, `.event-map`,
  `.event-link` — section styles inside the panel.

Mobile breakpoints (`max-width: 560px` and `max-width: 430px`) extend the
existing media queries with carousel-specific tweaks (arrows inside the
card, narrower indicators, smaller map height).

## Out of scope

- No build step, bundler, framework, or external JS library.
- No persistence of carousel position or expansion state between visits.
- No editing UI — the JSON is updated by hand.
- No internationalization beyond the existing pt-BR copy.
- No analytics.

## Migration plan for current content

The three hard-coded `.day-card` articles in `index.html` are removed and
replaced by a single `.carousel` element plus the inline JSON. The JSON is
seeded with the existing event copy so the first render reproduces the
current schedule before any new fields are added. Optional fields
(`description`, `coords`, `image`, etc.) can be filled in incrementally
later without code changes.
