# Modules

`Type` is the identifier used in patch JSON; every module exposes its
parameters as MIDI CCs per the [control scheme](MIDI.md#2-live-control--channel-16).

| Module | Type | Category | Description |
|---|---|---|---|
| MidiIn | `midi_in` | Routing | Live MIDI input from the Force's pads/keys into a lane (e.g. `midi_in` → Arp arpeggiates whatever you hold); filterable to Omni or a single channel. |
| BusIn | `bus_in` | Routing | Reads a named internal note bus and emits it as a source; two `bus_in` lanes reading the same name both play whatever's sent to it. |
| BusSend | `bus_send` | Routing | Mirrors a lane's notes onto a named internal bus while still passing them through — e.g. one ChordSeq feeding two independent Arps. |
| DrumSeq | `drum_seq` | Rhythm | Genre-aware 8-lane drum sequencer (23 genres) — per-step velocity/probability, swing, ratchet/roll, GM drum-note mapping to the MPC Drum group. |
| Euclid | `euclid` | Rhythm | Poly-lane Euclidean rhythm generator (Bjørklund); each lane runs its own steps/pulses/rotation, together building a full drum voice. |
| Grids | `grids` | Rhythm | Mutable Grids-style 2D drum pattern morphing — an X/Y coordinate blends four corner patterns per voice (kick/snare/hat). |
| StepSeq | `step_seq` | Rhythm | Elektron/Opal-style paged step sequencer for drums — per-step velocity, probability, ratchet, micro-timing, gate length, trig conditions; mouse-drag painting in the web UI. |
| ChordSeq | `chord_seq` | Harmony | Roman-numeral chord progression generator with voice-leading; publishes the current chord + scale to the shared harmony bus so other modules follow the changes. |
| Chord | `chord` | Harmony | Turns a single incoming note into a chord — diatonically via the harmony bus, or a fixed quality — handy for harmonizing a generative line. |
| Gen | `gen` | Melody | Scale-based generative sequencer with phrase-aware melodic logic — chord-tone bias on strong beats, phrase-end rests, a repeating motif buffer. |
| Marbles | `marbles` | Melody | Controllable-randomness note + gate source (Mutable Marbles-inspired); "déjà vu" blends fresh random notes with a locked, repeating loop. |
| Markov | `markov` | Melody | Markov-chain melody generator with musical priors — favors stepwise motion and chord tones over big leaps, boosted on strong beats by the harmony bus. |
| Turing | `turing` | Melody | Looping shift-register sequencer (Turing Machine-inspired); "chance" scrambles a locked, repeating loop into controlled randomness. |
| Arp | `arp` | Melody | Step arpeggiator over the currently held notes — mode (up/down/random/...), octave range, and rhythm pattern. |
| ScaleQuantizer | `scale_quantizer` | Utility | Snaps incoming note pitches to a chosen scale/key (or the shared harmony bus); no stuck notes. |
| LFO | `lfo` | Modulation | Clock-synced low-frequency oscillator emitting CC values, for hands-free movement of the Force's macros. |
| Tides | `tides` | Modulation | Shapeable clock-synced modulation source (Tides-inspired); slope morphs the waveform from ramp-down through triangle to ramp-up. |
| Ratchet | `ratchet` | Utility | Stutter/roll processor; chops an incoming note into rapid same-pitch retriggers at a configurable probability and division. |
| Swing | `swing` | Utility | Groove/timing processor; delays off-beat steps for a shuffled feel, with optional velocity and micro-timing humanization. |
| Branches | `branches` | Utility | Bernoulli gate/probability router (Mutable Branches-inspired); routes or drops each note across two outputs via a weighted coin flip. |

Gen, Marbles, Markov and Turing also take an explicit **key/root**, so a
standalone voice with no ChordSeq publisher on the harmony bus isn't stuck on
a default key.
