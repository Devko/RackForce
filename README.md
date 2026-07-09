# RackForce

[![Release](https://img.shields.io/github/v/release/Devko/RackForce?include_prereleases&label=release)](../../releases)

A browser-patchable modular MIDI rack for the **Akai Force**, running as a
[MockbaMod](https://github.com/MockbaTheBorg/MockbaMod) AddOn. Generative and
Euclidean sequencers, a drum machine, a chord engine, quantizers, LFOs, and a
full top-down composition engine ([SPINE](#spine)) — wired together and
driving the Force's own synths over MIDI. Design takes cues from Mutable
Instruments' Eurorack modules (Marbles, Grids, Tides, Branches): the Force
makes the sound, RackForce makes the musical decisions.

Runs alongside Akai's own app as an overlay AddOn — doesn't touch the firmware.

### [Download RackForce v0.3.0-beta](https://github.com/Devko/RackForce/releases/tag/v0.3.0-beta)

Prebuilt ARM binary, no GitHub account or build step needed — grab the
`.zip` from the "Assets" section on that page. See [Install](#install) below
for what to do with it.

> v0.3.0-beta — early build, expect rough edges. Bug reports welcome, see
> [Feedback](#feedback).

## Screenshots

The web patch bay — open from any phone or laptop on the same network — is
the easiest way to see what RackForce actually is:

**A patch, built from three voices — a chord engine publishing harmony, a
drum machine, and a bassline following the changes:**
![Rack overview: three voices — chords, drums, bass](docs/images/rack-overview.png)

**Every parameter is a live slider, not just a config file:**
![ChordSeq expanded, showing its live parameter sliders](docs/images/module-expanded.png)

**The step sequencer is a paged, Elektron/Opal-style editor with mouse-drag
painting:**
![StepSeq's paged grid editor: Trig/Vel/Prob/Gate/Ratchet/Nudge/Mod](docs/images/stepseq-editor.png)

## SPINE

The headline feature of this release: a **top-down generative composition
engine**, alongside the bottom-up live rack. One click composes a full
arrangement — harmony with voice-leading, groove, a long-form structure
(intro/build/breakdown/drop/outro), and baked CC automation — then installs
it as ordinary player modules in the rack. Re-roll any single part (bass,
kick, lead...) and only that part changes; everything else stays
byte-identical. A `variation` dial controls how much character changes from
seed to seed, not just which notes.

Full writeup: **[docs/SPINE.md](docs/SPINE.md)**.

## Highlights

- **SPINE** — top-down song generation with real harmony, arrangement, groove,
  and CC automation; per-part re-roll with a true independent lock; a piano
  roll to see what it wrote.
- **36 modules**, up from 20 — new this release: StageSeq, Polyrhythm,
  Harmonizer, SampleHold, Chaos, Stochastic, Transpose, Delay, Dynamics,
  NoteFilter, Gate, Humanize, Chance, Strum, Mutate, and ModSend (a
  modulation-bus sibling to BusSend — one module's CV can now modulate
  another's parameter live). Full list: **[docs/MODULES.md](docs/MODULES.md)**.
- **Correctness fixes** — live config edits (genre/scale/key changes
  mid-playback) no longer strand held notes on any module; the web UI's MIDI
  loop no longer blocks on a slow client.
- **Musicality pass** — velocity accents, humanization, phrase-aware
  generative lines, chord voice-leading, per-drum note config.
- **Routing** — MIDI-In brings live Force pads/keys into a chain; a named
  internal note bus (`bus_send`/`bus_in`) fans one lane's output into others.
- **Opal-style step editor** — paged (Trig/Vel/Prob/Gate/Ratchet/Nudge/Mod),
  mouse-drag painting of values across steps.
- **23 drum genres**, key/root picker on Gen/Marbles/Markov/Turing, an honest
  publisher/follower harmony badge, and per-module bypass/mute for A/B'ing a
  module live.

## Requirements

- Akai Force running **MockbaMod 4.51** (or compatible)
- SSH access to the Force, to make the binary executable and enable the AddOn

## Install

1. Download `RackForce-v0.3.0-beta.zip` from the [v0.3.0-beta release page](https://github.com/Devko/RackForce/releases/tag/v0.3.0-beta) (all versions: [Releases](../../releases)).
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

## Docs

- **[SPINE](docs/SPINE.md)** — the generative composition engine.
- **[MIDI setup](docs/MIDI.md)** — voice routing, the channel-16 control
  scheme, macro knobs, the web patch bay.
- **[Module catalog](docs/MODULES.md)** — all 36 modules, what each does.

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
