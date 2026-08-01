# Bubble Math

A one-file, touch-first math toy for a young kid who can read numerals. Bubbles carry numbers 1–50 (numeral primary, dot ring grouped in tens as the countable layer, area proportional to value). Combining bubbles adds; pulling them apart subtracts. Every operation is slow, visible, and abortable, with the equation spelled out as it happens.

## Play

| Gesture | Result |
|---|---|
| Drag a bubble onto another, let go | Merge (adds) |
| Push a bubble into another with some pace | They latch and slowly blend — «4 + 6» hovers while it can still be undone; pull away to cancel |
| Yank a bubble fast, keep the finger out | It stretches like taffy, shows «9 − 5», and tears; pull distance sets the proportion; bring the finger back to heal |
| Double-tap | Split in half |
| Two fingers on one bubble, spread | Proportional split with a live cut-line preview |
| Drag a matching bubble into the dashed ghost socket | ⭐ celebration, new target appears elsewhere |
| Merge everything into one bubble, leave it alone | Party, then it bursts into ones (≤12) or tens + ones — the new play set |

Corner controls: 🔊 mute, ⭐ target on/off, + add a bubble. Totals are conserved by every interaction.

## Run

Any static server:

```
python3 -m http.server 4310 --bind 0.0.0.0
```

Open `http://<host>:4310/`. On this machine it's also exposed at
`https://llm.rhino-balance.ts.net:5443/` via `tailscale serve --https=5443`.

iPad: open in Safari, Share → Add to Home Screen for a fullscreen, zoom-proof app.

## Files

- `index.html` — the entire app (canvas rendering, physics, audio via WebAudio, no dependencies)
