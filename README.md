# RackForce

[![Release](https://img.shields.io/github/v/release/Devko/RackForce?include_prereleases&label=release)](../../releases)

A browser-patchable modular MIDI rack for the **Akai Force**, running as a
[MockbaMod](https://github.com/MockbaTheBorg/MockbaMod) AddOn. Generative and
Euclidean sequencers, a drum machine, a chord engine, quantizers, LFOs — wired
together and driving the Force's own synths over MIDI. Design takes cues from
Mutable Instruments' Eurorack modules (Marbles, Grids, Tides, Branches): the
Force makes the sound, RackForce makes the musical decisions.

Runs alongside Akai's own app as an overlay AddOn — doesn't touch the firmware.

> v0.2.0-beta — early build, expect rough edges. Bug reports welcome, see
> [Feedback](#feedback).

## Highlights

- **Musicality pass** — velocity accents, humanization, phrase-aware
  generative lines, fixed chord voice-leading, per-drum note config.
- **Routing** — MIDI-In brings live Force pads/keys into a chain; a named
  internal note bus (`bus_send`/`bus_in`) fans one lane's output into others.
- **Live config edits** — changing a genre/scale/key mid-playback no longer
  panics all notes off and restarts the transport.
- **Opal-style step editor** — paged (Trig/Vel/Prob/Gate/Ratchet/Nudge/Mod),
  mouse-drag painting of values across steps.
- **23 drum genres**, key/root picker on Gen/Marbles/Markov/Turing, and an
  honest publisher/follower harmony badge in the web UI.

## Requirements

- Akai Force running **MockbaMod 4.51** (or compatible)
- SSH access to the Force, to make the binary executable and enable the AddOn

## Install

1. Download `RackForce-v0.2.0-beta.zip` from the [Releases page](../../releases).
2. Copy the `ModularRack` folder it contains onto your MockbaMod SD card, into:

   ```
   <SD card>/AddOns/ModularRack/
   ```

   (The AddOn folder keeps the `ModularRack` name on disk — same engine,
   RackForce is the project name.)

3. Zip files don't preserve Unix permissions, so make the binary and scripts
   executable over SSH:

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

   Starts the rack immediately and sets it to auto-launch on boot.
   `sh manage.sh DISABLE` stops it and removes auto-launch;
   `sh manage.sh UNINSTALL` also deletes the AddOn folder.

## MIDI setup

### 1. Route the voices

RackForce registers an ALSA client named `ModularRack`, with one MIDI input
and one MIDI output port. Confirm it's running over SSH:

```sh
aconnect -l   # look for a client called "ModularRack"
```

On each Force MIDI track driven by the rack:

- **INPUT** → the `ModularRack` client, so generated notes reach your
  instrument.
- **OUTPUT** (on the driving track) → `ModularRack`, **Sync ON**, so the
  Force sends MIDI clock (24 PPQN) plus start/stop/continue — without Sync
  the rack gets no clock and won't advance.

One patch drives a whole arrangement — each voice on its own channel:

| Voice  | MIDI channel | Route to |
|--------|:---:|---|
| Lead   | 1  | plugin / keygroup instrument |
| Chords | 2  | plugin / keygroup instrument |
| Bass   | 3  | plugin / keygroup instrument |
| Drums  | 10 | MPC Drum group / Drum Synth  |

### 2. Live control — channel 16

Channel 16 carries knob moves and patch switches only; notes, clock and
transport stay on the voice channels above.

- **Knobs → parameters:** each module gets a block of 10 CCs, in patch order,
  starting at CC 16 — `module = (cc-16)/10`, `param = (cc-16)%10`. A CC's
  0–127 value is rescaled into that parameter's real range. Start the daemon
  with `--verbose` to print the live map for the loaded patch — trust that
  over any static table.
- **Patch switching:** Program Change on channel 16, program = patch index.
- **Macro knobs:** a MIDI track, output `ModularRack`, channel 16, knobs
  learned to CCs from the `--verbose` map. Save it as a Force template
  (`.xtk`) to reuse — same pattern Harpie4T uses for its control surface.

### 3. Web patch bay

Open `http://<force-ip>:8088/` from a phone or laptop on the same network —
one card per module, sliders for every parameter, updating live in both
directions. Single self-contained page, no build step, works offline. A
loopback-only JSON API is also on `127.0.0.1:8099` for scripting.

## Modules

`Type` is the identifier used in patch JSON; every module exposes its
parameters as MIDI CCs per the [control scheme](#2-live-control--channel-16)
above.

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

## Feedback

Open an [issue](../../issues), ideally with the output of running the daemon
directly over SSH with `--verbose`:

```sh
/media/662522/AddOns/ModularRack/modularrack --verbose
```

## Credits

- Built on [MockbaMod](https://github.com/MockbaTheBorg/MockbaMod) 4.51 by
  MockbaTheBorg and Amit Talwar.
- Module design inspired by Mutable Instruments (Marbles, Grids, Tides,
  Branches) and the Music Thing Modular Turing Machine.
