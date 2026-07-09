# SPINE — the generative composition engine

Everything else in RackForce is **bottom-up**: you wire live modules together
and the patch plays. SPINE is the opposite — **top-down**: it composes a
finished multi-part arrangement first (harmony, rhythm, bass, melody, drums,
automation), then hands each part to the rack as an ordinary player module.
One click and you have a song; nothing to wire.

Open the **SPINE** tab (next to **Rack**) in the web patch bay.

## What it generates

- **A full band** — kick, bass, hats, chords, arp, and a lead, each aware of
  the others (bass and hats lock to the kick; melodic parts read the chord
  progression off a shared harmony bus, same as ChordSeq does for the rest of
  the rack).
- **Real harmony** — extended chords (maj7/min9/sus/6-9/13...), voice-leading
  between changes (the chord *glides* instead of re-rooting), idiomatic
  progressions, a mood/vibe pool to pick from.
- **Groove** — swing, per-part micro-timing, velocity dynamics, ghost notes —
  so it doesn't sound like a grid.
- **A long-form arrangement** — intro → build → breakdown → build-up → drop →
  evolve → outro, not just a looping 4 bars.
- **CC automation** — baked filter-cutoff sweeps, reverb/delay swells, and a
  sidechain-style volume duck locked to the kick, so the patch actually
  *moves* over the arrangement. Map a Force track's filter/volume CC to hear
  it.
- **Variation** — a `variation` dial controls how much a seed changes not
  just *which notes* but the whole character: kick pattern style, section
  lengths, harmonic rhythm, density spread. Turn it up and different seeds
  genuinely sound like different tracks, not reshuffled versions of one track.

## Using it

1. Set key/scale/bars/density (and optionally a specific progression) and hit
   **Generate**. SPINE builds the whole arrangement and installs it live —
   the rack adopts it the same way the classic one-click **Generate** panel
   (composer.cpp's simpler cousin) does, just with far more going on.
2. Don't like one part? Hit its **⟳ re-roll** button. Only that part changes
   — everything else stays byte-identical. This is real, not approximate: a
   splittable RNG gives every part (and the harmony) its own independent
   random stream, so re-rolling bass never touches kick or chords.
3. **Piano roll** — see every note SPINE generated, per part, so you're not
   guessing what a re-roll changed.
4. Once you've got something you like, it's just player modules
   (`spine_part`/`spine_cc`) sitting in the rack like anything else — expand,
   inspect, or route them same as any voice.

## The honest division of labor

SPINE writes the *notes and motion*: harmony, rhythm, arrangement,
automation targets. The Force still makes the *sound* — patch choice, reverb/
delay sends, and (if you map a CC) the actual filter SPINE is sweeping are
still yours to set up. SPINE gets the composition right; the mix is still a
Force patch.
