# Bubble Math

Single-file kids' math toy (`index.html` — no deps, no build) for a 3-year-old who reads numerals. Canvas bubbles carry values 1–50; merging adds, tearing/splitting subtracts. Everything is deliberately slow and abortable (~1.1s blend, ~0.7s tear) so a small child can change his mind mid-gesture.

## Serving (machine state, not derivable from code)

**Dev** (local iteration/testing, not connected to git):
- Backing server: `python3 -m http.server 4310 --bind 0.0.0.0` run from this dir as a plain background process — does **not** survive reboot; restart it if the page 404s.
- Tailnet: `tailscale serve --bg --https=5443 http://127.0.0.1:4310` → https://llm.rhino-balance.ts.net:5443/ (serve config persists across reboots). Always relay the link as `https://` — ts.net is HSTS-preloaded.

**Prod** (public, playable): repo is `ytubecoder/bubble-math` on GitHub (public — GitHub Pages on private repos needs a paid plan, which this account doesn't have). GitHub Pages serves straight from the `main` branch root (`.nojekyll` present, no build step). `git push origin main` **is** the prod deploy — anything pushed there goes live within ~1-2 min at:

https://ytubecoder.github.io/bubble-math/

There is no separate `dev` branch — local serving above is the pre-push testing step; `main` is both the working branch and what's live.

## Design invariants

- **Sum conservation.** Every interaction — merge, tear, double-tap, two-finger split, finale burst — conserves total value. Only the + button and the 25s trickle add value. A change that breaks this is wrong.
- **Additive interactions only.** The kid has already learned drop-to-merge, double-tap halve, and the two-finger split. New physics (momentum bonding, one-finger tear) are layered on top; never remove the old paths.
- **Size layering.** Draw order is biggest-at-back / smallest-on-top, and `hitTest` returns the smallest bubble under the finger — so small fingers grab the small bubble sitting on a big one. Keep draw order and hit order in sync.
- **Spring follow, not speed cap.** The dragged body chases the finger with an exponential spring (`followRate`). A hard speed cap was tried and rejected: it never converges on a long drag, so every fast carry across the screen reads as a deliberate pull-apart. Once a neck forms, `TEAR_STICKINESS` nearly stops the chase so the stretch holds without sustained fast pulling.
- **Bond immunity on finale spawns.** Finale pieces get `bondCooldown: 2.5` — without it the burst pieces momentum-bond back together within a second (10+10=20 happened spontaneously) and the finale can loop.
- **One story at a time.** Drag/tear effects are suppressed while the dragged bubble is in a bond (otherwise "4 + 6" and "4 − 2" render simultaneously).

## Testing gotchas (all hit for real, 2026-08-01)

- Playwright MCP screenshot round-trips are slower than the game's animations — merge→finale→burst completes between tool calls. Capture mid-animation frames from *inside* the page: return `canvas.toDataURL('image/png')` from `browser_evaluate` (use `filename` param), then base64-decode.
- Synthetic PointerEvents are untrusted: `setPointerCapture` throws on them (already try/catch-wrapped in the code). Drive the game by dispatching PointerEvents on the canvas via evaluate.
- The pinned Playwright profile persists `localStorage` across sessions: `bubblemath.target` / `bubblemath.muted` from an old test can silently disable features (a "broken" ghost socket was just the target toggled off weeks-old-state). Check localStorage before diagnosing.
