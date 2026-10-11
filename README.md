# Makam Practice Player

A quiet, offline practice companion for lavta and Turkish makam songs. It plays a synthesized plucked-string melody in 53-EDO while a highlight and time cursor follow a monospaced guide.

## Open it

Download `index.html` and double-click it, or use your browser's **File → Open**. Use a current Chrome, Safari, or Firefox. Press **Play** to enable browser audio. No installation, npm, build, network connection, account, or server is needed. All CSS, JavaScript, and the demo are inside that one file.

The built-in **Hicâz İlâhî · Ey rahmeti bol pâdişâh** is available immediately. Start with the default quarter = 60 BPM and written pitch. A bar of count-in clicks plays before starting; click **Count-in** to turn it off. Pause resumes at the same musical position, re-plucking a held note so it remains audible. Stop returns to the beginning or loop start.

## Practice

- **Click a bar** to play from it. **Shift-click** another bar to select a range, then use **Loop selection** or **L**. **Loop part** selects and starts that part. **Clear loop** resumes the full play order.
- **Tempo** runs from 30% to 150% of the suggested BPM. Changes during playback take effect after the current scheduling horizon (about 120 ms).
- **Sound → Transpose** selects whole-tone, fourth, fifth, or octave shifts. **Fine** changes the result by one koma per step. The readout shows the combined shift and resulting A4 frequency. Future note attacks use the new tuning; already ringing strings decay at their original pitch.
- **Display → Subdivision** offers quarters, eighths, 16ths, and 32nds. The default quiet ruler keeps small quarter-beat numbers visible and reveals fine ticks only on the playing bar. Choose **Counting** for `1 e & a` or `1 · e · & · a ·`, and **All bars** to show the detail throughout. Beat dividers, notes, and markers share the same time grid. `/8` meters use eight columns per denominator beat.
- **Display → Note names** offers **Do re mi** (default), **Letters**, or **Both** with either system dominant. Secondary names are smaller and neutral grey. Fixed-do uses C = do, D = re, E = mi, F = fa, G = sol, A = la, B = si. Accidentals stay attached (C♯ / do♯). This changes labels only; tuning stays the same.
- **Display → Note size** offers whole bars or **Larger · split bars**, which wraps bars at beat boundaries. Every note has the same font size, including dense passages. Narrow windows also wrap bars to preserve readability.
- **Metronome** clicks every denominator beat, accenting the first. Melody and click have independent volume sliders in **Sound**. Count-in is audible even with Metronome off.
- Scrolling manually pauses automatic following for 4.5 seconds. Reduced-motion mode uses a stepped cursor and immediate scrolling. Keep the browser in the foreground while practicing; operating systems can suspend background audio or timers.

## Insert notes

Use **Practice** for playback and **Edit** for note insertion. Edit reveals the Snap selector and Undo, and shows fine ticks on every bar. Choose **16ths** or **32nds** in **Snap**. A dashed marker follows the nearest grid onset. Click the position where the note belongs; the dialog shows its bar, beat, and subdivision step. Choose the letter, octave, Turkish accidental, and duration, then **Insert and save**. You can also focus a bar and press Enter to choose its position in the dialog.

Insertion occupies the chosen interval. It splits or replaces existing notes/rests there while preserving the bar length and all later onsets. For example, inserting a 32nd halfway through a held quarter leaves the first half, the new note, and the remaining tail. A surviving tail re-attacks when it resumes. A duration cannot cross the barline. If a very short leftover cannot fit its two-character label, validation rejects the edit; choose a different position or duration.

Edits save to your browser library and are included in song/library exports. Editing the built-in sample creates an **(edited)** copy. **Undo** reverses up to 50 insertions per song during the current page session; a reload, re-import, or deletion clears that song's undo history. **Practice** returns grid clicks to playback and hides editing controls. Entering Edit with subdivision Off or Quarters selects 32nds. Each fresh page starts in Practice mode.

## Bring your songs

Give **[SCHEMA.md](SCHEMA.md)** to the person or AI making your transcription. It defines the format and includes the complete demo. Durations use **64th-note ticks**, so each unit in the original 32nd-note transcription equals **two ticks**.

Open **Song library** and choose **Import JSON** or **Paste JSON**, or drop a `.json` file anywhere on the page. Invalid files show a specific error and do not alter the library. Re-importing the same title asks before replacing its imported version. The original built-in demo always remains available.

Songs and per-song tempo, tuning, subdivision, name display, ruler/counting style, detail scope, note size, metronome, and count-in settings are saved in this browser when storage permits. Local-file storage behavior differs between browsers and can change when you move or rename `index.html`. Use **Export library** to make a backup or move browsers, then import that JSON file into the other browser. **Export current song** saves only the selected transcription. Delete uses an in-page confirmation.

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
| S | Cycle Off / Quarters / Eighths / 16ths / 32nds (Off is skipped while inserting) |
| ? | Show shortcut help |

## Checks

Click **Run self-test** in the footer, or append `?selftest` to the file URL. The built-in checks cover the exact demo text, 53-EDO pitch values, all 16 part plays, subdivision positions, validation failures, simulated tempo continuity, and loop scheduling boundaries. They report their results in the app.

The synthesis uses cached damped harmonic buffers, with an exact fractional fundamental frequency and a short pluck transient. A compressor limits the master output. A 25 ms lookahead scheduler schedules 120 ms ahead on the Web Audio clock; requestAnimationFrame reads that same clock for visuals. Tempo changes add a continuous time mapping at the scheduling horizon.

For development or automated browser checks, a local static server is optional:

```sh
cd /workspace/MusicApp
python -m http.server 8000 --bind 127.0.0.1
```

This is a testing convenience; the app itself does not require a server. No dependencies need installing.

Validation passed in Linux Chromium with Web Audio: 32 built-in self-tests, 43 browser regression checks, and 37 redesigned-view checks. The latter cover consistent note sizing, ruler alignment, fine-detail visibility, wrapped-line insertion, Undo, saved preferences, playback during view changes, light/dark themes, and narrow windows. The original player was also validated with complete performances at 50%, 100%, and 150% tempo. Each performance scheduled all 376 melody attacks and 128 clicks across all 32 bar visits, with no detected highlight/cursor mismatch. These are scheduling and DOM observations, not an audible listening judgment. This environment's managed browser blocks navigation to `file://`, and it has no macOS Safari. Direct-file behavior and audible tone quality in Mac Chrome, Safari, and Firefox therefore still need a device check. The deterministic self-tests do not replace that listening test.
