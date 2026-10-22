# MeetingCostCounter (MCC) — Implementation Plan

## Context
`specs/base.md` describes a fully client-side web app that shows in real time how much money a meeting costs. The user enters the number of participants and one shared hourly rate. Play starts the meeting, Pause stops and resumes it, and Stop ends it and shows the total. The app must not load anything from the internet. The repo has nothing in it yet except the spec.

## Approach
One self-contained file, **`index.html`**, with inline `<style>` and `<script>`. That means vanilla JS, no build step, no CDN, no web fonts (system font stack only), so it works when opened straight from disk (`file://`) or offline.

### UI
- **Setup form:** "Participants" (integer, min 1) and "Hourly rate per participant" (number, min 0, step 0.01). Inputs are locked while a meeting is running or paused.
- **Huge counter:** the money spent so far, shown with `Intl.NumberFormat` using the browser's locale and currency **EUR** (a constant at the top of the script, easy to change). It takes up most of the viewport (`clamp()`-based font size) and uses tabular numerals so the digits don't jitter.
- **Elapsed time:** a smaller `hh:mm:ss` readout under the counter.
- **Buttons:** ▶ Play, ⏸ Pause / ▶ Resume (one toggle button), ■ Stop, and ↺ New meeting (shown after Stop).
- A responsive layout that also works on a phone or projector, with light and dark themes through `prefers-color-scheme`.

### State machine
`idle → running ⇄ paused → stopped → idle (reset)`
- **idle:** inputs editable. Play is enabled only when the inputs are valid.
- **running:** Pause and Stop enabled.
- **paused:** the counter freezes. Resume and Stop enabled.
- **stopped:** a final total is shown with a "Total meeting cost" label, plus the final duration. "New meeting" goes back to idle and keeps the previous inputs.

### Timing (accurate, drift-free)
- Elapsed time is tracked as `accumulatedMs + (running ? performance.now() - segmentStart : 0)`. Pause adds the current segment to `accumulatedMs`, and Resume sets a new `segmentStart`. Elapsed time is never built by counting ticks.
- Cost = `participants × rate × elapsedMs / 3_600_000`.
- The display updates through `requestAnimationFrame` while running. When the tab is hidden, rAF pauses, but the cost is still correct as soon as the tab is visible again because it's computed from timestamps.
- Optional nicety: `document.title` shows the current cost so it's visible from the tab bar.

## Files
- `index.html`: the whole app (new).
- `README.md`: a short note on how to use it: open `index.html` in a browser (new, optional).

## Verification
1. Open `index.html` directly in a browser. Check in the DevTools Network tab that it makes no external requests, and that it still works with the network turned off.
2. Math check: 6 participants × 100 €/h should give ≈ 10.00 € after 60 s, and 1 × 3600 €/h should add ≈ 1 € per second.
3. Pause for 10 s, then resume: the counter stays frozen while paused and picks up where it left off without jumping.
4. Stop: the final total stays fixed, and New meeting resets the counter to 0 while keeping the inputs.
5. Invalid input (0 or empty participants, negative rate): Play is disabled.
6. Switch tabs for 30 s while running, then come back: the counter has the correct value.
