# MeetingCostCounter (MCC)

Shows in real time how much a meeting costs. It runs entirely in the browser, needs no install, and makes no network requests.

## Usage

1. Open `app/index.html` in any modern browser (double-click it; no server needed).
2. Enter the number of participants and the hourly rate per participant.
3. **▶ Play** starts the meeting, **⏸ Pause / ▶ Resume** interrupts it and continues it, and **■ Stop** ends it and shows the total.
4. **↺ New meeting** resets the counter and keeps your inputs.

While the meeting runs, a 1-EUR coin falls in the background for every euro spent (turned off when the system asks for reduced motion).

The currency is set by the `CURRENCY` constant in `app/index.html` (default `EUR`).

## Project layout

```
app/index.html   the complete app (HTML, CSS and JS in one file)
specs/           feature specifications and their implementation plans
```

See `specs/base.md` for the base specification.
