# Development Plan

## Phase 1: MIDI Input + Note Detection (Week 1-2)

**Goal:** Prove we can read precise note data from the Fishman TriplePlay in real-time.

- [ ] Set up Vite + React + TypeScript project
- [ ] Get Fishman TriplePlay MIDI input working via Web MIDI API
- [ ] Parse per-string MIDI channels (TriplePlay sends each string on channels 1-6)
- [ ] Implement `midiNoteToFret()`: `fret = midiNote - openStringMidi[string]`
- [ ] Handle note-on, note-off, pitch bend, velocity
- [ ] Build a live display showing: string, fret, velocity, pitch bend in real-time
- [ ] Add velocity threshold filter (ignore < 20 to prevent ghost notes)
- [ ] Test with single notes, chords, bends, slides across all strings
- [ ] Measure latency (target: <10ms MIDI input-to-display)

**Milestone:** Plug in guitar with TriplePlay, see exact string/fret update live.

## Phase 2: Fretboard Visualization (Week 3)

**Goal:** Build an interactive fretboard that shows what you're playing.

- [ ] Build fretboard component (Canvas or SVG, frets 0-24, strings 1-6)
- [ ] Map detected MIDI notes to fretboard positions in real-time
- [ ] Highlight currently playing note(s)
- [ ] Support multiple simultaneous notes (chords)
- [ ] Color-code by string (E=red, A=yellow, D=blue, G=orange, B=green, e=purple)

**Milestone:** Play a chord, see all notes light up on the fretboard instantly.

## Phase 3: Tab Loading + Scrolling Highway (Week 4-5)

**Goal:** Render scrolling tab notation Guitar Hero-style.

- [ ] Define simple JSON tab format (see README)
- [ ] Build tab parser (JSON first, Guitar Pro later)
- [ ] Build scrolling tab highway:
  - 6 horizontal lanes (one per string, low E at bottom)
  - Fret numbers scroll downward toward a "hit zone" line
- [ ] Sync scroll speed to song BPM
- [ ] Handle chords (multiple notes at same beat)
- [ ] Create a few sample tabs for testing (Smoke on the Water, etc.)

**Milestone:** Load a tab, watch it scroll. No input matching yet.

## Phase 4: Note Matching + Scoring (Week 6)

**Goal:** Connect MIDI input to the highway and score the player.

- [ ] Match MIDI input (string + fret from TriplePlay) against expected tab notes
  - TriplePlay gives exact string/fret -- no ambiguity, direct comparison
- [ ] Implement hit/miss detection with timing window:
  - Perfect: within ±30ms
  - Good: within ±60ms
  - OK: within ±100ms
  - Miss: outside window or wrong note
- [ ] Score system: points per note, multiplier for streaks
- [ ] Visual feedback: note markers change color on hit/miss
- [ ] Color-code: upcoming (gray), in zone (white), hit (green), missed (red)

**Milestone:** Playable prototype -- load a tab, play along, get scored.

## Phase 5: Practice Tools (Week 7-8)

**Goal:** Make it genuinely useful for learning.

- [ ] Speed control: play at 25%, 50%, 75%, 100%
- [ ] Section looping: set A-B points, loop until mastered
- [ ] Progressive speed: start slow, auto-increase as accuracy improves
- [ ] Fretboard preview: highlight upcoming notes on the neck diagram
- [ ] Difficulty filtering: hide some notes for simplified playthrough
- [ ] Section-by-section breakdown with per-section scores

**Milestone:** Loop a hard section at half speed until clean, watch speed auto-increase.

## Phase 6: Music Theory Integration (Week 9)

**Goal:** Add visual theory overlays to the fretboard.

- [ ] Scale/mode overlay: toggle "A minor pentatonic" and see it on the fretboard
- [ ] Chord shape display (CAGED system positions)
- [ ] Interval labeling relative to root note
- [ ] Key detection from played notes
- [ ] Suggested next chords based on current progression

**Milestone:** Toggle scale overlay while playing, see which notes are in/out of key.

## Phase 7: Guitar Pro Import (Week 10)

**Goal:** Load real Guitar Pro tabs from the massive community library.

- [ ] Integrate alphaTab or similar GP parser
- [ ] Parse .gp5/.gpx into internal note model
- [ ] Handle tempo changes, time signatures
- [ ] Extract technique annotations (bend, slide, hammer-on, pull-off, etc.)
- [ ] Display technique indicators on the highway

**Milestone:** Download a Guitar Pro tab, load it, play along with full scoring.

## Phase 8: Polish & UX (Week 11-12)

- [ ] Song selection menu
- [ ] Score history / progress tracking per song
- [ ] Settings: MIDI device selection, latency calibration, visual theme
- [ ] Calibration wizard: play open strings to verify TriplePlay mapping
- [ ] Dark mode / themes
- [ ] Performance stats: accuracy %, notes hit, longest streak
- [ ] Mobile-friendly layout (tablet for music stand use)

---

## Key Technical Risks

| Risk | Mitigation |
|------|------------|
| TriplePlay wireless latency | Test and measure; ~5-7ms expected, acceptable |
| TriplePlay tracking on fast passages | Tune sensitivity; start with moderate-tempo songs |
| Palm muting triggers ghost notes | Velocity threshold filter (ignore < 20) |
| Guitar Pro format is complex | Use existing parser libs (alphaTab), start with simple tabs |
| TriplePlay discontinued/hard to find | Works standalone via standard MIDI; alternatives: any hex pickup + MIDI converter |
| Web MIDI not supported in all browsers | Target Chrome/Edge; Firefox/Safari don't support Web MIDI yet |

## Decision Log

| Decision | Options Considered | Choice | Reasoning |
|----------|-------------------|--------|-----------|
| Platform | Desktop (Godot/Electron), Web (React+Vite) | Web (React + Vite) | Fastest to prototype, Web MIDI API built-in, no install needed |
| Guitar input | Audio pitch detection, Fishman TriplePlay, Jamstik, Roland GK-3 | Fishman TriplePlay | Per-string MIDI eliminates pitch ambiguity; exact fret/string; ~5ms latency |
| Rendering | Three.js, Canvas API, SVG, DOM | Canvas API | Fast 2D rendering, good for scrolling highway + fretboard |
| Tab format | Guitar Pro only, MIDI, custom JSON | JSON (start) + Guitar Pro (later) | JSON for fast prototyping, GP for real songs later |
| State management | Redux, Zustand, Jotai, vanilla React | Zustand | Lightweight, great for real-time state updates |
| Theory lib | Custom, tonal.js | tonal.js | Proven, comprehensive, no need to reinvent |
