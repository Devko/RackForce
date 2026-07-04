# RackForce

A browser-patchable modular MIDI rack for the **Akai Force**, running as a
[MockbaMod](https://github.com/MockbaTheBorg/MockbaMod) AddOn. One background
process runs a graph of one-job modules — generative and Euclidean sequencers,
a drum machine, an intelligent chord/progression engine, quantizers, LFOs —
wired together and driving the Force's own synths over MIDI. Design is
inspired by the Mutable Instruments Eurorack ecosystem (Marbles, Grids, Tides,
Branches...); the Force makes the sound, RackForce makes the musical decisions.

It runs alongside Akai's own app as an overlay AddOn — it does not replace or
modify the firmware.

> **Status: v0.1-beta.** Early build from a community fork. Expect rough
> edges — bug reports and "it doesn't launch on my box" reports are genuinely
> useful, see [Feedback](#feedback) below.

## Requirements

- Akai Force running **MockbaMod 4.51** (or compatible)
- SSH access to the Force (used once, to make the binary executable and
  enable the AddOn)

## Install

1. Download `RackForce-v0.1-beta.zip` from the [Releases page](../../releases).
2. Unzip it, then copy the `ModularRack` folder it contains onto your Force's
   MockbaMod SD card, into:

   ```
   <SD card>/AddOns/ModularRack/
   ```

   (The AddOn folder is still named `ModularRack` on disk — same engine,
   public project name is RackForce.)

3. Make the binary and scripts executable. Zip files don't preserve Unix
   permissions, so this step is required — over SSH on the Force:

   ```sh
   chmod +x /media/662522/AddOns/ModularRack/modularrack
   chmod +x /media/662522/AddOns/ModularRack/manage.sh
   chmod +x /media/662522/AddOns/ModularRack/run_modularrack.sh
   ```

4. Enable it:

   ```sh
   cd /media/662522/AddOns/ModularRack
   sh manage.sh ENABLE
   ```

   This starts the rack immediately and sets it to auto-launch on boot.
   Use `sh manage.sh DISABLE` to stop it and remove it from auto-launch, or
   `sh manage.sh UNINSTALL` to stop it, remove auto-launch, and delete the
   AddOn folder.

## MIDI setup

### 1. Route the voices

RackForce registers its own ALSA sequencer client named `ModularRack`, with
one MIDI input port and one MIDI output port. Confirm it's running over SSH:

```sh
aconnect -l   # look for a client called "ModularRack"
```

On each Force MIDI track you want driven by the rack:

- Set the track's **INPUT** to the `ModularRack` client, so generated notes
  come back into the Force and play through your instrument.
- Set the *driving* track's **OUTPUT** to `ModularRack` and turn **Sync ON**,
  so the Force sends MIDI clock (24 PPQN) plus start/stop/continue. Without
  Sync enabled the rack receives no clock and will not advance.

The rack is multitimbral — one patch drives a whole arrangement at once, each
voice on its own channel. Default patch channel map:

| Voice  | MIDI channel | Route to |
|--------|:---:|---|
| Lead   | 1  | plugin / keygroup instrument |
| Chords | 2  | plugin / keygroup instrument |
| Bass   | 3  | plugin / keygroup instrument |
| Drums  | 10 | MPC Drum group / Drum Synth  |

### 2. Live control — channel 16

MIDI **channel 16 is the control channel**. Notes, clock and transport stay
on the voice channels above, so control traffic never collides with the
music.

- **Knobs → parameters:** every module gets a block of 10 CCs, in the order
  modules were added to the patch, starting at CC 16:

  ```
  module = (cc - 16) / 10      # which module (0 = first added)
  param  = (cc - 16) % 10      # which parameter of that module
  ```

  A CC's value (0–127) is rescaled into that parameter's real range, so a
  knob's full sweep always covers the whole parameter regardless of its
  native range. **Start the daemon with `--verbose`** to print the exact
  live CC → (module, parameter) map for the patch that's actually loaded —
  always trust that over any static table, since the layout depends on which
  modules the patch contains.

- **Patch switching:** send a **Program Change on channel 16** — program
  number = patch index (0 = default patch).

- **Assign the Force's macro knobs:** create a MIDI track, set its output to
  `ModularRack` and its channel to 16, then use the track's MIDI-Control /
  macro-assignment page to learn each knob to a CC from the `--verbose` map.
  Once it feels right, save the track as a Force **track template (`.xtk`)**
  to reuse the whole mapping later without re-learning — the same pattern
  Harpie4T uses for its own control surface.

### 3. Web patch bay

Open `http://<force-ip>:8080/` from a phone or laptop on the same network for
a live, visual view of the loaded patch — one card per module, with sliders
for every parameter, updating in both directions in real time. Fully
self-contained (no build step, no CDNs, works offline). A loopback-only JSON
API is also available on `127.0.0.1:8099` for scripting.

## Modules

17 modules are currently shipped. `Type` is the identifier used in patch
JSON; each exposes its parameters as MIDI CCs per the [control
scheme](#2-live-control--channel-16) above.

| Module | Type | Category | Description |
|---|---|---|---|
| DrumSeq | `drum_seq` | Rhythm | Genre-aware 8-lane drum sequencer — per-step velocity/probability, swing, ratchet/roll, GM drum-note mapping to the MPC Drum group. |
| Euclid | `euclid` | Rhythm | Poly-lane Euclidean rhythm generator (Bjørklund); each lane runs its own steps/pulses/rotation, together building a full drum voice. |
| Grids | `grids` | Rhythm | Mutable Grids-style 2D drum pattern morphing — an X/Y coordinate bilinearly blends four corner patterns per voice (kick/snare/hat). |
| StepSeq | `step_seq` | Rhythm | Deep hardware-flavored multi-lane step sequencer for drums (Digitakt/Opal spirit) — per-step velocity, probability, ratchet count, micro-timing, gate length, trig conditions. |
| ChordSeq | `chord_seq` | Harmony | Roman-numeral chord progression generator with optional voice-leading; publishes the current chord + scale to the shared harmony bus so other modules follow the changes. |
| Chord | `chord` | Harmony | Turns a single incoming note into a chord — diatonically via the harmony bus, or a fixed quality — handy for harmonizing a generative line. |
| Gen | `gen` | Melody | Scale-based generative sequencer; each note is picked near the last one, biased by a contour direction, snapped to the current scale/harmony. |
| Marbles | `marbles` | Melody | Controllable-randomness note + gate source (Mutable Marbles-inspired); a "déjà vu" control blends fresh random notes with a locked, repeating loop. |
| Markov | `markov` | Melody | Markov-chain melody generator with musical priors — favors stepwise motion and chord tones over big leaps, boosted by the harmony bus on strong beats. |
| Turing | `turing` | Melody | Looping shift-register sequencer (Music Thing Turing Machine-inspired); a "chance" control scrambles a locked, repeating loop into controlled randomness. |
| Arp | `arp` | Melody | Step arpeggiator over the currently held notes — mode (up/down/random/...), octave range, and rhythm pattern. |
| ScaleQuantizer | `scale_quantizer` | Utility | Snaps incoming note pitches to a chosen scale/key (or the shared harmony bus); no stuck notes. |
| LFO | `lfo` | Modulation | Clock-synced low-frequency oscillator emitting CC values, for hands-free movement of the Force's macros. |
| Tides | `tides` | Modulation | Shapeable clock-synced modulation source (Tides-inspired); a slope control morphs the waveform from ramp-down through triangle to ramp-up. |
| Ratchet | `ratchet` | Utility | Stutter/roll processor; chops an incoming note into rapid same-pitch retriggers at a configurable probability and division. |
| Swing | `swing` | Utility | Groove/timing processor; delays off-beat steps for a shuffled feel, with optional velocity and micro-timing humanization. |
| Branches | `branches` | Utility | Bernoulli gate / probability router (Mutable Branches-inspired); routes or drops each note across two outputs via a weighted coin flip. |

## Feedback

This is an early build from a community fork — bug reports are genuinely
useful, especially "it doesn't launch on my box" reports. Please open an
[issue](../../issues) and include the output of running the daemon directly
over SSH with `--verbose`:

```sh
/media/662522/AddOns/ModularRack/modularrack --verbose
```

## Credits

- Built on [MockbaMod](https://github.com/MockbaTheBorg/MockbaMod) 4.51 by
  MockbaTheBorg and Amit Talwar.
- Module design inspired by the Mutable Instruments Eurorack ecosystem
  (Marbles, Grids, Tides, Branches) and the Music Thing Modular Turing
  Machine.
