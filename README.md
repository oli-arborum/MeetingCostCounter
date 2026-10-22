# MeetingCostCounter (MCC)

Shows in real time how much a meeting costs. It runs entirely in the browser, needs no install, and makes no network requests.

## Usage

1. Open `index.html` in any modern browser (double-click it; no server needed).
2. Enter the number of participants and the hourly rate per participant.
3. **▶ Play** starts the meeting, **⏸ Pause / ▶ Resume** interrupts it and continues it, and **■ Stop** ends it and shows the total.
4. **↺ New meeting** resets the counter and keeps your inputs.

The currency is set by the `CURRENCY` constant in `index.html` (default `EUR`).

See `specs/base.md` for the specification.
