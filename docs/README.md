# Real Guitar Hero

A real-time guitar learning tool that takes MIDI input from your guitar via a Fishman TriplePlay hex pickup, detects what you're playing, and visualizes it against interactive tabs -- like Guitar Hero but for real tabs on a real guitar.

---

## Why This Exists

I have a lot of technical skill on guitar, but I don't know how to put things together -- scales, theory, chord progressions. This tool lets me:

- **Load any tab** and see it scroll toward me Guitar Hero-style
- **See exactly which notes I'm hitting** in real-time on a fretboard diagram
- **Instantly spot wrong notes** -- visual feedback means I can just move my finger over
- **Learn music theory visually** -- scales, intervals, and chord shapes overlaid on the fretboard
- **Slow down sections** -- loop a tricky part at half speed until I nail it

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    Browser (Vite + React)            │
│                                                     │
│  ┌──────────┐    ┌──────────┐    ┌───────────────┐  │
│  │  Input    │    │  Engine   │    │  Visualizer   │  │
│  │  Layer    │───>│  Layer    │───>│  Layer        │  │
│  │          │    │          │    │               │  │
│  │ Web MIDI │    │ Note     │    │ Fretboard     │  │
│  │ (primary)│    │ Matching │    │ Tab Scroller  │  │
│  │ Web Audio│    │ Scoring  │    │ Scale Overlay │  │
│  │ (fallback│    │ Timing   │    │ Score Display │  │
│  └──────────┘    └──────────┘    └───────────────┘  │
│                                                     │
│  ┌──────────────────────────────────────────────┐   │
│  │  Theory Module                                │   │
│  │  Scales, Intervals, Chord Shapes, Progressions│   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

### Tech Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| **Frontend** | React + TypeScript + Vite | Fast dev, hot reload, strong typing |
| **Rendering** | Canvas API | Scrolling tab highway + fretboard visualization |
| **Guitar Input** | Fishman TriplePlay (Web MIDI API) | Hex pickup: per-string MIDI, exact fret/string, no pitch ambiguity |
| **Fallback Input** | Web Audio API + pitchy | For users without a TriplePlay |
| **Tab Parsing** | Guitar Pro parser + custom JSON | .gp5/.gpx files + simple JSON format |
| **State** | Zustand | Lightweight, perfect for real-time |
| **Theory** | tonal.js | Scales, chords, intervals |

---

## Hardware

### Primary: Fishman TriplePlay Connect (~$300)

Wireless hex MIDI pickup -- one sensor per string, clips onto any electric guitar.

- **Signal flow:** Guitar strings → hex pickup (6 sensors) → onboard MIDI conversion → wireless → USB dongle → browser via Web MIDI API
- **Per-string detection:** Each string sends on a separate MIDI channel (1-6)
- **Fret calculation:** `fret = midiNote - openStringMidi[string]` -- exact, every time
- **Latency:** ~5-7ms total (imperceptible)
- **Tracks:** Note on/off, velocity, pitch bend (bends/slides)

See [hardware-setup.md](./hardware-setup.md) for full setup guide, MIDI integration code, and troubleshooting.

### Fallback: USB Audio Interface (~$60)

Scarlett Solo, Behringer UM2, etc. -- pitch detection via Web Audio API. Less precise (can't distinguish same pitch at different string/fret positions) but works with any guitar, no special hardware.

---

## Core Features

### 1. Interactive Tab Highway (Guitar Hero Mode)
- 6 horizontal lanes (one per string), fret numbers scroll toward a "hit zone"
- Color-coded: upcoming (gray), in the zone (white), hit (green), missed (red)
- Adjustable scroll speed / BPM
- Loop sections with A-B repeat

### 2. Real-Time Fretboard Visualization
- Full fretboard diagram (frets 0-24, strings 1-6)
- Real-time highlight of detected notes
- Overlay modes:
  - **Tab mode:** Shows where to put fingers for the current section
  - **Scale mode:** Highlights all notes in the current scale/key
  - **Interval mode:** Shows intervals relative to root note
  - **Chord mode:** Shows chord shape and name

### 3. Note Matching + Scoring
- TriplePlay gives exact string + fret -- direct comparison against tab
- Timing windows: Perfect (±30ms), Good (±60ms), OK (±100ms), Miss
- Score, streak counter, accuracy percentage
- Visual + optional audio feedback on hit/miss

### 4. Music Theory Engine
- All major/minor scales, modes (Ionian through Locrian), pentatonics, blues
- Chord construction: triads, 7ths, extended, altered
- Interval recognition: "that's a minor 3rd from the root"
- Key detection from played notes
- Suggested next chords based on current key/progression (diatonic harmony)

### 5. Practice Tools
- **Speed control:** 25%, 50%, 75%, 100%
- **Section looping:** A-B repeat until clean
- **Progressive speed:** Start slow, auto-increase as accuracy improves
- **Fretboard preview:** Highlight upcoming notes on the neck diagram
- **Difficulty filtering:** Hide some notes for simplified playthrough

---

## Tab Format

### Guitar Pro (.gp5/.gpx)
Primary format -- richest guitar-specific data (bends, slides, vibrato, palm mute, etc.). Massive community library of existing tabs.

### Simple JSON (custom)
For quick prototyping and custom exercises:

```json
{
  "title": "Smoke on the Water - Main Riff",
  "artist": "Deep Purple",
  "bpm": 112,
  "tuning": ["E2", "A2", "D3", "G3", "B3", "E4"],
  "measures": [
    {
      "notes": [
        { "string": 4, "fret": 0, "time": 0, "duration": 0.5 },
        { "string": 3, "fret": 0, "time": 0, "duration": 0.5 },
        { "string": 4, "fret": 3, "time": 0.5, "duration": 0.5 },
        { "string": 3, "fret": 3, "time": 0.5, "duration": 0.5 },
        { "string": 4, "fret": 5, "time": 1.0, "duration": 0.75 },
        { "string": 3, "fret": 5, "time": 1.0, "duration": 0.75 }
      ]
    }
  ]
}
```

---

## Music Theory Reference

### The 7 Modes (built from major scale)
| Mode | Formula | Feel |
|------|---------|------|
| Ionian (Major) | 1 2 3 4 5 6 7 | Happy, bright |
| Dorian | 1 2 b3 4 5 6 b7 | Jazzy minor |
| Phrygian | 1 b2 b3 4 5 b6 b7 | Spanish, dark |
| Lydian | 1 2 3 #4 5 6 7 | Dreamy, floating |
| Mixolydian | 1 2 3 4 5 6 b7 | Bluesy major |
| Aeolian (Minor) | 1 2 b3 4 5 b6 b7 | Sad, natural minor |
| Locrian | 1 b2 b3 4 b5 b6 b7 | Unstable, diminished |

### Pentatonic Scales
- **Minor Pentatonic:** 1 b3 4 5 b7 -- the "guitar solo" scale
- **Major Pentatonic:** 1 2 3 5 6 -- country, pop, bright solos
- **Blues Scale:** 1 b3 4 #4/b5 5 b7 -- minor pentatonic + blue note

### Standard Tuning
| String | Note | MIDI | Frequency |
|--------|------|------|-----------|
| 6 (low E) | E2 | 40 | 82.41 Hz |
| 5 | A2 | 45 | 110.00 Hz |
| 4 | D3 | 50 | 146.83 Hz |
| 3 | G3 | 55 | 196.00 Hz |
| 2 | B3 | 59 | 246.94 Hz |
| 1 (high E) | E4 | 64 | 329.63 Hz |

### Common Chord Progressions
| Name | Numerals | Example in C | Usage |
|------|----------|-------------|-------|
| Pop | I - V - vi - IV | C G Am F | Pop, rock |
| Blues | I - IV - V | C F G | Blues, rock |
| Jazz ii-V-I | ii - V - I | Dm G C | Jazz standards |
| Sad | vi - IV - I - V | Am F C G | Ballads |
| 12-Bar Blues | I I I I / IV IV I I / V IV I V | -- | Blues |

---

## Project Structure

```
Real-Guitar-Hero/
├── docs/
│   ├── README.md              # This file
│   ├── plan.md                # Development plan & milestones
│   ├── hardware-setup.md      # Fishman TriplePlay setup guide
│   └── ideas.md               # Feature ideas & brainstorming
├── src/
│   ├── input/
│   │   ├── MidiInput.ts       # Web MIDI API (TriplePlay)
│   │   ├── AudioInput.ts      # Web Audio API (fallback)
│   │   └── PitchDetector.ts   # Note detection from audio
│   ├── engine/
│   │   ├── NoteMapper.ts      # MIDI/pitch -> fretboard position
│   │   ├── TabPlayer.ts       # Tab playback + timing
│   │   ├── Scorer.ts          # Accuracy scoring
│   │   └── TheoryEngine.ts    # Scales, chords, intervals
│   ├── visualizer/
│   │   ├── Fretboard.tsx      # Fretboard component
│   │   ├── TabScroller.tsx    # Guitar Hero scrolling view
│   │   ├── ScaleOverlay.tsx   # Scale/mode overlay
│   │   └── ScoreDisplay.tsx   # Real-time scoring
│   ├── tabs/
│   │   ├── parser.ts          # Tab file parser (GP + JSON)
│   │   └── samples/           # Example tabs
│   ├── App.tsx
│   └── main.tsx
├── assets/
│   ├── songs/                 # Tab files
│   └── themes/                # Visual themes
├── package.json
├── tsconfig.json
├── vite.config.ts
└── tests/
```

---

## Getting Started

> **Status: Planning phase** -- see [plan.md](./plan.md) for development roadmap.

## Inspiration

- Guitar Hero / Rock Band (gameplay feel)
- Rocksmith (real guitar input concept)
- Yousician (learning progression)
- Clone Hero (open-source community charts)

The gap: Rocksmith is discontinued/closed-source, Yousician is subscription-based and limited. There's no good **open-source, tab-based, Guitar Hero-style real guitar trainer**. This project fills that gap.

## References

- [Web MIDI API](https://www.w3.org/TR/webmidi/)
- [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
- [pitchy (pitch detection)](https://github.com/ianprime0509/pitchy)
- [alphaTab (Guitar Pro renderer)](https://www.alphatab.net/)
- [tonal.js (music theory)](https://github.com/tonaljs/tonal)
- [Fishman TriplePlay](https://www.fishman.com/tripleplay/)
