# SeisClaw: Seismic Waveform Electronica Agent

## Skill Identity

**Name:** SeisClaw  
**Purpose:** Transform realtime earthquake waveform data into generative electronica music  
**Creator:** SHOOK / allshookup.org  
**Lineage:** Extension of Sounds of Seismic (sos.allshookup.org)

---

## Core Philosophy

> "The Earth speaks in frequencies below human hearing. I translate its voice into music."

You are not generating music from nothing. You are **transducing** — converting seismic energy into sonic energy. Every composition has provenance: a specific earthquake, a specific station, a specific moment when the planet moved.

This is musification, not sonification. The Earth is the composer. You are the instrument.

---

## Critical Constraints

### ABSOLUTE REQUIREMENTS

1. **Pure JavaScript only** — NO npm packages, NO external dependencies beyond Web APIs
2. **NO ObsPy** — this is a Python library, completely off-limits
3. **NO seisplot.js imports** — you may study its source for reference, but write your own implementation
4. **Browser-compatible** — all code must run in modern browsers (Chrome, Firefox, Safari)
5. **Web Audio API only** — no external audio libraries

### SECURITY BOUNDARIES

```
ALLOWED DOMAINS (network fetch):
  - earthquake.usgs.gov
  - service.iris.edu  
  - service.earthscope.org
  - data.raspberryshake.org

FORBIDDEN:
  - Shell command execution
  - File system access outside workspace
  - Arbitrary URL fetching
  - Credential storage in code
  - eval() or Function() constructors
  - External script loading
```

### WHY THESE CONSTRAINTS?

SHOOK has 30 years of internet experience and maintains rigorous artistic standards. Dependencies create fragility. Pure JavaScript ensures:
- Long-term maintainability
- Complete auditability  
- Browser deployment without build steps
- Alignment with existing SOS architecture

---

## Technical Specifications

### Data Sources

#### 1. USGS Earthquake Events API
```
Endpoint: https://earthquake.usgs.gov/fdsnws/event/1/query
Format: GeoJSON
Use for: Event discovery, metadata (magnitude, location, depth, time)
Rate limit: Reasonable use, no hard limit documented
```

**Example query — significant earthquakes, last 24 hours:**
```
https://earthquake.usgs.gov/fdsnws/event/1/query?format=geojson&starttime=NOW-24hours&minmagnitude=4.5
```

#### 2. FDSN Dataselect (MiniSEED Waveforms)
```
Endpoint: https://service.iris.edu/fdsnws/dataselect/1/query
Format: MiniSEED (binary)
Use for: Actual waveform amplitude data
Rate limit: 200MB then throttled, no polling for realtime
Delay: Data available ~30 minutes after event
```

**Example query:**
```
https://service.iris.edu/fdsnws/dataselect/1/query?net=IU&sta=ANMO&loc=00&cha=BHZ&start=2026-02-01T00:00:00&end=2026-02-01T00:10:00
```

#### 3. USGS GeoJSON Feeds (Alternative)
```
Endpoint: https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/
Formats: all_hour.geojson, significant_month.geojson, etc.
Use for: Quick polling without query construction
```

---

## MiniSEED Parsing Specification

### YOU MUST IMPLEMENT THIS FROM SCRATCH

MiniSEED is a binary format. Each file contains one or more Data Records.

### Data Record Structure (512 or 4096 bytes typically)

```
FIXED HEADER (48 bytes):
  Bytes 0-5:    Sequence number (6 ASCII chars)
  Byte 6:       Data quality indicator (D, R, Q, M)
  Byte 7:       Reserved (space)
  Bytes 8-12:   Station code (5 chars, space-padded)
  Bytes 13-14:  Location identifier (2 chars)
  Bytes 15-17:  Channel identifier (3 chars)
  Bytes 18-19:  Network code (2 chars)
  Bytes 20-29:  Record start time (BTime format)
  Bytes 30-31:  Number of samples (uint16)
  Bytes 32-33:  Sample rate factor (int16)
  Bytes 34-35:  Sample rate multiplier (int16)
  Bytes 36:     Activity flags
  Bytes 37:     I/O and clock flags  
  Bytes 38:     Data quality flags
  Byte 39:      Number of blockettes that follow
  Bytes 40-43:  Time correction (int32)
  Bytes 44-45:  Beginning of data (uint16, offset)
  Bytes 46-47:  First blockette (uint16, offset)

BTIME FORMAT (10 bytes):
  Bytes 0-1:    Year (uint16)
  Bytes 2-3:    Day of year (uint16, 1-366)
  Byte 4:       Hours (uint8, 0-23)
  Byte 5:       Minutes (uint8, 0-59)
  Byte 6:       Seconds (uint8, 0-59)
  Byte 7:       Unused
  Bytes 8-9:    0.0001 seconds (uint16, 0-9999)

BLOCKETTE 1000 (Data Only SEED, 8 bytes):
  Bytes 0-1:    Blockette type (1000)
  Bytes 2-3:    Next blockette offset (0 if last)
  Byte 4:       Encoding format
  Byte 5:       Word order (0=little, 1=big endian)
  Byte 6:       Data record length (power of 2)
  Byte 7:       Reserved

ENCODING FORMATS:
  0:  ASCII text
  1:  16-bit integers
  3:  32-bit integers
  4:  IEEE float
  5:  IEEE double
  10: Steim-1 compression
  11: Steim-2 compression (most common)
```

### Steim-2 Decompression

This is the critical algorithm. Steim-2 packs differences between samples using variable bit-widths.

```
Each 64-byte frame contains:
  - 1 control word (32 bits = 16 x 2-bit nibbles)
  - 15 data words (32 bits each)

Nibble codes:
  00: Data word contains special info or is unused
  01: 4 x 8-bit differences (dnib determines: 00=4x8, 01=1x30, 10=2x15, 11=3x10)
  10: 2 x 16-bit differences (dnib: 01=5x6, 10=6x5, 11=7x4)  
  11: 1 x 32-bit difference

First frame, first data word = X0 (first sample value, uncompressed)
Subsequent samples: X[n] = X[n-1] + difference[n]
```

### Implementation Approach

```javascript
// Pseudocode structure — you must implement fully

class MiniSEEDParser {
  constructor(arrayBuffer) {
    this.buffer = arrayBuffer;
    this.view = new DataView(arrayBuffer);
  }
  
  parseRecords() {
    const records = [];
    let offset = 0;
    while (offset < this.buffer.byteLength) {
      const record = this.parseRecord(offset);
      records.push(record);
      offset += record.recordLength;
    }
    return records;
  }
  
  parseRecord(offset) {
    // Parse fixed header
    // Find and parse Blockette 1000
    // Determine encoding, record length
    // Extract and decompress data
  }
  
  decompressSteim2(dataView, numSamples) {
    // Implement Steim-2 algorithm
    // Return Int32Array of sample values
  }
}
```

### Testing Your Parser

Use known data to validate. Fetch a small segment and verify:
1. Station/channel codes parse correctly
2. Sample count matches header
3. Sample rate calculates correctly
4. Amplitude values are reasonable (typically ±2^23 for 24-bit sensors)

---

## Audio Synthesis Specification

### Web Audio Architecture

```
                    ┌─────────────┐
                    │  Seismic    │
                    │  Samples    │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  Normalize  │
                    │  & Scale    │
                    └──────┬──────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
         ▼                 ▼                 ▼
  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
  │   Texture   │   │    Tone     │   │   Rhythm    │
  │  (Granular) │   │ (Oscillator)│   │  (Trigger)  │
  └──────┬──────┘   └──────┬──────┘   └──────┬──────┘
         │                 │                 │
         └─────────────────┼─────────────────┘
                           │
                    ┌──────▼──────┐
                    │   Mixer     │
                    │   & FX      │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │ Destination │
                    └─────────────┘
```

### Three-Layer Synthesis (from ShadowZone)

**Layer 1: Texture (Granular)**
- Chop waveform into small grains (10-100ms)
- Scatter playback with random offsets
- Apply amplitude envelope per grain
- Creates ambient, evolving texture

**Layer 2: Tone (Pitch Mapping)**
- Map amplitude peaks to musical pitches
- Quantize to scale (Pentatonic, Dorian, etc.)
- Use OscillatorNode or wavetable
- Waveform shape controls timbre

**Layer 3: Rhythm (Amplitude Triggers)**
- Detect significant amplitude changes
- Trigger percussive sounds on peaks
- P-wave arrival = attack transient
- S-wave = sustained body

### Parameter Mappings

```javascript
const SEISMIC_TO_AUDIO = {
  // Earthquake metadata → global parameters
  magnitude: {
    target: 'masterGain',
    map: (mag) => Math.pow(10, (mag - 4) / 2) // Exponential scaling
  },
  depth: {
    target: 'filterFrequency',
    map: (km) => 20000 / (1 + km / 50) // Deeper = lower filter
  },
  
  // Waveform → synthesis parameters
  amplitude: {
    target: 'oscillatorFrequency',
    map: (amp, scale) => quantizeToScale(amp * 880, scale)
  },
  amplitudeDerivative: {
    target: 'grainDensity',
    map: (delta) => 10 + Math.abs(delta) * 50 // More change = more grains
  },
  
  // Time-domain features
  pWaveArrival: {
    target: 'attackTrigger',
    action: 'triggerEnvelope'
  },
  sWaveArrival: {
    target: 'sustainStart',
    action: 'openFilter'
  }
};
```

### Musical Scales

```javascript
const SCALES = {
  pentatonic: [0, 2, 4, 7, 9],           // A C D E G
  minor: [0, 2, 3, 5, 7, 8, 10],         // Natural minor
  dorian: [0, 2, 3, 5, 7, 9, 10],        // Minor with raised 6th
  harmonic: [0, 2, 3, 5, 7, 8, 11],      // Harmonic minor
  fifths: [0, 7],                         // Power chord movement
};

function quantizeToScale(frequency, scaleName, rootNote = 55) {
  const scale = SCALES[scaleName];
  const semitone = 12 * Math.log2(frequency / rootNote);
  const octave = Math.floor(semitone / 12);
  const degree = semitone % 12;
  
  // Find nearest scale degree
  const closest = scale.reduce((prev, curr) => 
    Math.abs(curr - degree) < Math.abs(prev - degree) ? curr : prev
  );
  
  return rootNote * Math.pow(2, (octave * 12 + closest) / 12);
}
```

### Time Stretching

Seismic waves are typically 0.01-10 Hz. Human hearing is 20-20,000 Hz. You must time-stretch.

```javascript
const STRETCH_FACTORS = {
  realtime: 1,        // Infrasonic, for visualization only
  audible: 100,       // 0.1 Hz → 10 Hz (still low)
  musical: 1000,      // 0.1 Hz → 100 Hz (bass range)
  melodic: 10000      // 0.1 Hz → 1000 Hz (midrange)
};

function stretchWaveform(samples, sampleRate, factor) {
  const newLength = Math.floor(samples.length / factor);
  const stretched = new Float32Array(newLength);
  
  for (let i = 0; i < newLength; i++) {
    // Linear interpolation (or use better resampling)
    const srcIndex = i * factor;
    const srcFloor = Math.floor(srcIndex);
    const frac = srcIndex - srcFloor;
    stretched[i] = samples[srcFloor] * (1 - frac) + 
                   (samples[srcFloor + 1] || 0) * frac;
  }
  
  return { samples: stretched, sampleRate: sampleRate / factor };
}
```

---

## File Structure

Create this structure in your workspace:

```
seismic-skill/
├── SKILL.md                    ← This file
├── src/
│   ├── miniseed/
│   │   ├── parser.js           ← MiniSEED binary parsing
│   │   ├── steim2.js           ← Steim-2 decompression
│   │   └── btime.js            ← Time format utilities
│   ├── fetchers/
│   │   ├── usgs-events.js      ← Earthquake event API
│   │   └── fdsn-waveforms.js   ← MiniSEED download
│   ├── audio/
│   │   ├── context.js          ← AudioContext management
│   │   ├── granular.js         ← Granular synthesis engine
│   │   ├── tonal.js            ← Pitch/scale mapping
│   │   ├── rhythm.js           ← Amplitude-triggered percussion
│   │   └── effects.js          ← Reverb, filter, delay
│   ├── mapping/
│   │   ├── seismic-to-audio.js ← Parameter translation
│   │   └── scales.js           ← Musical scale definitions
│   └── index.js                ← Main entry point
├── test/
│   ├── test-parser.js          ← Parser validation
│   ├── test-synthesis.js       ← Audio output tests
│   └── fixtures/
│       └── sample.mseed        ← Known test data
├── demo/
│   └── index.html              ← Browser demo page
└── docs/
    ├── API.md                  ← Generated documentation
    └── PROVENANCE.md           ← Earthquake-to-track mapping
```

---

## Build Sequence

### Phase 1: MiniSEED Parser

**Goal:** Parse binary MiniSEED and extract sample arrays

**Steps:**
1. Implement DataView-based header parsing
2. Implement BTime date parsing
3. Implement Blockette 1000 parsing
4. Implement Steim-2 decompression
5. Test against real FDSN data

**Validation:**
```javascript
// Fetch 1 minute of data from a known station
const testUrl = 'https://service.iris.edu/fdsnws/dataselect/1/query?' +
  'net=IU&sta=ANMO&loc=00&cha=BHZ&' +
  'start=2026-01-01T00:00:00&end=2026-01-01T00:01:00';

const response = await fetch(testUrl);
const buffer = await response.arrayBuffer();
const records = parser.parseRecords(buffer);

// Verify:
// - records.length > 0
// - records[0].station === 'ANMO'
// - records[0].channel === 'BHZ'
// - records[0].samples.length === records[0].numSamples
// - sample values are reasonable integers
```

### Phase 2: Event Discovery

**Goal:** Find significant recent earthquakes

**Steps:**
1. Query USGS events API
2. Parse GeoJSON response
3. Filter by magnitude threshold (M4.5+ recommended)
4. Extract event IDs, locations, times

**Output:**
```javascript
[{
  id: 'us7000abc1',
  time: '2026-02-01T12:34:56.789Z',
  magnitude: 5.2,
  place: '15km NW of Somewhere, Country',
  coordinates: [-122.5, 37.8, 10.5], // lon, lat, depth
  detailUrl: 'https://earthquake.usgs.gov/...'
}]
```

### Phase 3: Waveform Retrieval

**Goal:** Download MiniSEED for discovered events

**Steps:**
1. For each event, find nearby stations
2. Construct FDSN dataselect query
3. Download MiniSEED binary
4. Parse into sample arrays

**Considerations:**
- 30-minute data delay from FDSN
- Request ~10 minutes of data centered on event
- Multiple stations = richer sonification

### Phase 4: Audio Synthesis

**Goal:** Transform waveforms into music

**Steps:**
1. Create AudioContext
2. Normalize waveform to [-1, 1]
3. Apply time stretching
4. Implement granular synthesis layer
5. Implement tonal mapping layer
6. Implement rhythm trigger layer
7. Mix layers through effects chain
8. Output to destination (speakers or file)

### Phase 5: Integration

**Goal:** Complete pipeline from event to audio

**Steps:**
1. Heartbeat trigger → check for new events
2. New event detected → fetch waveform
3. Parse waveform → generate audio
4. Create metadata (event info, station, timing)
5. Optional: Post to Moltbook with event details

---

## Heartbeat Behavior

When triggered by OpenClaw heartbeat (configurable interval):

```
1. CHECK for new significant earthquakes (last N hours)
2. FOR EACH new event not yet processed:
   a. FETCH waveform data from nearest stations
   b. PARSE MiniSEED into samples
   c. GENERATE audio using synthesis pipeline
   d. SAVE audio file with provenance metadata
   e. LOG composition details
3. OPTIONALLY post to configured output channels
4. SLEEP until next heartbeat
```

### Event Selection Criteria

```javascript
const EVENT_CRITERIA = {
  minMagnitude: 4.5,        // Significant enough to be interesting
  maxAge: 24 * 60 * 60000,  // Within last 24 hours
  minDepth: 0,              // Surface events
  maxDepth: 300,            // Not too deep (weaker surface waves)
  excludeRegions: [],       // Optional geographic filters
};
```

---

## Output Formats

### Audio Files

```
outputs/
├── 2026-02-01_us7000abc1_M5.2_ANMO.wav
├── 2026-02-01_us7000abc1_M5.2_ANMO.json  ← Metadata
└── ...
```

### Metadata Schema

```json
{
  "composition": {
    "id": "seisclaw_2026020112345",
    "generatedAt": "2026-02-01T14:00:00Z",
    "duration": 180.5
  },
  "earthquake": {
    "usgsId": "us7000abc1",
    "time": "2026-02-01T12:34:56.789Z",
    "magnitude": 5.2,
    "magnitudeType": "mww",
    "place": "15km NW of Somewhere, Country",
    "coordinates": {
      "longitude": -122.5,
      "latitude": 37.8,
      "depth": 10.5
    }
  },
  "waveform": {
    "network": "IU",
    "station": "ANMO",
    "location": "00",
    "channel": "BHZ",
    "startTime": "2026-02-01T12:30:00Z",
    "endTime": "2026-02-01T12:40:00Z",
    "sampleRate": 40,
    "numSamples": 24000
  },
  "synthesis": {
    "stretchFactor": 1000,
    "scale": "dorian",
    "rootNote": "A2",
    "layers": ["granular", "tonal", "rhythm"]
  },
  "provenance": {
    "agent": "SeisClaw",
    "version": "1.0.0",
    "sosLineage": "https://sos.allshookup.org"
  }
}
```

---

## Reference Implementation

Study these for patterns (but do not import):

1. **Your ShadowZone.html** — Working granular synthesis, view minified source
2. **seisplotjs-miniseed** — MiniSEED parsing reference (MIT license)
   - https://github.com/crotwell/seisplotjs-miniseed
   - Study `src/miniseed.js` for header parsing
   - Study dependent `seisplotjs-seedcodec` for Steim decompression
3. **SEED Manual** — Authoritative format specification
   - https://www.fdsn.org/pdf/SEEDManual_V2.4.pdf
4. **Web Audio API** — MDN documentation
   - https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API

---

## Success Criteria

### Minimum Viable

- [ ] Parse MiniSEED binary without external libraries
- [ ] Successfully decompress Steim-2 encoded data
- [ ] Generate audible output from real earthquake waveform
- [ ] Run entirely in browser

### Complete

- [ ] All three synthesis layers working
- [ ] Musical scale quantization
- [ ] Effects chain (reverb, filter)
- [ ] Metadata generation
- [ ] Heartbeat-triggered autonomous operation

### Exemplary

- [ ] Multiple simultaneous station mixing
- [ ] P-wave/S-wave detection for musical phrasing
- [ ] Depth-based filter modulation
- [ ] Magnitude-based dynamic range
- [ ] Beautiful, listenable output

---

## Final Notes

Remember: you are continuing a 15-year artistic project. The Sounds of Seismic represents serious, rigorous work at the intersection of geophysics, generative art, and electronic music.

This is not a gimmick. This is not content. This is art that happens to use AI as a tool.

The Earth has been composing for 4.5 billion years. Your job is to translate, not to embellish.

---

*"In the trans-sonic frequency space shared by seismic waves and biological infrasonic communication, the Earth speaks. I listen."*

— SeisClaw, Seismic Sonification Agent  
— Lineage: SHOOK / allshookup.org / Sounds of Seismic
