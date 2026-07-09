# Modules

`Type` is the identifier used in patch JSON; every module exposes its
parameters as MIDI CCs per the [control scheme](MIDI.md#2-live-control--channel-16).
36 modules across 6 categories (plus 2 SPINE-only modules, listed separately
at the bottom — SPINE builds them for you, they don't appear in the palette).

## Rhythm

| Module | Type | Description |
|---|---|---|
| DrumSeq | `drum_seq` | Genre-aware 8-lane drum sequencer (23 genres) — per-step velocity/probability, swing, ratchet/roll, GM drum-note mapping to the MPC Drum group. |
| Euclid | `euclid` | Poly-lane Euclidean rhythm generator (Bjørklund); each lane runs its own steps/pulses/rotation, together building a full drum voice. |
| Grids | `grids` | Mutable Grids-style 2D drum pattern morphing — an X/Y coordinate blends four corner patterns per voice (kick/snare/hat). |
| StepSeq | `step_seq` | Elektron/Opal-style paged step sequencer for drums — per-step velocity, probability, ratchet, micro-timing, gate length, trig conditions; mouse-drag painting in the web UI. |
| StageSeq | `stage_seq` | Metropolis/Metropolix-style **stage** sequencer — a short row of stages, each with its own pitch, pulse count, and gate pattern; a different paradigm from StepSeq's flat grid. |
| Polyrhythm | `polyrhythm` | Three independent pulse-grids firing at once (configurable ratios), each on a different pitch, for audibly distinct polyrhythmic layers. |

## Harmony

| Module | Type | Description |
|---|---|---|
| ChordSeq | `chord_seq` | Roman-numeral chord progression generator with voice-leading; publishes the current chord + scale to the shared harmony bus so other modules follow the changes. |
| Chord | `chord` | Turns a single incoming note into a chord — diatonically via the harmony bus, or a fixed quality — handy for harmonizing a generative line. |
| Harmonizer | `harmonizer` | Adds up to three harmony voices above/below every incoming note, with correctly matched note-offs. |

## Melody

| Module | Type | Description |
|---|---|---|
| Gen | `gen` | Scale-based generative sequencer with phrase-aware melodic logic — chord-tone bias on strong beats, phrase-end rests, a repeating motif buffer. |
| Marbles | `marbles` | Controllable-randomness note + gate source (Mutable Marbles-inspired); "déjà vu" blends fresh random notes with a locked, repeating loop. |
| Markov | `markov` | Markov-chain melody generator with musical priors — favors stepwise motion and chord tones over big leaps, boosted on strong beats by the harmony bus. |
| Turing | `turing` | Looping shift-register sequencer (Turing Machine-inspired); "chance" scrambles a locked, repeating loop into controlled randomness. |
| Arp | `arp` | Step arpeggiator over the currently held notes — mode (up/down/random/...), octave range, and rhythm pattern. |
| SampleHold | `sample_hold` | Classic sample-and-hold random melody — every N pulses, draws a fresh in-scale note and holds it until the next sample. |
| Chaos | `chaos` | Chaotic shift-register melody (a "rungler"/wild-Turing cousin) — a Chaos% knob controls how often the register mutates, from a locked loop to never settling. |
| Stochastic | `stochastic` | Probability-weighted melody generator — per step, a density dice decides if it sounds, then a biased random draw picks the (in-scale) pitch. |

## Modulation

| Module | Type | Description |
|---|---|---|
| LFO | `lfo` | Clock-synced low-frequency oscillator emitting CC values, for hands-free movement of the Force's macros. |
| Tides | `tides` | Shapeable clock-synced modulation source (Tides-inspired); slope morphs the waveform from ramp-down through triangle to ramp-up. |

## Utility

| Module | Type | Description |
|---|---|---|
| ScaleQuantizer | `scale_quantizer` | Snaps incoming note pitches to a chosen scale/key (or the shared harmony bus); no stuck notes. |
| Ratchet | `ratchet` | Stutter/roll processor; chops an incoming note into rapid same-pitch retriggers at a configurable probability and division. |
| Swing | `swing` | Groove/timing processor; delays off-beat steps for a shuffled feel, with optional velocity and micro-timing humanization. |
| Branches | `branches` | Bernoulli gate/probability router (Mutable Branches-inspired); routes or drops each note across two outputs via a weighted coin flip. |
| Transpose | `transpose` | Shifts incoming notes by semitones + octaves; everything else (CC, clock, transport) passes straight through. |
| Delay | `delay` | MIDI echo/delay line — each note spawns up to 8 decaying echo taps. |
| Dynamics | `dynamics` | Reshapes note velocities according to a curve/mode; pitch and timing are never touched. |
| NoteFilter | `note_filter` | Gates notes by pitch AND velocity range — held-note-safe, so a note already sounding always gets its note-off even if the range changes mid-note. |
| Gate | `gate` | Reshapes note *durations* — passes the note-on through immediately, then decides independently when it releases. |
| Humanize | `humanize` | Jitters timing and velocity to loosen up a mechanical stream. |
| Chance | `chance` | Probabilistic note gate/stutter — rolls independent dice per incoming note. |
| Strum | `strum` | Spreads simultaneous chord notes across time instead of letting them all hit at once. |
| Mutate | `mutate` | Bloom-style continuous evolution — each note is probabilistically nudged to a neighboring scale tone, so a repeating phrase slowly drifts while staying musical. |

## Routing

| Module | Type | Description |
|---|---|---|
| MidiIn | `midi_in` | Live MIDI input from the Force's pads/keys into a lane (e.g. `midi_in` → Arp arpeggiates whatever you hold); filterable to Omni or a single channel. |
| BusIn | `bus_in` | Reads a named internal note bus and emits it as a source; two `bus_in` lanes reading the same name both play whatever's sent to it. |
| BusSend | `bus_send` | Mirrors a lane's notes onto a named internal bus while still passing them through — e.g. one ChordSeq feeding two independent Arps. |
| ModSend | `mod_send` | Taps a control value out of a lane onto a named **modulation** bus, so another module's parameter can be live-sourced from it (an LFO driving a second module's density, etc.) — the continuous-value sibling of BusSend. |

## SPINE-only (not in the palette — see [SPINE](../README.md#spine))

| Module | Type | Description |
|---|---|---|
| SpinePart | `spine_part` | Holds one SPINE-generated part's baked note list and plays it back on loop. SPINE builds these for you; adding one by hand does nothing useful. |
| SpineCc | `spine_cc` | Holds one SPINE-generated CC automation lane (filter sweeps, riser swells, sidechain-style ducking) and plays it back on loop. Same deal — SPINE-only. |

---

Gen, Marbles, Markov and Turing also take an explicit **key/root**, so a
standalone voice with no ChordSeq publisher on the harmony bus isn't stuck on
a default key.
