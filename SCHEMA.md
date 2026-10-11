# Makam Practice Player — song format, version 1

This is a precise authoring specification for a **monophonic Turkish makam practice song**. Save UTF-8 JSON, without comments or Markdown fences, in a `.json` file. Import does not read scores or infer key signatures, ornaments, or repeats. The optional grid editor on this feature branch can insert notes and export the same version-1 format. Encode every sounding pitch, rest, duration, and performance repeat explicitly.

The player is the self-contained `index.html` beside this file. Open it, choose **Song library → Import JSON**, or paste/drop JSON. **Export current song** provides another authoring example.

## Fundamental rules

- Root `format` is exactly `"makam-song"`; `version` is the integer `1`.
- One duration tick is **one 64th note**. A quarter note is **16 ticks**. The supplied transcription's 32nd-note units must therefore be **multiplied by two**. This finer integer unit supports dotted 32nds without fractions.
- Scientific pitch octaves change at **C**: B4 is below C5. A4 is 440 Hz. No Western equal-tempered accidentals are inferred.
- Every accidental is local to its event. **Repeat key-signature accidentals on every affected note.** Omitting `accidental` means natural, even after an altered note or in an altered key signature.
- Every bar must contain exactly `numerator × 64 / denominator` ticks. Include explicit rests for silence. No pickup/short bars or meter changes in version 1; pad a pickup with rests, or prepare a separate song for a different meter.
- Bars are displayed once in `bars` array order. Parts partition that order exactly once. `playOrder` is the fully resolved performance; it may omit a part or repeat it any number of times within the limits below.
- Unknown extra properties are ignored in version 1. Do not rely on them to affect playback. Required fields must be present with the documented types; numbers must be JSON numbers, not strings.

## Root fields

| Field | Required | Type and rules |
| --- | --- | --- |
| `format` | yes | String, exactly `"makam-song"`. |
| `version` | yes | Integer, exactly `1`. |
| `metadata` | yes | Object described below. |
| `bars` | yes | Array of 1–2,000 bar objects; at most 100,000 total events. Array order is display and selection order. |
| `parts` | yes | Array of 1–2,000 part objects, in display order. |
| `playOrder` | yes | Array of 1–10,000 entries; expanded performance at most 100,000 bar visits. |

One imported file is at most 10 MiB. A browser library holds at most 200 imported songs. Titles identify imported songs: a later import with the **exact same title** asks before replacing the existing imported version. Titles are case-sensitive; no whitespace normalization is performed. The built-in demo is always available separately, even if an imported version uses its title. Settings are keyed by title, so such versions share settings.

## Metadata

| Field | Required | Type and rules |
| --- | --- | --- |
| `title` | yes | Nonempty string, at most 200 UTF-16 code units. Give the song an identifying title even if its original title is unknown. |
| `firstLyricLine` | no | Nonempty string or `null` when unknown. |
| `makam` | no | Nonempty string or `null`. Display metadata, not a tuning instruction. |
| `usul` | no | Nonempty string or `null`. Display metadata; the meter determines beat spacing and click positions. |
| `timeSignature` | yes | Object with integer `numerator` 1–64 and integer `denominator` in `2, 4, 8, 16, 32`. Applies to every bar. |
| `composer` | no | Nonempty string or `null`. |
| `lyricist` | no | Nonempty string or `null`. |
| `tempo` | yes | Object with finite numeric `bpm` from 10 to 400 and integer `beatUnit` in `2, 4, 8, 16, 32`. For quarter = 60, use `{"bpm":60,"beatUnit":4}`. |
| `notes` | no | Array of at most 50 nonempty strings, each at most 2,000 code units. Free text about uncertainties, interpretation, or performance. Shown below the guide. |
| `flags` | no | Array of at most 50 nonempty strings, each at most 2,000 code units. Examples: `"estimated-tempo"`, `"inferred-play-order"`. Shown below the guide. |

Unless otherwise limited above, metadata strings have a maximum of 500 UTF-16 code units. All nonempty strings must contain non-whitespace text and must not contain control characters U+0000–U+001F or U+007F. Empty optional strings are invalid: omit the property or use `null` for nullable fields. `notes` and `flags` must be arrays when present, not `null`.

Tempo's beat unit and the meter denominator are independent. Seconds per tick are `60 / (bpm × (64 / beatUnit))`. For 6/8 with quarter = 60, an eighth-note click occurs every 0.5 seconds. The player clicks each **denominator beat**, accents the first, and counts in one complete bar. Version 1 does not encode dotted tempo units, usul-specific accents, or compound beat grouping; convert dotted-quarter = 60 to eighth = 180.

## Bars and events

A bar has these required fields:

| Field | Type and rules |
| --- | --- |
| `id` | Unique positive integer. Displayed as its bar number and referenced by parts. IDs need not be consecutive; their numeric value does not change display order. |
| `events` | Nonempty array of note/rest events, in chronological order. Events have no explicit onset: sum all preceding durations in the bar. No chords or overlapping voices. |

Every event has `type` (`"note"` or `"rest"`) and `duration` (positive integer ticks). For a rest, use only these two fields:

```json
{"type":"rest","duration":8}
```

This is an eighth-note rest. It displays `R`. A rest schedules no pluck; the natural decay of the previous pluck may still be audible.

A note has these fields:

| Field | Required | Type and rules |
| --- | --- | --- |
| `type` | yes | Exactly `"note"`. |
| `duration` | yes | Positive integer ticks; must fit the bar total. |
| `letter` | yes | One uppercase letter, `A` through `G`. |
| `octave` | yes | Integer 1–7, in scientific pitch notation. |
| `label` | yes | The same uppercase letter, optionally followed by **one** of `♯`, `♭`, `↓`, `↑`. One or two characters, no octave digit or spaces. Examples: `A`, `B↓`, `C♯`. |
| `accidental` | no | One of the eight exact string values below. Omit for natural. `null`, `0`, `"natural"`, and Western accidental names are invalid. |
| `turkishName` | no | Nonempty string, at most 100 code units, or `null`. Examples: `dügâh`, `dik kürdî`, `nîm hicâz`, `eviç`. |

A label is presentation only: **`accidental` controls pitch and color**. Author consistent labels, but do not expect `♯` in a label to change a frequency. The key distinguishes altered pitches by letter plus accidental, independent of octave. All octaves of the same alteration share a color. A six-color calm palette cycles if a song contains more than six distinct alterations.

To prevent collisions with the next event or a beat divider, a label's character count must not exceed either its duration or the number of ticks remaining before the next denominator-beat boundary. An ordinary 32nd note has two ticks and can display a two-character label. A one-tick 64th note can display only a one-character label. A two-character altered label cannot begin on the final tick before a beat divider; express that unusual notation with a one-character label and the explicit accidental.

### Durations

| Value | Ticks | Dotted ticks |
| --- | ---: | ---: |
| Whole | 64 | 96 |
| Half | 32 | 48 |
| Quarter | 16 | 24 |
| Eighth | 8 | 12 |
| Sixteenth | 4 | 6 |
| Thirty-second | 2 | 3 |
| Sixty-fourth | 1 | Not an integer; unsupported |

Other positive integer durations are valid. For instance a held dotted quarter is 24 ticks. This integer grid cannot represent triplets exactly. Ties are encoded as one long event within a bar. A note repeated in successive events, or across a barline, re-attacks. There is no tie-across-bars field in version 1. A note may cross beat boundaries within its bar; no extra label is inserted at those boundaries.

### AEU accidentals

One koma is 1/53 of an octave, approximately 22.641509 cents. Use ASCII identifiers exactly as listed; Turkish names appear in the UI.

| `accidental` | Koma offset | Display name |
| --- | ---: | --- |
| `koma-flat` | −1 | koma flat |
| `koma-sharp` | +1 | koma sharp |
| `bakiye-flat` | −4 | bakiye flat |
| `bakiye-sharp` | +4 | bakiye sharp |
| `kucuk-mucenneb-flat` | −5 | küçük mücenneb flat |
| `kucuk-mucenneb-sharp` | +5 | küçük mücenneb sharp |
| `buyuk-mucenneb-flat` | −8 | büyük mücenneb flat |
| `buyuk-mucenneb-sharp` | +8 | büyük mücenneb sharp |

For example:

```json
{"type":"note","letter":"B","octave":4,"label":"B↓","accidental":"bakiye-flat","turkishName":"dik kürdî","duration":8}
```

### Pitch computation

The unaltered ascending sequence starting at A4 is A4 = 0, B4 = 9, C5 = 13, D5 = 22, E5 = 31, F5 = 35, G5 = 44 koma relative to A4. Because scientific octaves change at C, the equivalent C-based offsets within an octave are:

```text
C = 0, D = 9, E = 18, F = 22, G = 31, A = 40, B = 49
komaFromA4 = 53 × (octave − 4) + offset[letter] − 40 + accidentalKoma
frequencyHz = 440 × 2 ^ ((komaFromA4 + transposeKoma) / 53)
```

With no transpose: A4 = 440 Hz, B↓4 = 469.7337150 Hz, C♯5 = 549.5528539 Hz, D5 = 586.6897810 Hz. Named transpose shifts are 0, ±9, ±22, ±31, ±53 koma, plus an independent integer fine adjustment from −53 to +53 koma in steps of one. These settings belong to the player, not to the song's written pitches.

## Parts

Each part is an object:

| Field | Required | Type and rules |
| --- | --- | --- |
| `id` | yes | Unique nonempty string, at most 80 code units, no control characters. Used by `playOrder`. |
| `title` | yes | Nonempty string, at most 500 code units, no control characters. The **entire heading**, reproduced verbatim above the bars. Include any `PART 1:`, bar range, or `(repeat)` annotations yourself. Two spaces are preserved. |
| `bars` | yes | Nonempty array of existing bar IDs, with no duplicates. |

Concatenating the `bars` arrays of all parts must equal the root `bars` IDs in order, with every bar included exactly once. Display repeats by using `playOrder`, not by placing the same bar in two parts. If separate printed bars happen to have identical notes (demo bars 3 and 5), give each its own bar ID and event list.

## Play order

Each entry is an object:

| Field | Required | Type and rules |
| --- | --- | --- |
| `part` | yes | String referencing an existing part's `id`. |
| `label` | no | Nonempty string, at most 500 code units, no control characters. Use a verse/lyric name for the position readout. Omit if unknown; `null` is invalid. |

Each entry performs the entire referenced part once. Flatten all repeats, verses, D.S., segno, and endings into this array beforehand. No music-symbol interpretation occurs at runtime. Consecutive entries with the same part **and label** are a repeat group; the player derives `pass 1 of 2`, `pass 2 of 2`, etc. A different part or label starts a new group. Text headings remain in one fixed place; the highlight jumps back when a part repeats.

A loop uses an inclusive contiguous range in **displayed bar order**, ignoring `playOrder`. For example selecting bars 2–3 repeats bar 2 then bar 3, including the part boundary.

## Text rendering contract

- One tick occupies one monospace character column: 16 columns per quarter, 8 per eighth, 2 per 32nd.
- Every denominator beat is followed by `|`. Add one more `|` at the end of the bar. Thus a 4/4 bar is `4 × (16 + 1) + 1 = 69` characters.
- The first character of a label sits at its onset tick, plus one extra column for each preceding beat divider. Remaining columns contain spaces. Rests show `R` at their onset.
- For long meters, find the fewest equal-length lines that partition the numerator into whole denominator beats and keep lines at most 72 characters. Every line ends after a beat divider; only the last line gets the extra `|`. A prime-numbered long meter may need one beat per line to keep equal lengths.
- A note continuing across a visual line break is not relabeled. Its onset label remains highlighted, while the cursor moves to the continuation line.
- The player can display fixed-do names (C/do, D/re, E/mi, F/fa, G/sol, A/la, B/si) below the stored letter labels, or make fixed-do dominant. Names are derived from `letter` plus the display accidental suffix; they are not added to song events and do not affect frequencies or onset columns.
- 16ths mode places `1 e & a` at four positions per quarter; 32nds places `1 · e · & · a ·` at eight positions. These are view/editor settings; event duration ticks remain unchanged.
- Quarters mode marks actual quarter-note positions `1 2 3 …`; Eighths mode marks those positions plus `&` halfway between. These remain true quarter/eighth positions in `/8` meters; denominator beat dividers and metronome clicks still occur every eighth.
- Only notes with an explicit accidental are colored. Natural notes use the main text color; rest labels and beat dividers are muted. The active note/rest uses the highlight box.

## Library backup format

**Export library** creates a separate envelope; it is not a song:

```json
{
  "format": "makam-library",
  "version": 1,
  "songs": [],
  "settings": {
    "An exact song title": {
      "tempoPercent": 100,
      "transpose": 0,
      "fine": 0,
      "subdivision": "off",
      "noteNames": "western",
      "metronome": false,
      "countIn": true
    }
  }
}
```

- `format`, `version`, and `songs` are required. `songs` is an array of 0–200 complete valid songs; duplicate titles within one backup are rejected.
- `settings` is an optional object keyed by exact song title. Tempo percent is clamped to 30–150; transpose must be one of the named shifts above; integer fine is clamped to −53…+53; subdivision is `off`, `quarters`, `eighths`, `sixteenths`, or `thirtyseconds`; `noteNames` is `western` (letters dominant, default) or `solfege` (fixed-do dominant); toggles are booleans. Invalid/missing setting fields use defaults shown above.
- Import merges the songs into the current library, asking before same-title replacement. A cancelled replacement changes nothing. All songs are validated **before any change**. Existing songs not mentioned are retained.
- Settings for imported song titles are restored. The built-in sample's settings can also be restored even when the backup contains no songs. The built-in sample itself is not automatically included in exports; it is already available in every copy of the app.
- Export does not include transient selection, loop, playhead, or volume positions. Those controls reset on a fresh page.
- If browser storage is unavailable or full, import/playback work in memory and a visible message recommends exporting a backup.

## Complete worked example

This is the exact built-in demo. Its 16 play-order entries perform 32 bar visits. **All durations below have already been converted to 64th-note ticks.** Do not multiply them again.

```json
{
  "format": "makam-song",
  "version": 1,
  "metadata": {
    "title": "Hicâz İlâhî · Ey rahmeti bol pâdişâh",
    "firstLyricLine": "Ey rahmeti bol pâdişâh",
    "makam": "Hicaz",
    "usul": "Sofyan",
    "timeSignature": {
      "numerator": 4,
      "denominator": 4
    },
    "composer": null,
    "lyricist": "Kuddûsî",
    "tempo": {
      "bpm": 60,
      "beatUnit": 4
    },
    "notes": [
      "Tempo is an estimate; no tempo is written on the score.",
      "Play order is the best reading of a score with no Fine mark.",
      "B↓ = bakiye flat (−4 koma); C♯ = bakiye sharp (+4 koma). F♯ in bar 1 is eviç; all other F notes are natural (acem)."
    ],
    "flags": [
      "estimated-tempo",
      "inferred-play-order"
    ]
  },
  "bars": [
    {
      "id": 1,
      "events": [
        {
          "type": "note",
          "letter": "A",
          "octave": 4,
          "label": "A",
          "duration": 8,
          "turkishName": "dügâh"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 8,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 8,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 8,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 4,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "E",
          "octave": 5,
          "label": "E",
          "duration": 4,
          "turkishName": "hüseynî"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 4,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 4,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 8,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "E",
          "octave": 5,
          "label": "E",
          "duration": 4,
          "turkishName": "hüseynî"
        },
        {
          "type": "note",
          "letter": "F",
          "octave": 5,
          "label": "F♯",
          "duration": 4,
          "turkishName": "eviç",
          "accidental": "bakiye-sharp"
        }
      ]
    },
    {
      "id": 2,
      "events": [
        {
          "type": "note",
          "letter": "G",
          "octave": 5,
          "label": "G",
          "duration": 8,
          "turkishName": "gerdaniye"
        },
        {
          "type": "note",
          "letter": "F",
          "octave": 5,
          "label": "F",
          "duration": 4,
          "turkishName": "acem"
        },
        {
          "type": "note",
          "letter": "E",
          "octave": 5,
          "label": "E",
          "duration": 4,
          "turkishName": "hüseynî"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 6,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 2,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "B",
          "octave": 4,
          "label": "B↓",
          "duration": 8,
          "turkishName": "dik kürdî",
          "accidental": "bakiye-flat"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 8,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 4,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 4,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 4,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "B",
          "octave": 4,
          "label": "B↓",
          "duration": 4,
          "turkishName": "dik kürdî",
          "accidental": "bakiye-flat"
        },
        {
          "type": "note",
          "letter": "A",
          "octave": 4,
          "label": "A",
          "duration": 8,
          "turkishName": "dügâh"
        }
      ]
    },
    {
      "id": 3,
      "events": [
        {
          "type": "note",
          "letter": "E",
          "octave": 5,
          "label": "E",
          "duration": 8,
          "turkishName": "hüseynî"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 4,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "E",
          "octave": 5,
          "label": "E",
          "duration": 4,
          "turkishName": "hüseynî"
        },
        {
          "type": "note",
          "letter": "F",
          "octave": 5,
          "label": "F",
          "duration": 8,
          "turkishName": "acem"
        },
        {
          "type": "note",
          "letter": "E",
          "octave": 5,
          "label": "E",
          "duration": 4,
          "turkishName": "hüseynî"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 4,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 6,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "B",
          "octave": 4,
          "label": "B↓",
          "duration": 2,
          "turkishName": "dik kürdî",
          "accidental": "bakiye-flat"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 4,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 4,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 4,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "B",
          "octave": 4,
          "label": "B↓",
          "duration": 4,
          "turkishName": "dik kürdî",
          "accidental": "bakiye-flat"
        },
        {
          "type": "note",
          "letter": "A",
          "octave": 4,
          "label": "A",
          "duration": 8,
          "turkishName": "dügâh"
        }
      ]
    },
    {
      "id": 4,
      "events": [
        {
          "type": "note",
          "letter": "E",
          "octave": 5,
          "label": "E",
          "duration": 8,
          "turkishName": "hüseynî"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 4,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 4,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 4,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "B",
          "octave": 4,
          "label": "B↓",
          "duration": 4,
          "turkishName": "dik kürdî",
          "accidental": "bakiye-flat"
        },
        {
          "type": "note",
          "letter": "B",
          "octave": 4,
          "label": "B↓",
          "duration": 4,
          "turkishName": "dik kürdî",
          "accidental": "bakiye-flat"
        },
        {
          "type": "note",
          "letter": "A",
          "octave": 4,
          "label": "A",
          "duration": 4,
          "turkishName": "dügâh"
        },
        {
          "type": "note",
          "letter": "A",
          "octave": 4,
          "label": "A",
          "duration": 4,
          "turkishName": "dügâh"
        },
        {
          "type": "note",
          "letter": "B",
          "octave": 4,
          "label": "B↓",
          "duration": 4,
          "turkishName": "dik kürdî",
          "accidental": "bakiye-flat"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 4,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 4,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "E",
          "octave": 5,
          "label": "E",
          "duration": 4,
          "turkishName": "hüseynî"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 4,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "E",
          "octave": 5,
          "label": "E",
          "duration": 8,
          "turkishName": "hüseynî"
        }
      ]
    },
    {
      "id": 5,
      "events": [
        {
          "type": "note",
          "letter": "E",
          "octave": 5,
          "label": "E",
          "duration": 8,
          "turkishName": "hüseynî"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 4,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "E",
          "octave": 5,
          "label": "E",
          "duration": 4,
          "turkishName": "hüseynî"
        },
        {
          "type": "note",
          "letter": "F",
          "octave": 5,
          "label": "F",
          "duration": 8,
          "turkishName": "acem"
        },
        {
          "type": "note",
          "letter": "E",
          "octave": 5,
          "label": "E",
          "duration": 4,
          "turkishName": "hüseynî"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 4,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 6,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "B",
          "octave": 4,
          "label": "B↓",
          "duration": 2,
          "turkishName": "dik kürdî",
          "accidental": "bakiye-flat"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 4,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 4,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 4,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "B",
          "octave": 4,
          "label": "B↓",
          "duration": 4,
          "turkishName": "dik kürdî",
          "accidental": "bakiye-flat"
        },
        {
          "type": "note",
          "letter": "A",
          "octave": 4,
          "label": "A",
          "duration": 8,
          "turkishName": "dügâh"
        }
      ]
    },
    {
      "id": 6,
      "events": [
        {
          "type": "note",
          "letter": "E",
          "octave": 5,
          "label": "E",
          "duration": 8,
          "turkishName": "hüseynî"
        },
        {
          "type": "note",
          "letter": "D",
          "octave": 5,
          "label": "D",
          "duration": 4,
          "turkishName": "neva"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 4,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "C",
          "octave": 5,
          "label": "C♯",
          "duration": 4,
          "turkishName": "nîm hicâz",
          "accidental": "bakiye-sharp"
        },
        {
          "type": "note",
          "letter": "B",
          "octave": 4,
          "label": "B↓",
          "duration": 4,
          "turkishName": "dik kürdî",
          "accidental": "bakiye-flat"
        },
        {
          "type": "note",
          "letter": "B",
          "octave": 4,
          "label": "B↓",
          "duration": 4,
          "turkishName": "dik kürdî",
          "accidental": "bakiye-flat"
        },
        {
          "type": "note",
          "letter": "A",
          "octave": 4,
          "label": "A",
          "duration": 4,
          "turkishName": "dügâh"
        },
        {
          "type": "note",
          "letter": "A",
          "octave": 4,
          "label": "A",
          "duration": 24,
          "turkishName": "dügâh"
        },
        {
          "type": "rest",
          "duration": 8
        }
      ]
    }
  ],
  "parts": [
    {
      "id": "part-1",
      "title": "PART 1: Ey rahmeti bol pâdişâh  (bars 1-2, repeat)",
      "bars": [
        1,
        2
      ]
    },
    {
      "id": "part-2",
      "title": "PART 2: Cürmüm ile (Mevlâm) geldim sana  (bars 3-4)",
      "bars": [
        3,
        4
      ]
    },
    {
      "id": "part-3",
      "title": "PART 3: İsyan ile (Mevlâm) geldim sana  (bars 5-6)",
      "bars": [
        5,
        6
      ]
    }
  ],
  "playOrder": [
    {
      "part": "part-1",
      "label": "Verse 1 · Ey rahmeti bol pâdişâh"
    },
    {
      "part": "part-1",
      "label": "Verse 1 · Ey rahmeti bol pâdişâh"
    },
    {
      "part": "part-2",
      "label": "Verse 1 · Ey rahmeti bol pâdişâh"
    },
    {
      "part": "part-3",
      "label": "Verse 1 · Ey rahmeti bol pâdişâh"
    },
    {
      "part": "part-1",
      "label": "Verse 1 · Ben işledim hadsiz günah"
    },
    {
      "part": "part-1",
      "label": "Verse 1 · Ben işledim hadsiz günah"
    },
    {
      "part": "part-2",
      "label": "Verse 1 · Ben işledim hadsiz günah"
    },
    {
      "part": "part-3",
      "label": "Verse 1 · Ben işledim hadsiz günah"
    },
    {
      "part": "part-1",
      "label": "Verse 2 · Kuddûsî isyanda şedîd"
    },
    {
      "part": "part-1",
      "label": "Verse 2 · Kuddûsî isyanda şedîd"
    },
    {
      "part": "part-2",
      "label": "Verse 2 · Kuddûsî isyanda şedîd"
    },
    {
      "part": "part-3",
      "label": "Verse 2 · Kuddûsî isyanda şedîd"
    },
    {
      "part": "part-1",
      "label": "Verse 2 · Der senden kesmeyip ümîd"
    },
    {
      "part": "part-1",
      "label": "Verse 2 · Der senden kesmeyip ümîd"
    },
    {
      "part": "part-2",
      "label": "Verse 2 · Der senden kesmeyip ümîd"
    },
    {
      "part": "part-3",
      "label": "Verse 2 · Der senden kesmeyip ümîd"
    }
  ]
}
```
