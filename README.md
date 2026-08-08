# bd_M_Fractal — L-system fractal music module

Standalone browser app that turns an [L-system](https://en.wikipedia.org/wiki/L-system) fractal curve into music. Following Prusinkiewicz's approach (*The Algorithmic Beauty of Plants*, Chapter 6): interpret the curve as a 2-D turtle path, sonify horizontal segments as notes (y-coordinate → scale degree, run length → note duration), voice each note as a 3-note chord by looking ahead at future notes in the sequence, render via [Tone.js](https://tonejs.github.io/) with bass-recorder samples.

- **Live**: [wrcstewart.github.io/bd_M_Fractal/preview.html](https://wrcstewart.github.io/bd_M_Fractal/preview.html)
- **Repo**: [github.com/wrcstewart/bd_M_Fractal](https://github.com/wrcstewart/bd_M_Fractal)
- **License**: CC0-1.0 (public domain dedication)
- **Deps**: none to install. Tone.js + abcjs load from CDN, everything else is inline. Three HTML files + four sample MP3s. No build step.

Sibling modules in the same family: [bd_V_Kolam](https://github.com/wrcstewart/bd_V_Kolam) (visual L-system kolam patterns) and [bd_M_ABC](https://github.com/wrcstewart/bd_M_ABC) (direct ABC-notation music player).

---

## Table of contents

1. [Quick start](#quick-start)
2. [Directive reference](#directive-reference)
3. [Pipeline overview](#pipeline-overview)
4. [Key algorithms](#key-algorithms)
5. [File layout](#file-layout)
6. [Extending the module](#extending-the-module) — designed to be forked with AI assistance
7. [Message protocol (embedding in a host)](#message-protocol)
8. [Sibling modules and the two-copy convention](#sibling-modules)
9. [License](#license)

---

## Quick start

1. Visit [the live URL](https://wrcstewart.github.io/bd_M_Fractal/preview.html).
2. The default script loads a Peano-curve grammar at iteration 5, minor pentatonic on C3, played as octave-spread chords. Press **Play**.
3. Edit any `%%bd_` line in the left panel textarea, press **↓ Send** to re-derive.
4. Use the right-column steppers for real-time tweaks (iteration count, tempo, scale, effects). Structural changes stop playback and re-derive; effect changes route straight to the live audio graph.
5. **Copy Grammar** copies the current script (grammar + all directives). **Copy for ABC Player** wraps the derived ABC into a `%%bd_module bd_M_ABC` shell for pasting into the [bd_M_ABC](https://github.com/wrcstewart/bd_M_ABC) module. **Bake** renders offline to WAV and drops the file into the bottom-bar media player. **Save wav** / **Save midi** download the result.

### Default script

```
%%bd_module bd_M_Fractal
%%bd_axiom X
%%bd_rule X: XFYFX+F+YFXFY-F-XFYFX
%%bd_rule Y: YFXFY-F-XFYFX+F+YFXFY
%%bd_iterations 5
%%bd_angle 90
%%bd_scale minor_pentatonic
%%bd_root C,
%%bd_step_seconds 0.2
%%bd_vertical_time 0.2
%%bd_reverb_wet 0.35
%%bd_reverb_decay 2.5
%%bd_vibrato_frequency 5.0
%%bd_vibrato_depth 0.15
%%bd_chorus_wet 0.2
%%bd_chorus_depth 0.3
%%bd_loop false
%%bd_loop_gap 6
%%bd_offset2 30
%%bd_offset3 60
```

### Swap in a Hilbert grammar

```
%%bd_axiom A
%%bd_rule A: -BF+AFA+FB-
%%bd_rule B: +AF-BFB-FA+
```

Any L-system grammar with `F` (draw forward), `+` (turn CCW), `-` (turn CW), and single-letter non-drawing placeholders works.

---

## Directive reference

The script is a list of `%%bd_<name> <value>` lines. Every directive is optional (fallbacks apply); the `%%bd_module bd_M_Fractal` line identifies the module type but isn't strictly needed once running standalone.

### Structural (grammar-side)

| Directive | Default | Notes |
|---|---|---|
| `%%bd_axiom <string>` | `X` | Starting symbol string for the L-system. |
| `%%bd_rule <sym>: <expansion>` | `X: XFYFX+F+YFXFY-F-XFYFX` / `Y: YFXFY-F-XFYFX+F+YFXFY` | One line per single-character symbol. Any number of rule lines allowed. |
| `%%bd_iterations <int>` | `5` | L-system expansion depth. Range 5–20 (min 5 to guarantee interesting output). Very high iterations degrade gracefully via memory-ceiling stepdown. |
| `%%bd_angle <deg>` | `90` | Turn angle applied by `+` / `-` in the turtle interpreter. |

### Pitch mapping

| Directive | Default | Notes |
|---|---|---|
| `%%bd_scale <name>` | `minor_pentatonic` | One of: `minor`, `major`, `minor_pentatonic`, `blues`. Alias `pentatonic` → `minor_pentatonic` (legacy). |
| `%%bd_root <note>` | `C,` | ABC-notation pitch: letter (A–G), optional `#` or `b`, optional `,`/`'` octave suffix (`,` = one octave down, `,,` = two, `'` = one up, etc.). Clamped so all derived notes fit within full piano range MIDI [21, 108]. |
| `%%bd_step_seconds <sec>` | `0.2` | Real-time duration of one turtle F step (and thus of one quarter-note in the derived ABC). Range 0.1–2.0. |
| `%%bd_vertical_time <mult>` | `0.2` | Multiplier applied to a vertical run's length to determine rest duration. `0` = silent transition (no time cost), `1` = full rest matching vertical run length. Range 0.0–2.0. |

### Chord voicing

Each melody note is voiced as a 3-note chord by looking ahead at future notes in the sequence. Set both offsets to `0` to disable chord mode (single-note melody).

| Directive | Default | Notes |
|---|---|---|
| `%%bd_offset2 <int>` | `30` | Number of notes ahead in the sequence to source the middle voice from. That voice's pitch is then transposed **down** one octave. |
| `%%bd_offset3 <int>` | `60` | Same for the top voice, transposed **up** one octave. |

The piece ends `max(offset2, offset3)` notes shorter than melody form so every emitted chord is a complete triad.

### Effects (Tone.js audio graph)

Live-tweak these via right-column steppers without re-deriving the ABC or stopping playback. Same names + shape as [bd_M_ABC](https://github.com/wrcstewart/bd_M_ABC) so scripts are cross-compatible where the note content overlaps.

| Directive | Default | Range | Notes |
|---|---|---|---|
| `%%bd_reverb_wet` | `0.35` | 0–1 | Dry/wet mix. |
| `%%bd_reverb_decay` | `2.5` | 0.1–20 | Reverb tail (seconds). Log-stepped in the UI (perceptual). |
| `%%bd_vibrato_frequency` | `5.0` | 0.5–20 | Hz. Log-stepped. |
| `%%bd_vibrato_depth` | `0.15` | 0–1 | Vibrato modulation depth. |
| `%%bd_chorus_wet` | `0.2` | 0–1 | Chorus mix. |
| `%%bd_chorus_depth` | `0.3` | 0–1 | Chorus modulation depth. |
| `%%bd_loop <bool>` | `false` | true/false | Auto-restart at end. |
| `%%bd_loop_gap <sec>` | `6` | 0–30 | Seconds between loop iterations. |

---

## Pipeline overview

The module runs a fixed pipeline. Each stage takes the previous stage's output as input and produces a new representation.

```
    grammar (%%bd_ directives)
         │
         ▼   parseGrammarFromScript
    { axiom, rules, iterations, angle, scale, root, offsets, effects }
         │
         ▼   expandLSystem   (DFS streaming — memory O(iter))
    symbol string, up to MAX_TURTLE_INPUT_CHARS
         │
         ▼   turtleWalk       (F draws, +/− turn, other letters skipped)
    segments [ { symIdx, x1, y1, x2, y2, isHorizontal, length } ]
         │
         ▼   skip past iter (N−1) prefix
    post-skip segments
         │
         ▼   collapseRuns     (merge consecutive same-y horizontals)
    runs [ { isHorizontal, y1, length } ]
         │
         ▼   applyPitchReflection  (bounce between ±scaleLength walls)
    runs with effective-y values in [−scaleLen, +scaleLen]
         │
         ▼   tonic scan       (start at first horizontal, y ≡ 0)
    pitched runs
         │
         ▼   generateAbc      (chord voicing + ABC emission)
    ABC string with chord brackets [abc]<len>
         │
         ├───────────────► display in read-only #abc-pane
         │
         ▼   abcjs.parseOnly → extractNotes → buildEvents
    Tone.js note-event list [ { time, note, duration } ]
         │
         ▼   startPlayback    (Tone.Part → sampler → chorus → vibrato → reverb → destination)
    audio
```

---

## Key algorithms

### DFS streaming L-system expansion

L-system rewriting is a tree: axiom → children (rule expansion) → grand-children, etc. To materialise iteration N as a linear string, most implementations build iter 1's string, then iter 2's string, etc. — memory grows as `O(k^iterations)`.

We instead traverse the tree **depth-first**, emitting one symbol at a time via an explicit-stack loop. Memory cost is `O(iterations)` for the stack. For Peano's 9× per-iteration growth, that's the difference between iter 10 being "trivially fast" and iter 10 being "150 megabytes in a JS string".

Trade-off: DFS emits leftmost-first, meaning iter N's first L(N−1) symbols are identical to iter (N−1)'s full output. Without countermeasure this makes iterations audibly identical at the start.

### Skip mechanism (post-DFS deduplication)

Because of the DFS shared-prefix property, `regenerateAbc()` computes `L(N−1)` via memoised `computeExpansionLength()` and slices the segment list to drop everything up to that symbol index. Turtle state is preserved by walking through the whole prefix (not just skipping); only the *emitted* segments start after the skip point.

For very high iterations where `L(N−1)` alone exceeds our `MAX_TOTAL_EMISSION` (5 M symbols) memory cap, an effective-iteration guard walks the requested iter down until skip fits. Status readout flags this: `iter 15 too big → using 7`.

### Pitch reflection (bounded 2-octave range)

Raw curve y can span dozens of units for a high-iteration Peano — a naive y → scale-degree mapping would produce 5+ octaves of range, unmusical.

`applyPitchReflection` walks y-deltas as a particle bouncing between two walls at `−scaleLength` and `+scaleLength`. Each unit step of the raw delta is applied; if a step would cross a wall, direction reverses and the step is bounced back into range (not consumed). Effective y always stays in `[−scaleLength, +scaleLength]`, mapping to two octaves centred on the tonic.

**Seed nuance**: the walk starts at `effectiveY = rawY mod scaleLength` rather than 0. This makes different iterations (which land at different absolute y coordinates at the slice point) produce genuinely different opening notes — otherwise Peano's self-similar delta shape would give iter 2 and iter 3 identical opening melodies even after the skip.

### Tonic scan

Consequence of the raw-y seed: iterations open on random scale degrees. Musically odd (piece in C could start on E or G).

Fix: after reflection, `findIndex` the first horizontal run whose effective y is `≡ 0 mod scaleLength`. Slice runs from there. Every iteration starts on the tonic; material after is whatever the walk produces from that point.

### Run collapsing

Consecutive same-y horizontal segments merge into a single sustained note whose length is the run count. Same for consecutive verticals → single rest. This gives the melody varying note durations (halves, dotted-halves, whole notes) instead of monotonous quarters — matches the "varying-length horizontal lines" the book describes.

### Chord voicing

For each note at position k in the notes list, emit an ABC chord `[p1 p2 p3]<len>`:

- `p1` = notes[k].midi (unchanged)
- `p2` = notes[k + offset2].midi − 12 (octave down)
- `p3` = notes[k + offset3].midi + 12 (octave up)

If a shift would push a note off the piano ([21, 108]), that voice keeps its original octave.

Piece ends at `k = notes.length − max(offset2, offset3)` so every chord is a complete triad.

### Root octave clamping

`%%bd_root <note>` accepts ABC octave suffixes (`C,,` = 2 octaves below middle C, `C''` = 2 above). Parsed as: `octShift = suffix.length × sign`. Clamped so `tonicMidi ∈ [33, 96]` — with the reflection's ±octave range, every note lands in `[21, 108]` = the 88-key piano.

---

## File layout

```
bd_M_Fractal/
├── music_module.html    ← the module: generator + player + UI
├── preview.html         ← standalone harness (loads music_module in iframe)
├── bass-recorder/       ← 4 mp3 samples (F#2, C3, G#3, E4) for Tone.Sampler
│   ├── bass_recorder_C3.mp3
│   ├── bass_recorder_E4.mp3
│   ├── bass_recorder_Fs2.mp3
│   └── bass_recorder_Gs3.mp3
├── LICENSE              ← CC0-1.0
├── README.md            ← this file
└── .nojekyll            ← disables Jekyll on GitHub Pages
```

The two HTML files are structured for readability: section markers `// ── <name> ────` before each major block, and function order roughly matches the pipeline order in the [overview](#pipeline-overview).

---

## Extending the module

The code is deliberately organised so an AI assistant (or human) can locate and modify each concern in isolation. Key entry points:

### Add a new scale

1. Add an entry to the `SCALES` object in `music_module.html` (search for `const SCALES`). Value is an array of semitone offsets from the tonic. E.g. `dorian: [0, 2, 3, 5, 7, 9, 10]`.
2. Add a corresponding entry to `SCALE_TO_ABC_KEY_SUFFIX` (`'min'` or `'maj'` — governs the `K:` header, affects implicit accidentals in the emitted ABC).
3. The stepper picks up new scales automatically via `SCALE_NAMES = Object.keys(SCALES)`.

### Add a new numeric-arg directive with a stepper

1. Add an entry to the `STEPPERS` array (search for `const STEPPERS`). Fields: `name`, `min`, `max`, `step`, `fallback`, plus optional `regen` (re-derives ABC on change), `logStep` (perceptual ratio scaling), `isToggle` (boolean on/off), `isEnum` (cycled string values).
2. Add a parser line in `parseGrammarFromScript` matching `/^%%bd_<name>\s+([\d.]+)/m`.
3. Add a sync line in `syncSteppersFromScript` and an emit line in `currentGrammarScript`.
4. If it affects the derived ABC (structural), read it in `regenerateAbc`. If it's an effect (audio-graph), route it through `applyParam`.

### Add a new script-only directive (no stepper)

Same as above minus the STEPPERS entry. Parse in `parseGrammarFromScript`, use in whichever function consumes it, echo back in `currentGrammarScript` by reading `parsedForOctave.<field>` at the top of that function.

### Change the sonification (y → pitch is one choice)

Replace the `pitchForY` function. Inputs: raw y (or effective y after reflection), scale name, root pitch class, base octave. Output: MIDI note number. Everything downstream (chord voicing, ABC emission, playback) is agnostic to how you derive pitch.

Some interesting alternatives:
- **x → pitch** (curves traversed horizontally suggest melodic ideas).
- **Segment direction → pitch** (east/north/west/south → four scale degrees).
- **Distance from origin → pitch** (concentric zones = tonic clusters).

### Add a drone layer

In the `try { … audio graph }` block, add a `Tone.Oscillator` (or `Tone.PolySynth`) alongside the sampler. Start it on Play, stop on Stop. Tune to `pitchForY(0, g.scale, g.root, 4 + g.rootOctave)` for tonic. Add a `%%bd_drone` directive + stepper to control gain.

### Change the chord voicing

Modify the `if (chordMode) { … }` block in `generateAbc`. To emit 4-note chords, add p4 as another look-ahead. To use octave-DOWN for offset3 as well, negate the shift. To make voicing per-note random, swap in `Math.random()` in the shift.

### Debugging

Uncaught JS errors surface in the status strip via `window.onerror`. Deliberate log points:
- `[bd_M_Fractal] bake:` — three lines during a Bake showing start / AudioBuffer / posted-message stages
- `[bd_M_Fractal] L-system:` — memory-cap warnings
- `[bd_M_Fractal] truncating` — MAX_NOTES cap

Open browser DevTools → Console.

---

## Message protocol

The module is designed to be dropped into any host that speaks its `postMessage` protocol (same protocol as [bd_V_Kolam](https://github.com/wrcstewart/bd_V_Kolam) and [bd_M_ABC](https://github.com/wrcstewart/bd_M_ABC)). It works as a standalone via `preview.html` but can equally live inside a two-iframe stack under a graph-viewer host.

### Downstream (host → module)

| Type | Payload | Purpose |
|---|---|---|
| `BD_INIT` | `{ text }` | Initial script on load. |
| `bd_script_update` | `{ script }` | Replace current script (fresh grammar / re-parse / re-derive). |
| `bd_script_request` | — | Ask the module to send its current script back. |
| `BD_STOP` | — | Stop playback. |

### Upstream (module → host)

| Type | Payload | Purpose |
|---|---|---|
| `BD_READY` | — | Sent once samples + reverb IR are loaded. |
| `bd_script_response` | `{ script }` | Reply to `bd_script_request`, includes current stepper values baked back into the script. |
| `BD_MEDIA_BLOB` | `{ label, audioData: ArrayBuffer, mime, sizeBytes }` | Bake-to-WAV output. Host attaches to a media player. |
| `BD_MODULE_COPY_LINK_REQUEST` | `{ text }` | Ask host to build a share-link URL wrapping the current script. |
| `BD_ERROR` | `{ message }` | Non-fatal error to surface in host UI. |

### URL params (standalone only)

| Param | Format | Purpose |
|---|---|---|
| `?data=<base64>` | JSON `{ script, node_url, source_text, title, name }` | Full arrival envelope, used when linking in from a Butterfly-Dreaming platform node. |
| `?script=<base64>` | Raw script text | Simpler share-link. |

---

## Sibling modules

Third of three complementary standalones under the ButterflyDreaming media-module family:

- **[bd_V_Kolam](https://github.com/wrcstewart/bd_V_Kolam)** — visual L-system → kolam pattern.
- **[bd_M_ABC](https://github.com/wrcstewart/bd_M_ABC)** — direct ABC-notation music player.
- **bd_M_Fractal** (this repo) — L-system → curve → ABC melody with chord voicing.

Each shares the same `%%bd_…` directive convention, the same postMessage protocol, and the same BD-return deep-link envelope. The "Copy for ABC Player" button in this module produces a script that can be pasted straight into a bd_M_ABC node — one direction of a small module-to-module handoff pattern.

### Two-copy convention

Each module lives in two places once wired into the parent [ButterflyDreaming graph viewer](https://github.com/wrcstewart/butterflydreaming-graphviewer1): once as the standalone here, once embedded in the viewer's repo. When both copies exist, they must be manually kept in sync — small acceptable drift (e.g. the standalone-only `Save wav` / `Save midi` / `Download` buttons in bd_M_ABC) is documented in the file-top comment of the divergent copy.

`bd_M_Fractal`'s embedded BD copy is **not yet done** at time of writing.

---

## License

Released under [**CC0-1.0**](https://creativecommons.org/publicdomain/zero/1.0/) (public domain dedication). Do what you like with it, no attribution required, no warranty implied.
