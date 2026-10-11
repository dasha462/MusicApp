# Makam Practice Player

A small, offline practice companion for lavta and Turkish makam songs. It plays a synthesized plucked-string melody in 53-EDO while a highlight and time cursor follow a monospaced guide.

## Open it

Download `index.html` and double-click it, or use your browser's **File → Open**. Use a current Chrome, Safari, or Firefox. Press **Play** to enable browser audio. No installation, npm, build, network connection, account, or server is needed. All CSS, JavaScript, and the demo are inside that one file.

The built-in **Hicâz İlâhî · Ey rahmeti bol pâdişâh** is available immediately. Start with the default quarter = 60 BPM and written pitch. A bar of count-in clicks plays before starting; click **Count-in** to turn it off. Pause resumes at the same musical position, re-plucking a held note so it remains audible. Stop returns to the beginning or loop start.

## Practice

- **Click a bar** to play from it. **Shift-click** another bar to select a range, then use **Loop selection** or **L**. **Loop part** selects and starts that part. **Clear loop** resumes the full play order.
- **Tempo** runs from 30% to 150% of the suggested BPM. Changes during playback take effect after the current scheduling horizon (about 120 ms).
- **Transpose** selects whole-tone, fourth, fifth, or octave shifts. **Fine** changes the result by one koma per step. The readout shows the combined shift and resulting A4 frequency. Future note attacks use the new tuning; already ringing strings decay at their original pitch.
- **Grid** adds quarter or eighth markers. Beat dividers, notes, and markers share the exact character grid. `/8` meters use eight columns per denominator beat.
- **Metronome** clicks every denominator beat, accenting the first. Melody and click have independent volume sliders. Count-in is audible even with Metronome off.
- Scrolling manually pauses automatic following for 4.5 seconds. Reduced-motion mode uses a stepped cursor and immediate scrolling. Keep the browser in the foreground while practicing; operating systems can suspend background audio or timers.

## Bring your songs

Give **[SCHEMA.md](SCHEMA.md)** to the person or AI making your transcription. It defines the format and includes the complete demo. Durations use **64th-note ticks**, so each unit in the original 32nd-note transcription equals **two ticks**.

Open **Song library** and choose **Import JSON** or **Paste JSON**, or drop a `.json` file anywhere on the page. Invalid files show a specific error and do not alter the library. Re-importing the same title asks before replacing its imported version. The original built-in demo always remains available.

Songs and per-song tempo, tuning, grid, metronome, and count-in settings are saved in this browser when storage permits. Local-file storage behavior differs between browsers and can change when you move or rename `index.html`. Use **Export library** to make a backup or move browsers, then import that JSON file into the other browser. **Export current song** saves only the selected transcription. Delete uses an in-page confirmation.

If storage is blocked or full, the app stays usable for the current session and displays a backup reminder. Export before closing.

## Keyboard shortcuts

Shortcuts are suspended while typing into controls or while a dialog is open.

| Key | Action |
| --- | --- |
| Space | Play / pause |
| Esc | Stop and return to start (or loop start); dismiss an open dialog |
| ← / → | Previous / next bar in the performance, including repeat occurrences |
| ↑ / ↓ | Tempo ±5 percentage points of the suggested BPM |
| L | Loop the selected range, or the current bar |
| Shift+L | Clear loop |
| M | Metronome on/off |
| C | Count-in on/off |
| S | Cycle Off / Quarters / Eighths |
| ? | Show shortcut help |

## Checks

Click **Run self-test** in the footer, or append `?selftest` to the file URL. The built-in checks cover the exact demo text, 53-EDO pitch values, all 16 part plays, subdivision positions, validation failures, simulated tempo continuity, and loop scheduling boundaries. They report their results in the app.

The synthesis uses cached damped harmonic buffers, with an exact fractional fundamental frequency and a short pluck transient. A compressor limits the master output. A 25 ms lookahead scheduler schedules 120 ms ahead on the Web Audio render clock. Visuals and pause position follow `getOutputTimestamp()`, extrapolated to the current animation frame, so highlights track the sound reaching the output device rather than audio still buffered for playback. The compressor's 6 ms pre-delay is included. Engines without a usable output timestamp fall back to reported base/output latency; accuracy then depends on the browser's latency estimates. Tempo changes add a continuous time mapping at the scheduling horizon.

For development or automated browser checks, a local static server is optional:

```sh
cd /workspace/MusicApp
python -m http.server 8000 --bind 127.0.0.1
```

This is a testing convenience; the app itself does not require a server. No dependencies need installing.

Validation passed in Linux Chromium with Web Audio: 24 built-in self-tests, 43 browser interaction checks, and six output-clock checks covering startup, note boundaries, pause/resume, tempo edits, count-in, completion, and looping with a simulated 200 ms device delay. In a short playback measurement, highlight DOM updates followed estimated audible onsets by 11–13 ms, within one 60 Hz frame. These are browser-clock and DOM observations, not measurements of physical speaker/display output or an audible listening judgment. macOS Safari is unavailable in this environment; direct-file behavior and audible synchronization on the user's device still need a listening check.
