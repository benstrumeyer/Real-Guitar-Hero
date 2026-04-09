# Hardware Setup -- Fishman TriplePlay Connect

## Overview

The Fishman TriplePlay Connect is a wireless hex MIDI pickup that mounts on any electric guitar. It tracks each string independently and sends MIDI over USB -- per-string note detection with ~5ms latency.

---

## What You Need

| Part | Price | Notes |
|------|-------|-------|
| Fishman TriplePlay Connect | ~$300 | Wireless hex pickup + USB receiver dongle |
| Any electric guitar | -- | Mounts on most guitars without modification |
| USB port on computer | -- | For the wireless receiver dongle |
| **Total** | **~$300** | (plus guitar you already own) |

That's it. No soldering, no mods, no drivers to install. Plug in the dongle, clip on the pickup, play.

---

## How It Works

### Hex Pickup
- **6 individual pickups** -- one under each string
- Each pickup tracks its string independently
- This means it knows which STRING you played, not just the pitch
- Detects: note-on, note-off, pitch bend, velocity (how hard you pick)

### Signal Flow
```
Guitar strings vibrate
       |
Hex pickup (one sensor per string) detects vibration
       |
Onboard processor converts analog -> MIDI (per-string)
       |
Wireless transmitter sends to USB dongle
       |
Computer receives MIDI via Web MIDI API
       |
Guitar Coach processes note events
```

### MIDI Output Format
The TriplePlay sends standard MIDI messages:
- **Note On:** `[0x90 + channel, note, velocity]` -- a string was plucked
- **Note Off:** `[0x80 + channel, note, 0]` -- string stopped
- **Pitch Bend:** `[0xE0 + channel, LSB, MSB]` -- string bend detected
- **Channel per string:** Strings 1-6 map to MIDI channels 1-6 (configurable)

This per-channel setup is key -- it tells us which string produced each note.

---

## Installation

### Physical Setup
1. **Mount the pickup:** Clip the hex pickup onto the guitar between the neck pickup and the end of the fretboard. It uses a removable adhesive mount -- no drilling.
2. **Position:** The pickup should sit ~2-3mm below the strings, centered under all 6 strings.
3. **Plug in the dongle:** Insert the USB wireless receiver into your computer.
4. **Power on:** The pickup has a small power button. LED indicates connection status.

### Software Setup
1. The TriplePlay appears as a standard USB MIDI device -- no special drivers needed on modern OS.
2. Verify it shows up: open any MIDI monitor app, or check in the browser via Web MIDI API.
3. In Guitar Coach, select "Fishman TriplePlay" as the MIDI input device.

### Calibration
1. **String sensitivity:** Each string's pickup sensitivity can be adjusted via the TriplePlay app. Start with defaults.
2. **Tracking speed:** The TriplePlay has "speed vs accuracy" tuning. For our use case, **prioritize accuracy** -- we care more about correct notes than sub-millisecond speed.
3. **Channel mapping:** Ensure strings 1-6 map to MIDI channels 1-6. This is the default.

---

## Web MIDI API Integration

### Detecting the TriplePlay

```typescript
async function connectTriplePlay(): Promise<MIDIInput | null> {
  const access = await navigator.requestMIDIAccess();

  for (const input of access.inputs.values()) {
    if (input.name?.includes('TriplePlay') || input.name?.includes('Fishman')) {
      return input;
    }
  }

  // Fall back to first available MIDI input
  const firstInput = access.inputs.values().next().value;
  return firstInput || null;
}
```

### Reading Note Events

```typescript
function handleMIDIMessage(event: MIDIMessageEvent) {
  const [status, note, velocity] = event.data;
  const messageType = status & 0xF0;
  const channel = (status & 0x0F) + 1;  // 1-indexed

  // Channel = string number (1-6)
  const string = channel;

  if (messageType === 0x90 && velocity > 0) {
    // Note On
    const fret = midiNoteToFret(note, string);
    onNoteOn(string, fret, note, velocity);
  } else if (messageType === 0x80 || (messageType === 0x90 && velocity === 0)) {
    // Note Off
    onNoteOff(string, note);
  } else if (messageType === 0xE0) {
    // Pitch Bend (string bend)
    const bendValue = (event.data[2] << 7) | event.data[1];
    onPitchBend(string, bendValue);
  }
}

function midiNoteToFret(midiNote: number, string: number): number {
  // Standard tuning open string MIDI notes
  const openStringMidi: Record<number, number> = {
    1: 64,  // E4
    2: 59,  // B3
    3: 55,  // G3
    4: 50,  // D3
    5: 45,  // A2
    6: 40,  // E2
  };

  return midiNote - openStringMidi[string];
}
```

### Full Connection Example

```typescript
async function initMIDI() {
  if (!navigator.requestMIDIAccess) {
    console.error('Web MIDI API not supported in this browser');
    return;
  }

  try {
    const access = await navigator.requestMIDIAccess();

    // Listen for device connections
    access.onstatechange = (event) => {
      console.log('MIDI device changed:', event.port.name, event.port.state);
    };

    // Connect to TriplePlay
    const input = await connectTriplePlay();
    if (input) {
      input.onmidimessage = handleMIDIMessage;
      console.log(`Connected to: ${input.name}`);
    } else {
      console.warn('No MIDI device found');
    }
  } catch (err) {
    console.error('MIDI access denied:', err);
  }
}
```

---

## String-to-Fret Mapping

The TriplePlay sends MIDI note numbers. Combined with the channel (= string), we can calculate the exact fret:

| String | Open Note | MIDI # | Fret 1 | Fret 5 | Fret 12 |
|--------|-----------|--------|--------|--------|---------|
| 1 (high E) | E4 | 64 | F4 (65) | A4 (69) | E5 (76) |
| 2 | B3 | 59 | C4 (60) | E4 (64) | B4 (71) |
| 3 | G3 | 55 | G#3 (56) | C4 (60) | G4 (67) |
| 4 | D3 | 50 | D#3 (51) | G3 (55) | D4 (62) |
| 5 | A2 | 45 | A#2 (46) | D3 (50) | A3 (57) |
| 6 (low E) | E2 | 40 | F2 (41) | A2 (45) | E3 (52) |

**Formula:** `fret = midiNote - openStringMidi[string]`

Since the TriplePlay sends on separate channels per string, we always know which string -- so fret calculation is unambiguous.

---

## Latency

| Stage | Latency |
|-------|---------|
| String vibration -> pickup detection | ~2ms |
| Onboard MIDI conversion | ~2-3ms |
| Wireless transmission | ~1ms |
| USB MIDI to browser | <1ms |
| **Total** | **~5-7ms** |

5-7ms is imperceptible for practice purposes. For reference, the speed of sound from amp to your ears at 3 meters is ~9ms.

---

## Limitations & Workarounds

| Limitation | Impact | Workaround |
|-----------|--------|------------|
| Slight tracking delay on low strings | Low E can take ~10ms to detect pitch | Acceptable for practice |
| Palm muting can trigger false notes | Ghost notes in MIDI output | Velocity threshold filter -- ignore velocity < 20 |
| Wireless battery life | ~8 hours | USB charging, lasts multiple practice sessions |
| Slides send multiple note-on events | Rapid same-string events | Debounce rapid same-string events in software |

---

## Troubleshooting

- **No MIDI input detected:** Check that the dongle is plugged in and pickup is powered on (LED should be solid/blinking). Try a different USB port.
- **Wrong notes detected:** Recalibrate string sensitivity in the TriplePlay app. Make sure the pickup is centered under all strings.
- **Ghost notes / false triggers:** Add a velocity threshold in the software (ignore velocity < 15-20). This filters out incidental string touches.
- **High latency:** Make sure you're using Web MIDI API directly (not routing through DAW software). Direct MIDI is fastest.
- **Browser doesn't support Web MIDI:** Use Chrome or Edge. Firefox and Safari don't support Web MIDI API yet.

---

## Resources

- [Fishman TriplePlay Connect](https://www.fishman.com/tripleplay/)
- [Web MIDI API Spec](https://www.w3.org/TR/webmidi/)
- [Web MIDI Browser Support](https://caniuse.com/midi)
- [MIDI Note Number Reference](https://www.inspiredacoustics.com/en/MIDI_note_numbers_and_center_frequencies)
