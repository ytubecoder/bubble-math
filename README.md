# Bubble Math

![Zero dependencies](https://img.shields.io/badge/dependencies-zero-blue)
![Single file](https://img.shields.io/badge/build-none%2C%20single%20file-blue)
![License: Source Available](https://img.shields.io/badge/license-Source%20Available-yellow)
![Works with Claude Code](https://img.shields.io/badge/works%20with-Claude%20Code-blueviolet)

```
                         _______________________________________________
                        /                                              /|
                       /  BUBBLE MATH                                 / |
                      /    ════════════                              /  |
                     /     (5) + (6)  ·  (9) − (5)  ·  ⭐ target     /   |
                    /_______________________________________________/    |
                    |     ___         ___         ___                |    |
     ___            |   .'   '.    .'   '.     .-'   '-.             |    |
    (・‿・)          |  ( 3     ) →( merge ) → ( split ) ←╮           |  /
     /||\           |   '.___.'    '.___.'     '-.___.-'    ╰──╮     | /
    /  \            |_______________________________________________|/
```

> Drag to merge. Pull apart and hold to tear. Nothing commits without a warning first.

A one-file, touch-first math toy for a young kid who can read numerals. Bubbles carry numbers 1–50 (numeral primary, dot ring grouped in tens as the countable layer, area proportional to value). Combining bubbles adds; pulling them apart subtracts. Every operation is slow, visible, and abortable, with the equation spelled out as it happens.

Built for a 3-year-old. No build step, no dependencies, no backend — one `index.html` with canvas rendering, physics, and WebAudio tones.

## Screenshot

![Bubbles mid-play, showing merge-in-progress equation and the ghost-socket target](docs/screenshot.png)

## Play

**[Play it here](https://ytubecoder.github.io/bubble-math/)**

| Gesture | Result |
|---|---|
| Drag a bubble onto another, let go | Merge (adds) |
| Push a bubble into another with some pace | They latch and slowly blend — «4 + 6» hovers while it can still be undone; pull away to cancel |
| Pull a bubble away from where you grabbed it and hold | It anchors and shows «9 − 5» — grab near the middle for close to half, near the edge for a thin sliver; works at any pull speed, no need to yank; bring the finger back before it commits to call it off |
| Double-tap | Splits proportionally to where you tapped — center is close to half, near the edge is a sliver |
| Two fingers on one bubble, spread | Proportional split with a live cut-line preview |
| Drag a matching bubble into the dashed ghost socket | ⭐ celebration, new target appears elsewhere |
| Merge everything into one bubble, leave it alone | Party, then it bursts into ones (≤12) or tens + ones — the new play set |
| Fill the mat with bubbles (mash +) | Everything bounces off each other for a beat, then pops and deals a fresh starting hand |

Corner controls: 🔊 mute, ⭐ target on/off, + add a bubble. Totals are conserved by every interaction — the mat-full reset is the one deliberate exception.

iPad: open in Safari, Share → Add to Home Screen for a fullscreen, zoom-proof app.

## Run locally

Any static server works:

```bash
python3 -m http.server 4310
```

Open `http://localhost:4310/`.

## Files

- `index.html` — the entire app (canvas rendering, physics, audio via WebAudio, no dependencies)

**Compatible with:** any modern browser · iPad Safari (home-screen app) · [Claude Code](https://claude.ai/code)
