# Development Plan

## Phase 1: MIDI Input Foundation (Week 1-2)

**Goal:** Prove we can read precise note data from the Fishman TriplePlay in real-time.

- [ ] Set up project scaffold (Godot 4 or Electron -- decide after prototype)
- [ ] Get Fishman TriplePlay MIDI input working via USB receiver
- [ ] Use rtmidi (native) or Web MIDI API (Electron) to read MIDI messages
- [ ] Parse per-string MIDI channels (TriplePlay sends each string on a separate channel)
- [ ] Build a simple display showing: string, fret, velocity, pitch bend in real-time
- [ ] Test with single notes, chords, bends, slides across all strings
- [ ] Measure latency (target: <10ms MIDI input-to-display)

**Deliverable:** A live monitor that shows exactly which string/fret you're pressing in real-time.

## Phase 2: Tab Parsing (Week 3)

**Goal:** Load a Guitar Pro file and extract note/timing data.

- [ ] Research Guitar Pro format parsers:
  - alphaTab (TypeScript) -- if going Electron route
  - TuxGuitar libraries (Java) -- reference implementation
  - rust-gpx or similar native parsers
- [ ] Parse a .gp5/.gpx file into internal data model:
  ```
  Song → Track → Measure → Beat → Note
  Each Note: { string, fret, duration, technique, timing_ms }
  ```
- [ ] Handle tempo changes, time signature changes
- [ ] Extract technique annotations (bend, slide, hammer-on, pull-off)
- [ ] Write unit tests with known tab files

**Deliverable:** CLI tool that dumps a Guitar Pro file as timestamped note events.

## Phase 3: The Highway (Week 4-5)

**Goal:** Render scrolling tab notation Guitar Hero-style.

- [ ] Design the highway layout:
  - 6 horizontal lanes (one per string, low E at bottom)
  - Fret numbers scroll downward toward a "hit zone" line
  - Color-code strings (standard: E=red, A=yellow, D=blue, G=orange, B=green, e=purple)
- [ ] Implement smooth scrolling synchronized to song timing
- [ ] Render note markers with fret numbers
- [ ] Add a "hit zone" indicator where notes should be played
- [ ] Support variable scroll speed (tied to song tempo)
- [ ] Handle chords (multiple notes at same beat)

**Deliverable:** Visual highway that scrolls through a parsed tab file (no input detection yet).

## Phase 4: Connect Input to Highway (Week 6)

**Goal:** Match detected pitch to expected notes and score the player.

- [ ] Match MIDI input (string + fret from TriplePlay) against expected notes from tab
  - TriplePlay gives exact string/fret -- no ambiguity, direct comparison
- [ ] Implement hit/miss detection with timing window:
  - Perfect: within ±30ms
  - Good: within ±60ms
  - OK: within ±100ms
  - Miss: outside window or wrong note
- [ ] Score system: points per note, multiplier for streaks
- [ ] Visual feedback: note markers change color on hit/miss
- [ ] Audio feedback: optional miss sound effect

**Deliverable:** Playable prototype -- load a tab, play along, get scored.

## Phase 5: Practice Tools (Week 7-8)

**Goal:** Make it actually useful for learning.

- [ ] Speed control: slow down to 25/50/75% speed
- [ ] Section looping: select a range of measures, loop until mastered
- [ ] Progressive speed: start slow, auto-increase speed as accuracy improves
- [ ] Note preview: highlight upcoming notes on a fretboard diagram
- [ ] Fretboard visualization: show a guitar neck with dots where fingers should go
- [ ] Difficulty filtering: hide some notes for simplified playthrough

**Deliverable:** A practice mode that genuinely helps learn difficult passages.

## Phase 6: Polish & UX (Week 9-10)

- [ ] Song selection menu
- [ ] Score history / progress tracking per song
- [ ] Settings: audio input device, latency calibration, visual theme
- [ ] Calibration wizard: play open strings to set detection baseline
- [ ] Import songs from local Guitar Pro files
- [ ] Performance stats: accuracy %, notes hit, longest streak

## Future Ideas (Post-MVP)

- Online tab library integration
- Multiplayer / leaderboards
- Technique-specific lessons (bending, vibrato, tapping)
- AI-generated practice exercises targeting weak areas
- Standalone hardware version (Bela / Daisy)
- Support for bass guitar (4-string mode)
- Amp sim integration (play through the app with effects)

## Key Technical Risks

| Risk | Mitigation |
|------|------------|
| TriplePlay wireless latency | Test and measure; fall back to wired MIDI if needed |
| TriplePlay tracking on fast passages | May need to tune sensitivity; start with moderate-tempo songs |
| Guitar Pro format is complex/undocumented | Use existing parser libs, start with simple tabs |
| TriplePlay discontinued/hard to find | Works standalone via MIDI; alternatives: hex pickups + MIDI converter |

## Decision Log

| Decision | Options Considered | Choice | Reasoning |
|----------|-------------------|--------|-----------|
| Game engine | Godot 4, Electron, Love2D, SDL | TBD | Prototype both Godot and Electron, pick based on feel |
| Guitar input | Audio pitch detection, Fishman TriplePlay, Jamstik, hex pickup + MIDI | Fishman TriplePlay | Per-string MIDI eliminates pitch ambiguity; exact fret/string detection; near-zero latency |
| Tab format | Guitar Pro, MIDI, MusicXML, custom | Guitar Pro | Most guitar-specific data, huge community library |
| MIDI library | rtmidi, Web MIDI API, portmidi | TBD | rtmidi for native; Web MIDI for Electron. Decide with engine choice |
