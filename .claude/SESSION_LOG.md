# Session Log — Bubble Math

## 2026-08-03 — Grab-position proportional tearing + overcrowding reset

### Summary
- Replaced the `TEAR_STICKINESS` elastic-spring hack with an anchor-and-hold model: the dragged body follows the finger normally (feels attached, used for carrying to a merge), then freezes in place once pulled past a threshold distance *and* genuinely stopped moving — works at any drag speed instead of requiring a fast yank, and doesn't drift back together while held.
- Split proportion (one-finger tear and double-tap) is now decided by where the bubble was grabbed/tapped relative to its center (`circleSplitFraction`, the same geometry the two-finger split already used) — center ≈ 50/50, rim ≈ a thin sliver — locked in at grab time, not recomputed from pull distance.
- Added a `chaos` overcrowding reset: past `MAX_BUBBLES` (16), everything gets a random velocity kick, bounces for ~2.2s (drags/bonds/ghost-fill suspended), then pops and respawns the starting hand with a new target — the antidote to mashing the + button.
- Verified all three end-to-end via synthetic PointerEvents in a live headless browser (temporary `window.__debug` hook, removed before finishing) — slow/fast pulls, cancel-by-pulling-back, rim vs. center grabs, double-tap proportions, and the full chaos trigger→bounce→reset cycle.

### Lessons Learned
- **Rejected:** gating the tear-anchor purely on live gap-to-body (what `TEAR_STICKINESS` effectively measured). Proven mathematically and empirically: for any constant drag velocity the gap settles to a steady state (`speed/followRate`) that a slow, sustained pull can never cross — no amount of total distance helps. This was the actual root cause of the user's "slow pull rejoins" complaint, not just insufficient stickiness.
- **Rejected:** gating on distance-from-touchdown alone (no speed/settle check). Any normal cross-screen carry travels far past a bubble-radius-scaled threshold almost immediately, so this misfires on ordinary drag-to-merge.
- **Accepted:** gate anchoring on TWO conditions together — pulled distance from the touchdown point past `SPLIT_TRIGGER_RATIO*r`, AND the finger has genuinely stopped (debounced velocity check, not instantaneous) — a "pull it out and hold" gesture that's speed-independent for the pull but still can't misfire during a normal continuous carry (which never truly stops until release).
- **Gotcha:** a single frame's computed finger velocity is noisy — browser event delivery isn't perfectly even, so one quiet frame during an otherwise-fast, continuous drag can read as "stopped." Required accumulating a short dwell (`SETTLE_DWELL`, ~0.12s) below the speed threshold before trusting "settled"; without it, fast carries occasionally misfired as tears.
- **Gotcha:** grabbing near a bubble's rim (not moving at all) already sits a good distance from center — using raw grab-offset as part of the anchor trigger meant merely touching the outer ~45% of a bubble's radius and holding still could auto-split it with zero pulling. Fixed by measuring "pulled" distance from the touchdown point, not from center.
- **Gotcha:** synthetic `pointermove` dispatched slower than the game's ~60fps loop (e.g. one event per 40-60ms) leaves several update() frames per event where `fingerX` hasn't changed, which read as zero velocity even mid-"fast" test drag — an artifact of the test harness, not the game. Dispatching at ≤16ms intervals fixed it.
- **Gotcha:** when a tracked bubble's position appeared "stuck," the actual cause was that it had already split — the original id becomes the *remainder* (reduced value, recoil velocity) while a new id (the "bud") took over following the finger. Chasing this as a movement bug wasted time before tracing decision variables (not just position) revealed the split had already happened.

### Decisions
- Both the one-finger tear and double-tap now share proportional-split geometry with the existing two-finger split, rather than each having its own ad hoc rule — one mental model for "where you touch decides how much comes off" across every split gesture.
- Overcrowding reset intentionally breaks sum-conservation (the project's core invariant) — documented as the one deliberate exception, since it's a full restart rather than a merge/tear/split interaction.
- User feedback mid-session: after one round of clarifying questions, further confirm-the-synthesized-design questions were unwanted — "I made my comments, please make the changes as I've asked." Proceeded straight to implementation and used engineering judgment on unresolved details (e.g. picking MAX_BUBBLES=16, choosing the bounce-then-reset behavior over the alternative "push together" option) rather than asking again.

## 2026-08-01 — Built the app: sandbox → one-handed physics → game elements

### Summary
- Built the whole app in one session, three passes: (1) initial sandbox (drag-merge, double-tap halve, two-finger proportional split, target pill, WebAudio tones, iPad-hardened shell), (2) one-handed momentum physics (elastic finger-follow, momentum bonding with slow abortable blend, one-finger taffy tear), (3) game elements (live + floating equations, ghost-socket target on the mat, last-bubble finale bursting into ones / tens+ones).
- Served on 0.0.0.0:4310 (python http.server) behind `tailscale serve --https=5443`; verified all interaction paths end-to-end in headless Playwright with a temporary in-page debug hook (added for tests, removed after).

### Lessons Learned
- **Rejected:** hard speed-cap chase for the dragged bubble — on any long drag the body never catches up, so a fast carry is indistinguishable from a deliberate pull-apart; every cross-screen merge attempt read as a tear. Replaced with exponential spring follow + `TEAR_STICKINESS` once necked.
- **Rejected:** representing a dragged bubble by finger position *and finger velocity* for bond scanning — broke the "carry briskly then hold still over target" case (finger velocity drops to 0). Final model: finger position for proximity, body's real capped velocity for momentum.
- **Gotcha:** `closingSpeed` had a sign error, so momentum bonds never formed — earlier "successful merges" were all silently taking the drop-merge path. Symptom-masking by a parallel code path made this hard to spot; caught only by testing the specific mechanism.
- **Gotcha:** bond break test used relative velocity, but ambient drift on the non-dragged partner kept tripping it — bonds dissolved mid-blend. Fix: distance-only break test + freeze ambient wander on bonded bubbles (the bond drives position).
- **Gotcha:** finale burst pieces momentum-bonded back together within a second (spontaneous "10 + 10 = 20") — the fresh play set ate itself. Fix: `bondCooldown: 2.5` on finale spawns; deliberate drop-merge still works during cooldown.
- **Gotcha:** tear preview («4 − 2») rendered simultaneously with the blend equation («4 + 6») because a bonded drag still has body-to-finger stretch. Fix: suppress drag effects while bonded.
- **Gotcha:** Playwright MCP screenshot round-trip is slower than the game's animations (merge→finale→burst completed between tool calls, twice). Fix: capture frames in-page via `canvas.toDataURL()` returned through evaluate's `filename` param, base64-decode locally.
- **Gotcha:** the pinned Playwright profile persists localStorage — `bubblemath.target=0` from an earlier UI test made the ghost socket look broken a phase later. Check stored state before diagnosing "feature doesn't fire".
- **Accepted:** temporary `window.__bm` debug hook (list/sum/reset) for deterministic in-page testing of arithmetic invariants — much stronger than screenshot-eyeballing; removed before shipping each time.

### Decisions
- Numeral is primary (kid reads numbers); dot ring grouped in tens is the countable secondary layer; area ∝ value. Range 1–50, target range 2–20.
- User correction mid-build: the kid had already learned the original gestures — new physics must be **additive**, not a replacement. All legacy paths (drop-merge, double-tap, two-finger split) restored and kept.
- User correction: draw order by size (small on top) so small bubbles don't vanish under big ones; hitTest matched to it (smallest wins).
- Slow + abortable as the core interaction principle: blend ~1.1s, tear-hold ~0.7s, both cancellable mid-way; nothing commits without a visible progress ring.
- Target = ghost socket on the mat (size + numeral matching, requires delivery); celebration only on bringing a matching bubble into it, not on mere existence.
- Finale (last bubble standing): party → burst into ones (≤12) or tens+ones (>12) as the new set — teaches place value and restarts play without a game-over.
