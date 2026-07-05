# MIDI setup

## 1. Route the voices

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

## 2. Live control — channel 16

Channel 16 carries knob moves and patch switches only; notes, clock and
transport stay on the voice channels above.

- **Knobs → parameters:** each module gets a block of 10 CCs, in patch order,
  starting at CC 16:

  ```
  module = (cc - 16) / 10      # which module (0 = first added)
  param  = (cc - 16) % 10      # which parameter of that module
  ```

  A CC value (0–127) is rescaled into that parameter's real range, so one
  knob sweep always covers the whole parameter. Start the daemon with
  `--verbose` to print the live CC → (module, parameter) map for the loaded
  patch — trust that over any static table, since the layout depends on
  which modules the patch contains.

- **Patch switching:** send a **Program Change on channel 16**; program
  number = patch index (0 = default).

- **Macro knobs:** create a MIDI track, set its output to `ModularRack` and
  channel to 16, then learn each knob to a CC from the `--verbose` map on
  the track's MIDI-Control page. Save the track as a Force template
  (`.xtk`) to reuse the mapping later — the same pattern Harpie4T uses for
  its own control surface.

## 3. Web patch bay

Open `http://<force-ip>:8088/` from a phone or laptop on the same network —
one card per module, sliders for every parameter, updating live in both
directions. Single self-contained page, no build step, works offline. A
loopback-only JSON API is also on `127.0.0.1:8099` for scripting.

See the [screenshots](../README.md#screenshots) in the main README for what
this looks like in practice.
