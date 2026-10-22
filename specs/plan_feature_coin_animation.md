# Feature: Falling EUR Coins — Implementation Plan

## Context
`specs/feature_coin_animation.md` asks for 1-EUR coins falling in the page background while a meeting runs. The frequency must match the spending rate: 10 participants × 100 EUR/h = 1000 EUR/h means one coin every 3.6 s. The animation should look organic. The app is the single offline file `app/index.html` (vanilla JS, no external resources), and it must stay that way.

## Approach
Add everything inside `app/index.html`: one `<canvas>`, a little CSS and one self-contained `coinRain` module in the existing script. Coins are drawn procedurally on the canvas, with no image assets.

### Frequency = money spent (exact, drift-free)
- Coins are tied to the cost itself, not to a separate timer. Track `coinsDropped`. On every frame, `due = Math.floor(elapsedMs() * costPerMs)`, and spawn `due - coinsDropped` coins. So **one coin falls for every full euro spent**.
- Because this reuses the existing timestamp-based `elapsedMs()` / `costPerMs`, pausing stops the coins automatically, and the coin count always matches the counter. 1000 EUR/h gives a coin every 3.6 s, as the spec requires.
- Rate 0 means no coins.
- **Guards:**
  - Coming back to a hidden tab can leave a large backlog, so spawn at most ~5 coins per frame and drop the rest of the backlog (set `coinsDropped = due`).
  - Cap the number of live coins at ~200. At very high rates (above ~20 €/s) some coins are skipped visually, and the counter stays the source of truth.

### Organic look
Each coin gets randomized parameters when it spawns:
- **Depth** `z ∈ [0.5, 1]`: scales size (~28–56 px), fall speed and opacity, which gives a parallax effect.
- **Physics:** gravity acceleration up to a per-coin terminal velocity (so coins ease in instead of moving linearly), plus horizontal sway `x += sin(t·freq + phase)·amp`.
- **Tumbling:** 3D flip simulated with `scaleX = cos(flipAngle)`, so the coin narrows to an edge, plus slow rotation.
- **Thickness:** while a coin is turned, a brass edge band (thickness ≈ 1/10 of the diameter) is drawn behind the face, offset by `thickness·sin(flipAngle)`, so the coin reads as a solid disc.
- **Glint:** a diagonal light streak, clipped to the face, sweeps across the coin as `flipAngle` changes.
- **Start:** a random x position, just above the top edge. The coin is removed once it falls below the bottom.

### Coin drawing
A simplified rendering of the 1-EUR common side, drawn procedurally:
- **Metals:** an outer nickel-brass ring and an inner cupronickel core, both shaded with radial gradients. The core diameter is ≈ 0.667 of the coin diameter, as on the real coin.
- **Edge details:** a milled outer edge (140 short ticks), a raised outer rim, and a seam between ring and core.
- **Relief (embossed with a light edge up-left and a shadow down-right):** a large "1" on the left of the core, "EURO" below on the right, six parallel lines running from the core across the ring, and 12 five-pointed stars on the right-hand side of the ring.
- **Not included:** the map of Europe and a national reverse side. Both faces of a tumbling coin show the same design.

To keep it fast, pre-render one coin face per size bucket to an offscreen canvas once, then `drawImage` it with transforms on each frame.

### Integration with the existing code (`app/index.html`)
- **Markup:** `<canvas id="coins" aria-hidden="true">` as the first child of `body`.
- **CSS:** `position: fixed; inset: 0; z-index: -1; pointer-events: none;`, and `body` gets `isolation: isolate` so the canvas sits behind the content. Coins use reduced opacity (~0.5) so the counter stays readable. The setup form already has a solid card background.
- **Canvas size:** set from `devicePixelRatio`, and updated on `resize`.
- **Animation loop:** `coinRain` runs its own rAF loop, and only while coins are visible or the meeting is running. It stops when idle and the screen is empty.
- **Hooks into the existing state functions:**
  - `start()` → `coinRain.reset()`, which sets `coinsDropped = 0`.
  - `tick()` → `coinRain.sync(elapsedMs() * costPerMs)`.
  - **Pause:** no new spawns. Coins already on screen freeze in mid-air until Resume, which matches "the counter freezes".
  - `stop()` → no new spawns, and the remaining coins fall out.
  - `reset()` → clear all coins.
- **Reduced motion:** respect `prefers-reduced-motion: reduce` and turn the animation off entirely.

## Files
- `app/index.html`: canvas, CSS and the `coinRain` module, plus small hooks in `start`, `tick`, `togglePause`, `stop` and `reset`.
- `specs/plan_feature_coin_animation.md`: a copy of this plan, matching the earlier `specs/plan.md`.
- `README.md`: one line mentioning the coin rain.

## Verification
1. Open `app/index.html` offline and check that there are still no network requests.
2. 10 × 100 €/h: a coin about every 3.6 s, so about 16–17 coins after 60 s, matching the counter's whole euros.
3. 1 × 3600 €/h: about one coin per second, and coins appear exactly when the counter crosses a full euro.
4. Pause: no new coins and the on-screen coins freeze. Resume: they continue.
5. Stop: coins finish falling and no new ones appear. New meeting: the canvas clears.
6. High rate (50 × 5000 €/h): stays smooth because of the live-coin cap.
7. Switch away from the tab for 30 s and come back: no burst of hundreds of coins.
8. Text over the coins stays readable, and coins don't block clicks. Check both light and dark mode.
9. With "reduce motion" turned on in macOS, no coins appear.
