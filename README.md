# bd_M_Fractal — fractal-curve music module for ButterflyDreaming

Standalone deployment of the **bd_M_Fractal** music module — an
L-system driven fractal-curve to ABC melody generator, following
Prusinkiewicz's approach ("The Algorithmic Beauty of Plants",
Chapter 6). The default grammar is the Hilbert curve; the same
module handles Peano, dragon, and other 90° curves via different
grammars.

Live at: **https://wrcstewart.github.io/bd_M_Fractal/preview.html**

## How it works

1. Expand the L-system grammar (axiom + rules) for N iterations.
2. Walk the expanded symbol string with a 2-D turtle (`F` = step
   forward, `+` = turn CCW, `-` = turn CW; other letters are
   non-drawing placeholders).
3. Traverse the resulting curve and turn each **horizontal** segment
   into a note: pitch = y-coordinate mapped through the chosen
   scale, duration = one step. **Vertical** segments become rests
   whose length is `vertical_time × 1 step` (or silent transitions
   with no time cost when `vertical_time = 0`).
4. Emit the note stream as ABC and display it in a read-only pane
   in the module. Same abcjs + Tone.js audio chain as
   [bd_M_ABC](https://github.com/wrcstewart/bd_M_ABC) plays it back.

## Directive vocabulary

```
%%bd_module bd_M_Fractal
%%bd_axiom A
%%bd_rule A: -BF+AFA+FB-
%%bd_rule B: +AF-BFB-FA+
%%bd_iterations 3
%%bd_angle 90
%%bd_scale minor            # minor | major | pentatonic | blues
%%bd_root C                 # C, C#, D, … or a MIDI index 0–11
%%bd_step_seconds 0.5       # duration of one horizontal segment
%%bd_vertical_time 0.5      # fraction of a step used for vertical rests
```

There is **no** `%%bd_score` block — the ABC is *derived* every time
by the module, not authored directly. To edit the ABC output
collaboratively, use the **"Copy for ABC Player"** button in the
module: it wraps the derived ABC in a fully-formed
`%%bd_module bd_M_ABC` script (with sensible reverb/vibrato/chorus
defaults) that can be pasted straight into any bd_M_ABC node in
Butterfly Dreaming.

## Deep links

- **Link into the standalone** — `preview.html?data=<base64>` with
  the ButterflyDreaming payload `{ script, node_url, source_text,
  title, name }`. Used by the platform's "Copy Link to External
  Website" button.
- **Link back to BD** — the "Jump to Butterfly Dreaming" and
  "Copy Link to Butterfly Dreaming" buttons in `preview.html`
  hand a visitor back to the graph at the originating node.

## Relation to bd_V_Kolam and bd_M_ABC

Third member of the media-module family:

- [bd_V_Kolam](https://github.com/wrcstewart/bd_V_Kolam) — visual
  L-system → kolam pattern.
- [bd_M_ABC](https://github.com/wrcstewart/bd_M_ABC) — direct
  ABC-notation music player.
- **bd_M_Fractal** (this repo) — L-system → curve → ABC melody.

The three share the same `%%bd_…` directive convention, the same
BD ⇄ standalone deep-link envelope, and (for the two music modules)
the same Tone.js + bass-recorder sampler audio path.

## License

Released under **CC0-1.0** (public domain dedication). Do what
you like with it, no attribution required.
