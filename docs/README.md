# Real Guitar Hero

A Guitar Hero-style game that teaches you to play **real guitar** using **interactive tablature**. Notes scroll down a highway (like Guitar Hero), but instead of 5 colored buttons, you see real fret numbers on 6 strings -- and you play them on an actual guitar.

## Core Concept

- **Interactive scrolling tabs** -- not sheet music, not simplified buttons
- **Real guitar input** -- plug in your guitar, the app hears what you play
- **Pitch detection** -- knows if you hit the right note, bend, slide, or chord
- **Gamified learning** -- score, streaks, difficulty progression, just like Guitar Hero

## Architecture Overview

```
┌─────────────┐    ┌──────────────────────┐    ┌─────────────────┐
│  Real Guitar │───>│  Fishman TriplePlay  │───>│  Desktop App    │
│  (w/ hex     │    │  (wireless MIDI,     │    │                 │
│   pickup)    │    │   per-string detect) │    │  - Tab Highway  │
│              │    └──────────────────────┘    │  - Note Matching│
│              │                               │  - Scoring      │
└─────────────┘                                └─────────────────┘
```

## Tech Stack (Planned)

| Layer              | Technology              | Why                                        |
|--------------------|-------------------------|--------------------------------------------|
| **Game Engine/UI** | Godot 4 or Electron     | Godot: fast 2D rendering. Electron: web tech flexibility |
| **Guitar Input**   | Fishman TriplePlay      | Hex pickup: per-string MIDI, exact fret/string detection, no pitch ambiguity |
| **MIDI Processing**| rtmidi / Web MIDI API   | Read MIDI note-on/off, pitch bend, velocity from TriplePlay |
| **Tab Parsing**    | alphaTab / TuxGuitar    | Parse Guitar Pro (.gp5/.gpx) files         |
| **Song Format**    | Guitar Pro tabs (.gpx)  | Richest guitar data: bends, slides, hammer-ons, timing |

## Hardware Requirements

### Primary Input: Fishman TriplePlay
- **Fishman TriplePlay** hex pickup (~$300) -- mounts on any guitar
- Sends MIDI wirelessly via USB receiver dongle
- Detects each string independently -- knows exactly which string and fret
- Tracks pitch bends, velocity, and note-on/off per string
- Near-zero latency MIDI output
- No pitch detection guesswork -- the hardware solves the hardest problem

### Also Needed
- Any electric or acoustic guitar (TriplePlay mounts to the body)
- Computer running Windows/Mac/Linux
- TriplePlay USB wireless receiver (included with the unit)

### Future: Fallback Audio Input
- USB audio interface + Aubio pitch detection for users without a TriplePlay
- Less precise (pitch ambiguity across strings) but still functional

## How It Works

1. **Load a song** -- import a Guitar Pro tab file
2. **Tab highway scrolls** -- 6 horizontal lanes (strings), fret numbers scroll toward a hit zone
3. **Play along** -- the app listens via audio input
4. **Real-time feedback** -- hit/miss detection, score, streak counter
5. **Practice tools** -- slow down sections, loop difficult parts, increase speed gradually

## Input Detection Strategy

### Primary: Fishman TriplePlay (MIDI)

The TriplePlay hex pickup sends **per-string MIDI data**, which eliminates pitch detection entirely:

- **Exact string + fret**: Each string is a separate MIDI channel -- no ambiguity
- **Note on/off**: Precise timing of when each note starts and stops
- **Pitch bend**: Detects bends and slides as MIDI pitch bend messages
- **Velocity**: Knows how hard you pick each string
- **Chords**: Trivial -- just read all 6 channels simultaneously

### Fallback: Audio Pitch Detection

For users without a TriplePlay, the app can fall back to audio-based detection:

- **Single notes**: YIN algorithm via Aubio -- fast and accurate
- **Chords**: Harmonic Product Spectrum or ML-based (Crepe/SPICE)
- **Limitation**: Cannot distinguish same pitch at different string/fret positions

## Song/Tab Format

Guitar Pro files (.gp5/.gpx) are the primary format because they contain:
- Note-by-note fret/string assignments
- Timing and tempo information
- Technique annotations (bends, slides, vibrato, palm mute, etc.)
- Difficulty can be derived from the tab data
- Massive community library of existing tabs

## Project Structure (Planned)

```
Real-Guitar-Hero/
├── docs/
│   ├── README.md          # This file
│   ├── plan.md            # Development plan & milestones
│   └── ideas.md           # Feature ideas & brainstorming
├── src/
│   ├── audio/             # Pitch detection, audio input
│   ├── tabs/              # Tab parsing (Guitar Pro format)
│   ├── game/              # Highway renderer, scoring, game loop
│   └── ui/                # Menus, settings, song select
├── assets/
│   ├── songs/             # Guitar Pro tab files
│   └── themes/            # Visual themes for the highway
└── tests/
```

## Getting Started

> **Status: Planning phase** -- see [plan.md](./plan.md) for development roadmap.

## Inspiration

- Guitar Hero / Rock Band (gameplay feel)
- Rocksmith (real guitar input concept)
- Yousician (learning progression)
- Clone Hero (open-source community charts)

The gap: Rocksmith is discontinued/closed-source, Yousician is subscription-based and limited. There's no good **open-source, tab-based, Guitar Hero-style real guitar trainer**. This project fills that gap.
