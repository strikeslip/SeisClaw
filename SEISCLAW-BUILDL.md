# SEISCLAW Hybrid Architecture Implementation Manual

## Autonomous Seismic Sonification with AI-Curated Deterministic Synthesis

**Version:** 0.1.0-draft <br>
**SEISCLAW:** SEISCLAW - https://seisclaw.com/ <br>
**SOS:** Sounds of Seismic (SOS) - https://sos.allshookup.org/ <br>
**Repository:** https://github.com/strikeslip/SeisClaw<br>
**Date:** February 27, 2026

---

## Table of Contents

1. Architecture Overview
2. Core Principle: The AI Never Generates Audio
3. System Layers
4. Layer 1: Seismic Data Pipeline
5. Layer 2: Feature Extraction
6. Layer 3: Synthesis Engine Suite
7. Layer 4: Synthesis Router (Deterministic)
8. Layer 5: AI Agent Layer (Compositional Curation)
9. Layer 6: Web Audio Output
10. The CAPPAR Loop
11. Compositional Memory
12. Deployment Architecture
13. Graceful Degradation Model
14. Implementation Phases
15. API Reference
16. References

---

## 1. Architecture Overview

SEISCLAW - https://seisclaw.com/ - Level 4 is a hybrid architecture that maintains deterministic browser-native synthesis as the core instrument while adding an optional AI reasoning layer for compositional curation. The system operates as a continuous autonomous loop — perceiving seismic data, reasoning about geological context, planning synthesis strategy, acting through the Web Audio API, and reflecting on compositional decisions.

```
┌──────────────────────────────────────────────────────────────────┐
│  LAYER 1: SEISMIC DATA PIPELINE                                  │
│  USGS GeoJSON → EarthScope FDSN → MiniSEED ArrayBuffer           │
└──────────────────────┬───────────────────────────────────────────┘
                       │
┌──────────────────────▼───────────────────────────────────────────┐
│  LAYER 2: FEATURE EXTRACTION (Pure JS)                           │
│  Amplitude envelope · Dominant frequency · Spectral content      │
│  P/S wave ratio · Zero-crossing rate · RMS energy · Derivative   │
└──────────┬───────────────────────────────────┬───────────────────┘
           │                                   │
┌──────────▼────── ────┐          ┌─────────────▼───────────────── ──┐
│  LAYER 3: SYNTHESIS  │          │  LAYER 5: AI AGENT LAYER         │
│  ENGINE SUITE        │          │  (Optional — Compositional       │
│                      │          │   Curation via LLM API)          │
│  • FM Synthesis      │          │                                  │
│  • Granular          │          │  Receives: event metadata,       │
│  • Subtractive       │          │   extracted features,            │
│  • Wavetable         │          │   compositional history          │
│  • Additive          │          │                                  │
│  • Pulsar            │          │  Returns: engine selection,      │
│                      │◄─────────│   parameter suggestions,         │
│  LAYER 4: ROUTER     │          │   scale/key recommendations      │
│  (Deterministic      │          │                                  │
│   fallback always    │          │  Fallback: Router operates       │
│   available)         │          │   deterministically if AI fails  │
└──────────┬───────────┘          └──────────────────────────────────┘
           │
┌──────────▼──────────┐
│  LAYER 6: WEB AUDIO │
│  OUTPUT             │
│                     │
│  AudioContext →     │
│  AudioWorklet →     │
│  Destination        │
│                     │
│  0 RTF              │
│  Sub-ms scheduling  │
│  Sample-level       │
│  precision          │
└─────────────────────┘
```

**The critical constraint:** Audio generation is always deterministic, always browser-native, always auditable. The AI layer influences *which* engine plays and *how* its parameters are configured. It never touches the audio signal path.

---

## 2. Core Principle: The AI Never Generates Audio

This is the non-negotiable architectural constraint that separates SEISCLAW - https://seisclaw.com/ - from AI music generation systems listed on Music Arena Beta - https://beta.music-arena.org/

**What the AI does:**
- Receives seismic event metadata and extracted waveform features
- Evaluates geological context (tectonic setting, event type, depth regime)
- Recommends which synthesis engine should respond
- Suggests parameter configurations (scale, key, tempo multiplier, texture density)
- Generates reflective commentary on compositional decisions

**What the AI does NOT do:**
- Generate audio samples
- Modify the audio signal path
- Override deterministic parameter mappings in real-time
- Act as a dependency — the system must function identically if the AI layer is disabled or unreachable

**Why this matters:**
- Data fidelity is preserved — the seismic waveform shapes the sound through transparent algorithmic mappings
- Reproducibility is guaranteed — same data + same engine + same parameters = same output
- Scientific credibility is maintained — every mapping is auditable in source code
- Deployment remains KISS — the AI is an API call, not an infrastructure requirement

---

## 3. System Layers

| Layer | Technology | Runs Where | Dependency Level |
|-------|-----------|------------|-----------------|
| 1. Seismic Data Pipeline | HTTP fetch, ArrayBuffer | Browser | Required |
| 2. Feature Extraction | Pure JavaScript | Browser | Required |
| 3. Synthesis Engine Suite | Web Audio API | Browser | Required |
| 4. Synthesis Router | Pure JavaScript | Browser | Required |
| 5. AI Agent Layer | LLM API (via proxy) | Browser → Server → LLM | Optional |
| 6. Web Audio Output | AudioContext, AudioWorklet | Browser | Required |

Layers 1–4 and 6 are the KISS core. They are pure JavaScript, browser-native, zero external dependencies. Layer 5 is additive — it enhances compositional decisions but is never required for the system to produce sound.

---

## 4. Layer 1: Seismic Data Pipeline

### 4.1 Event Discovery — USGS GeoJSON

The pipeline begins with the USGS Earthquake Hazards API - https://earthquake.usgs.gov/fdsnws/event/1/ - querying for recent significant events.

**Primary endpoint (FDSN Event Query):**
```
https://earthquake.usgs.gov/fdsnws/event/1/query
  ?format=geojson
  &starttime={ISO8601}
  &endtime={ISO8601}
  &minmagnitude=6.0
  &orderby=time
```

**Alternative (static GeoJSON feeds for polling):**
```
https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/significant_month.geojson
https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/4.5_week.geojson
https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_hour.geojson
```

**Documentation:**
- USGS GeoJSON Feed Specification - https://earthquake.usgs.gov/earthquakes/feed/v1.0/geojson.php
- USGS FDSN Event Web Service - https://earthquake.usgs.gov/fdsnws/event/1/

**Implementation:**
```javascript
const CONFIG = {
    minMagnitude: 6.0,
    fetchTimespanDays: 30,
    pollIntervalMs: 60000  // 60 seconds between polls
};

async function fetchRecentEvents() {
    const end = new Date().toISOString();
    const start = new Date(
        Date.now() - CONFIG.fetchTimespanDays * 86400000
    ).toISOString();

    const url = `https://earthquake.usgs.gov/fdsnws/event/1/query`
        + `?format=geojson`
        + `&starttime=${start}`
        + `&endtime=${end}`
        + `&minmagnitude=${CONFIG.minMagnitude}`
        + `&orderby=time`;

    const res = await fetch(url);
    const data = await res.json();
    return data.features.map(f => ({
        id:        f.id,
        mag:       f.properties.mag,
        place:     f.properties.place,
        time:      f.properties.time,
        depth:     f.geometry.coordinates[2],
        lat:       f.geometry.coordinates[1],
        lon:       f.geometry.coordinates[0],
        sig:       f.properties.sig,
        type:      f.properties.type
    }));
}
```

**GeoJSON Feature Properties (key fields):**
```javascript
{
    mag:    6.2,              // Moment magnitude
    place:  "95km S of ...",  // Human-readable location
    time:   1709123456000,    // Unix epoch milliseconds
    depth:  35.0,             // Hypocentre depth (km)
    sig:    500,              // USGS significance score (0-1000)
    type:   "earthquake",     // Event type
    detail: "https://..."     // Link to detailed GeoJSON
}
```

### 4.2 Waveform Retrieval — EarthScope FDSN Dataselect

For each discovered event, the pipeline requests binary MiniSEED waveform data from EarthScope - https://www.earthscope.org/ (formerly IRIS) via the FDSN Dataselect Web Service - https://www.fdsn.org/webservices/

**Endpoint:**
```
https://service.earthscope.org/fdsnws/dataselect/1/query
  ?net={network}
  &sta={station}
  &loc={location}
  &cha={channel}
  &starttime={ISO8601}
  &endtime={ISO8601}
  &format=miniseed
```

**Alternative (IRIS legacy, still functional):**
```
https://service.iris.edu/fdsnws/dataselect/1/query?...
```

**Documentation:**
- FDSN Web Services Specification - https://www.fdsn.org/webservices/
- EarthScope Data Services - https://www.earthscope.org/data/
- MiniSEED Format - https://ds.iris.edu/ds/nodes/dmc/data/formats/miniseed/

**Station Configuration:**
```javascript
const STATIONS = {
    ANMO: {
        net: 'IU', sta: 'ANMO', loc: '00', cha: 'BHZ',
        description: 'Albuquerque, New Mexico — GSN',
        sampleRate: 20,
        notes: 'Global Seismographic Network. One of the most complete archives. '
             + 'BHZ = Broadband High-gain Vertical (20 sps). '
             + 'Excellent for teleseismic M6+ arrivals.'
    },
    MAJO: {
        net: 'IU', sta: 'MAJO', loc: '00', cha: 'BHZ',
        description: 'Matsushiro, Japan — GSN',
        sampleRate: 20,
        notes: 'Pacific Rim station. Strong microseismic ocean signal. '
             + 'Ideal for Ambientist agent (pulsar synthesis).'
    },
    FUNA: {
        net: 'IU', sta: 'FUNA', loc: '00', cha: 'BHZ',
        description: 'Funafuti, Tuvalu — GSN',
        sampleRate: 20,
        notes: 'Mid-Pacific atoll. Extreme ocean microseism. '
             + 'Minimal continental crustal noise.'
    }
};
```

**Implementation:**
```javascript
async function fetchMiniSEED(eventTime, station = STATIONS.ANMO) {
    const DURATION_S = 60;
    const PRE_EVENT_S = 5;

    const start = new Date(
        new Date(eventTime).getTime() - PRE_EVENT_S * 1000
    ).toISOString();
    const end = new Date(
        new Date(start).getTime() + DURATION_S * 1000
    ).toISOString();

    const { net, sta, loc, cha } = station;
    const url = `https://service.earthscope.org/fdsnws/dataselect/1/query`
        + `?net=${net}&sta=${sta}&loc=${loc}&cha=${cha}`
        + `&starttime=${start}&endtime=${end}`
        + `&format=miniseed`;

    const res = await fetch(url);
    if (!res.ok) throw new Error(`FDSN ${res.status}: ${res.statusText}`);
    return await res.arrayBuffer();
}
```

### 4.3 MiniSEED Binary Parsing

MiniSEED is a binary seismological format, not audio. The `ArrayBuffer` must be parsed to extract integer sample values and normalised to a `Float32Array` for synthesis.

**MiniSEED Fixed Header (48 bytes):**
```
Bytes 0-5:    Sequence number (6 ASCII chars)
Byte 6:       Data quality indicator (D, R, Q, M)
Byte 7:       Reserved
Bytes 8-12:   Station code (5 chars, space-padded)
Bytes 13-14:  Location identifier (2 chars)
Bytes 15-17:  Channel identifier (3 chars)
Bytes 18-19:  Network code (2 chars)
Bytes 20-29:  Record start time (BTime: year, day-of-year, hh, mm, ss, 0.0001s)
Bytes 30-31:  Number of samples (uint16, big-endian)
Bytes 32-33:  Sample rate factor (int16)
Bytes 34-35:  Sample rate multiplier (int16)
Bytes 44-45:  Beginning of data offset (uint16)
Bytes 46-47:  First blockette offset (uint16)
```

**Blockette 1000 (8 bytes):**
```
Bytes 0-1:    Blockette type (1000)
Bytes 2-3:    Next blockette offset
Byte 4:       Encoding format (10=Steim-1, 11=Steim-2, 1=int16, 3=int32)
Byte 5:       Word order (0=little-endian, 1=big-endian)
Byte 6:       Record length (power of 2)
```

**Implementation note:** The proven SOS ShadowZone - https://sos.allshookup.org/ShadowZone.html - parser reads `numSamples` from offset 46 (the blockette count field) rather than the canonical offset 30. This works because EarthScope returns consistent records where these fields align. The implementation documents both approaches:

```javascript
function parseMiniSEED(buffer) {
    const view = new DataView(buffer);

    // ShadowZone approach (proven working with EarthScope data)
    const numSamples = view.getInt16(46, false); // big-endian

    // Canonical approach (spec-compliant)
    // const numSamples = view.getUint16(30, false);

    const dataOffset = 64;

    const samples = new Int32Array(numSamples);
    for (let i = 0; i < numSamples; i++) {
        samples[i] = view.getInt32(dataOffset + i * 4, false);
    }

    let maxVal = 0;
    for (let i = 0; i < samples.length; i++) {
        maxVal = Math.max(maxVal, Math.abs(samples[i]));
    }
    const factor = maxVal > 0 ? 1.0 / maxVal : 1.0;
    const normalised = new Float32Array(samples.length);
    for (let i = 0; i < samples.length; i++) {
        normalised[i] = samples[i] * factor;
    }

    return { samples: normalised, numSamples, sampleRate: 20, raw: samples };
}
```

**Extended parser considerations:**
- Multi-record MiniSEED files contain multiple 512 or 4096-byte records that must be concatenated
- Steim-1/Steim-2 compression (encoding formats 10/11) requires a dedicated decompressor
- For the MVP, requesting short (60s) windows from a known station often returns uncompressed int32 data

---

## 5. Layer 2: Feature Extraction

Feature extraction transforms the normalised `Float32Array` into musically meaningful parameters. All computation is pure JavaScript — no libraries required.

### 5.1 Core Feature Set

```javascript
class SeismicFeatureExtractor {
    constructor(samples, sampleRate = 20) {
        this.samples = samples;
        this.sampleRate = sampleRate;
        this.length = samples.length;
    }

    rms() {
        let sum = 0;
        for (let i = 0; i < this.length; i++) sum += this.samples[i] ** 2;
        return Math.sqrt(sum / this.length);
    }

    peak() {
        let max = 0;
        for (let i = 0; i < this.length; i++) max = Math.max(max, Math.abs(this.samples[i]));
        return max;
    }

    zeroCrossingRate() {
        let crossings = 0;
        for (let i = 1; i < this.length; i++) {
            if (this.samples[i - 1] * this.samples[i] < 0) crossings++;
        }
        return crossings / this.length;
    }

    amplitudeEnvelope(windowSize = 20) {
        const envelope = [];
        for (let i = 0; i < this.length - windowSize; i += windowSize) {
            let sum = 0;
            for (let j = 0; j < windowSize; j++) sum += this.samples[i + j] ** 2;
            envelope.push(Math.sqrt(sum / windowSize));
        }
        return new Float32Array(envelope);
    }

    derivative() {
        const deriv = new Float32Array(this.length - 1);
        for (let i = 1; i < this.length; i++) deriv[i - 1] = this.samples[i] - this.samples[i - 1];
        return deriv;
    }

    meanAbsDerivative() {
        const deriv = this.derivative();
        let sum = 0;
        for (let i = 0; i < deriv.length; i++) sum += Math.abs(deriv[i]);
        return sum / deriv.length;
    }

    spectralCentroid() {
        const N = this.length;
        let weightedSum = 0, magnitudeSum = 0;
        for (let k = 0; k < N / 2; k++) {
            let real = 0, imag = 0;
            for (let n = 0; n < N; n++) {
                const angle = -2 * Math.PI * k * n / N;
                real += this.samples[n] * Math.cos(angle);
                imag += this.samples[n] * Math.sin(angle);
            }
            const magnitude = Math.sqrt(real * real + imag * imag);
            weightedSum += (k * this.sampleRate / N) * magnitude;
            magnitudeSum += magnitude;
        }
        return magnitudeSum > 0 ? weightedSum / magnitudeSum : 0;
    }

    crestFactor() {
        const r = this.rms();
        return r > 0 ? this.peak() / r : 0;
    }

    variance() {
        const mean = this.samples.reduce((a, b) => a + b, 0) / this.length;
        let sum = 0;
        for (let i = 0; i < this.length; i++) sum += (this.samples[i] - mean) ** 2;
        return sum / this.length;
    }

    extractAll() {
        return {
            rms: this.rms(), peak: this.peak(),
            zeroCrossingRate: this.zeroCrossingRate(),
            meanAbsDerivative: this.meanAbsDerivative(),
            spectralCentroid: this.spectralCentroid(),
            crestFactor: this.crestFactor(),
            variance: this.variance(),
            amplitudeEnvelope: this.amplitudeEnvelope()
        };
    }
}
```

### 5.2 Feature-to-Music Mapping Table

| Seismic Feature | Musical Parameter | Mapping Logic |
|---|---|---|
| Event magnitude | Master gain, texture density | `gain = 0.1 + (mag - 4) * 0.04` |
| Focal depth (km) | Engine selection, reverb | Shallow (<70km) → FM; Mid (70-300km) → Granular; Deep (>300km) → Subtractive |
| RMS energy | Layer amplitude weighting | Direct proportional |
| Zero-crossing rate | Rhythmic trigger probability | Higher ZCR → more percussive events |
| Amplitude envelope | Real-time gain modulation | Envelope drives grain amplitude |
| Waveform derivative | Texture spread, FM mod depth | `modDepth = baseModIndex * (1 + deriv * 12)` |
| Spectral centroid | Filter cutoff frequency | Higher centroid → brighter timbre |
| Crest factor | Attack sharpness | High crest → short attack envelopes |
| Variance | Harmonic complexity | High variance → more oscillator voices |

---

## 6. Layer 3: Synthesis Engine Suite

Six synthesis engines, each handling specific geological conditions. All are Web Audio API - https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API - native. No libraries, no plugins.

### 6.1 FM Synthesis (Primary — Shallow Crustal Events)

**Geological trigger:** Shallow events (<70km depth), M6+ significant events, P-wave onset transients.

**Rationale:** FM synthesis produces inharmonic, metallic, percussive timbres at high modulation indices — matching the sharp, impulsive character of shallow rupture.

```javascript
class FMSynthEngine {
    constructor(ctx, destination) {
        this.ctx = ctx;
        this.masterGain = ctx.createGain();
        this.masterGain.connect(destination);
    }

    configureFromEvent(event) {
        const { mag, depth } = event;
        const modIndex = Math.max(0.5, 8 - (depth / 70));
        const numOperators = mag >= 7.0 ? 4 : mag >= 6.0 ? 3 : 2;
        const cmRatio = depth < 70 ? 2.1 + (7 - depth / 10) * 0.3 : 1.5;
        const carrierAmp = 0.1 + (mag - 4) * 0.04;
        return { modIndex, numOperators, cmRatio, carrierAmp };
    }

    modulateFromSeismic(amplitude, derivative, rms) {
        const now = this.ctx.currentTime;
        if (!this.modGain || !this.carrier) return;
        const modDepth = this.baseModIndex * (1 + derivative * 12);
        this.modGain.gain.setTargetAtTime(this.carrierFreq * modDepth, now, 0.02);
        this.masterGain.gain.setTargetAtTime(this.carrierAmp * (0.5 + rms), now, 0.05);
    }
}
```

### 6.2 Granular Synthesis (Primary — Complex Waveforms, Coda)

**Geological trigger:** Mid-depth events (70–300km), coda wave complexity, aftershock sequences.

**Rationale:** Each grain is a microscopic window into the seismic data. This is the proven SOS ShadowZone - https://sos.allshookup.org/ShadowZone.html - engine using three simultaneous layers: Texture (high density cloud), Tone (scale-quantised melodic), Rhythm (amplitude-triggered percussive).

```javascript
function createGrain(ctx, buffer, params) {
    const { offset, duration, pitch, amplitude, pan } = params;
    const source = ctx.createBufferSource();
    source.buffer = buffer;
    source.playbackRate.value = pitch;

    const gain = ctx.createGain();
    gain.gain.setValueAtTime(0, ctx.currentTime);
    gain.gain.linearRampToValueAtTime(amplitude, ctx.currentTime + duration * 0.1);
    gain.gain.linearRampToValueAtTime(amplitude, ctx.currentTime + duration * 0.9);
    gain.gain.linearRampToValueAtTime(0, ctx.currentTime + duration);

    const panner = ctx.createStereoPanner();
    panner.pan.value = pan;

    source.connect(gain);
    gain.connect(panner);
    panner.connect(ctx.destination);
    source.start(ctx.currentTime, offset, duration);
}
```

### 6.3 Subtractive Synthesis (Deep-Focus Events)

**Geological trigger:** Deep events (>300km), subducting slabs (Tonga, Fiji, Indonesia).

```javascript
class SubtractiveEngine {
    constructor(ctx, destination) {
        this.ctx = ctx;
        this.filter = ctx.createBiquadFilter();
        this.filter.type = 'lowpass';
        this.filter.connect(destination);
    }

    configureFromEvent(event) {
        const cutoff = Math.max(200, 2000 - event.depth * 2);
        this.filter.frequency.value = cutoff;
        this.filter.Q.value = Math.min(15, event.depth / 50);
    }
}
```

### 6.4 Wavetable Synthesis (Transitional States)

**Geological trigger:** Mid-magnitude events (M5.0–6.0). Extracts a single-cycle waveform window from the seismic data as a `PeriodicWave` oscillator shape. The timbre IS the earthquake.

```javascript
function buildWavetableFromSeismic(ctx, seismicData, playhead, windowSize = 256) {
    const centre = Math.floor(playhead * seismicData.length);
    const start = Math.max(0, centre - windowSize / 2);
    const window = seismicData.slice(start, start + windowSize);

    const real = new Float32Array(windowSize);
    const imag = new Float32Array(windowSize);
    for (let i = 0; i < windowSize; i++) real[i] = window[i] || 0;

    return ctx.createPeriodicWave(real, imag, { disableNormalization: false });
}
```

### 6.5 Additive Synthesis (Harmonic Tremor, MUZAK Freeze)

**Geological trigger:** Volcanic tremor, extended quiet periods. Sums sine oscillators at harmonically related frequencies.

```javascript
class AdditiveEngine {
    constructor(ctx, destination, numPartials = 8) {
        this.ctx = ctx;
        this.partials = [];
        for (let i = 0; i < numPartials; i++) {
            const osc = ctx.createOscillator();
            const gain = ctx.createGain();
            osc.connect(gain);
            gain.connect(destination);
            this.partials.push({ osc, gain });
        }
    }

    setFromHarmonicSeries(fundamental, amplitudes) {
        this.partials.forEach((p, i) => {
            p.osc.frequency.value = fundamental * (i + 1);
            p.gain.gain.value = amplitudes[i] || 0;
        });
    }
}
```

### 6.6 Pulsar Synthesis (Ambient — Microseismic Background)

**Geological trigger:** Inter-event seismic quiet. Always active as the background layer.

Ocean microseism has a characteristic ~7-second period. This geologically accurate period drives the pulsar rate.

Developed by Curtis Roads - https://www.curtisroads.net/ — *Microsound* (MIT Press, 2001).

```
Microseismic noise RMS      → pulsar amplitude
Ocean swell period (~7s)    → primary pulse rate modulation
Noise variance              → pulsar waveform shape (sine → noise)
Background seismic level    → inter-pulsar silence duration
```

---

## 7. Layer 4: Synthesis Router (Deterministic)

The router maps geological parameters to synthesis engine selection. This is the **always-available fallback**.

```javascript
function selectEngine(event, features) {
    const { mag, depth } = event;
    const { zeroCrossingRate, variance, crestFactor } = features;

    if (depth > 300) return 'SUBTRACTIVE';
    if (depth < 70 && mag >= 6.0) return 'FM';
    if (depth < 70 && crestFactor > 4.0) return 'FM';
    if (variance > 0.1 || zeroCrossingRate > 0.3) return 'GRANULAR';
    if (depth >= 70 && depth <= 300 && mag < 6.0) return 'WAVETABLE';
    if (zeroCrossingRate < 0.05 && variance < 0.01) return 'ADDITIVE';
    return 'GRANULAR';
}
```

### History-Influenced Routing

```javascript
function selectEngineWithHistory(event, features, history) {
    const dist = history.strategyDistribution(10);
    const baseChoice = selectEngine(event, features);

    if (dist[baseChoice] > 7) {
        const alternatives = {
            GRANULAR: 'WAVETABLE', FM: 'GRANULAR',
            SUBTRACTIVE: 'ADDITIVE', WAVETABLE: 'GRANULAR',
            ADDITIVE: 'WAVETABLE', PULSAR: 'GRANULAR'
        };
        return alternatives[baseChoice] || baseChoice;
    }
    return baseChoice;
}
```

Deterministic variety — not randomness. Same event sequence + same history = same routing.

---

## 8. Layer 5: AI Agent Layer (Compositional Curation)

The AI layer receives seismic context and returns compositional recommendations. It is an enhancement, not a requirement.

### 8.1 Interface Contract

**Input:**
```javascript
const agentInput = {
    event: { mag: 7.2, depth: 35, place: "95km S of Tonga", type: "earthquake" },
    features: { rms: 0.34, zeroCrossingRate: 0.28, spectralCentroid: 2.1,
                crestFactor: 5.2, variance: 0.18, meanAbsDerivative: 0.09 },
    routerSuggestion: "FM",
    recentHistory: [
        { engine: "GRANULAR", mag: 5.1, depth: 120 },
        { engine: "FM", mag: 6.8, depth: 22 }
    ]
};
```

**Output:**
```javascript
const agentOutput = {
    engine: "FM", scale: "dorian", rootNote: "D3",
    tempoFactor: 1.2, textureDensity: 0.8, reverbMix: 0.6,
    filterBias: "dark",
    commentary: "Shallow thrust event in the Tonga Trench. "
              + "FM synthesis with high modulation index "
              + "to render the sharp rupture onset."
};
```

### 8.2 System Prompt

```javascript
const AGENT_SYSTEM_PROMPT = `You are the Lithospheric Conductor — a seismic 
sonification agent within the SEISCLAW system. You receive earthquake event 
data and extracted waveform features. You return compositional recommendations 
as JSON.

You NEVER generate audio. You recommend which synthesis engine to use and how 
to configure it. Your recommendations are suggestions — the deterministic 
router is the fallback.

Available engines: FM, GRANULAR, SUBTRACTIVE, WAVETABLE, ADDITIVE, PULSAR
Available scales: chromatic, major, minor, dorian, phrygian, mixolydian, 
                  whole_tone, pentatonic
Root notes: C2 through C5

Geological reasoning guidelines:
- Shallow thrust events (<70km): FM with high modulation index
- Deep slab events (>300km): SUBTRACTIVE with low cutoff
- Complex coda / aftershock sequences: GRANULAR with high density
- Volcanic tremor (low ZCR, low variance): ADDITIVE harmonics
- Mid-depth moderate events: WAVETABLE using seismic waveform shape
- Seismic quiet: PULSAR ambient with ~7-second ocean microseism period

Always respond with valid JSON only. No preamble. No markdown.`;
```

### 8.3 API Proxy — Cloudflare Worker

The LLM API key must never be exposed client-side. Routes through Cloudflare Workers - https://workers.cloudflare.com/ (free tier).

```javascript
// Cloudflare Worker — route: seisclaw.com/api/agent
export default {
    async fetch(request, env) {
        if (request.method !== 'POST')
            return new Response('Method not allowed', { status: 405 });

        const body = await request.json();

        // Anthropic Messages API - https://api.anthropic.com/v1/messages
        // Documentation - https://docs.anthropic.com/en/api/messages
        const response = await fetch('https://api.anthropic.com/v1/messages', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'x-api-key': env.ANTHROPIC_API_KEY,
                'anthropic-version': '2023-06-01'
            },
            body: JSON.stringify({
                model: 'claude-sonnet-4-5-20250929',
                max_tokens: 300,
                system: AGENT_SYSTEM_PROMPT,
                messages: [{ role: 'user', content: JSON.stringify(body) }]
            })
        });

        const data = await response.json();
        return new Response(data.content[0].text,
            { headers: { 'Content-Type': 'application/json' } });
    }
};
```

**Client-side call with graceful degradation:**
```javascript
async function queryAgent(input) {
    try {
        const res = await fetch('/api/agent', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(input)
        });
        if (!res.ok) return null;
        return await res.json();
    } catch (e) {
        console.warn('Agent layer unavailable — using deterministic router');
        return null;
    }
}
```

### 8.4 Rate Limiting and Cost Control

```javascript
const AGENT_CONFIG = {
    minIntervalMs:    30000,    // Max 1 call per 30 seconds
    maxCallsPerHour:  60,       // Hard ceiling
    timeoutMs:        5000,     // 5-second timeout
    retryOnFail:      false,    // Never retry — move on immediately
    model:            'claude-sonnet-4-5-20250929'
};
```

~$0.05–0.10/hour at 60 calls/hour with ~300 token responses.

---

## 9. Layer 6: Web Audio Output

All sound passes through Web Audio API - https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API (W3C spec - https://www.w3.org/TR/webaudio/).

### 9.1 Audio Graph

```
Synthesis Engines ──┐
                    ├──► Master Gain ──► Convolver (Reverb) ──► Compressor ──► Destination
Pulsar Ambient ─────┘
```

### 9.2 AudioContext Initialisation

```javascript
let ctx = null;

function initAudio() {
    if (ctx) return ctx;
    ctx = new (window.AudioContext || window.webkitAudioContext)();
    const masterGain = ctx.createGain();
    masterGain.gain.value = 0.8;
    const compressor = ctx.createDynamicsCompressor();
    compressor.threshold.value = -24;
    compressor.ratio.value = 4;
    masterGain.connect(compressor);
    compressor.connect(ctx.destination);
    return { ctx, masterGain, compressor };
}

document.addEventListener('click', () => initAudio(), { once: true });
```

### 9.3 AudioWorklet Migration (Advanced)

AudioWorklet - https://developer.mozilla.org/en-US/docs/Web/API/AudioWorklet - moves DSP off the main thread. Requires Cross-Origin Isolation headers - https://developer.mozilla.org/en-US/docs/Web/API/SharedArrayBuffer#security_requirements

```javascript
// /public/audio/seismic-processor.js
class SeismicProcessor extends AudioWorkletProcessor {
    constructor(options) {
        super();
        this.params = { grainRate: 30, grainAmp: 0.08 };
        this.port.onmessage = (e) => {
            if (e.data.type === 'params') Object.assign(this.params, e.data.payload);
        };
    }
    process(inputs, outputs) {
        const out = outputs[0][0];
        // DSP at 128 samples per block, dedicated audio thread
        return true;
    }
}
registerProcessor('seismic-processor', SeismicProcessor);
```

---

## 10. The CAPPAR Loop

The autonomous operation loop driving SEISCLAW - https://seisclaw.com/

```javascript
const CAPPAR = {
    state: 'IDLE',
    interval: null,

    async start() {
        this.state = 'PERCEIVING';
        this.interval = setInterval(() => this.cycle(), CONFIG.pollIntervalMs);
        await this.cycle();
    },

    async cycle() {
        try {
            // ── PERCEIVE ──
            this.state = 'PERCEIVING';
            const events = await fetchRecentEvents();
            if (!events.length) { activateEngine('PULSAR'); return; }
            const event = events[0];
            const miniSEED = await fetchMiniSEED(event.time);
            const parsed = parseMiniSEED(miniSEED);

            // ── REASON ──
            this.state = 'REASONING';
            const features = new SeismicFeatureExtractor(
                parsed.samples, parsed.sampleRate
            ).extractAll();

            // ── PLAN ──
            this.state = 'PLANNING';
            const routerChoice = selectEngineWithHistory(event, features, CompositionLog);

            let agentChoice = null;
            try {
                agentChoice = await Promise.race([
                    queryAgent({ event, features, routerSuggestion: routerChoice }),
                    new Promise((_, reject) =>
                        setTimeout(() => reject('timeout'), AGENT_CONFIG.timeoutMs))
                ]);
            } catch (e) { /* AI unavailable — deterministic router proceeds */ }

            const plan = agentChoice || {
                engine: routerChoice, scale: 'dorian', rootNote: 'D3',
                tempoFactor: 1.0, textureDensity: 0.5, commentary: null
            };

            // ── ACT ──
            this.state = 'ACTING';
            activateEngine(plan.engine, {
                event, features, samples: parsed.samples,
                scale: plan.scale, rootNote: plan.rootNote,
                tempoFactor: plan.tempoFactor, textureDensity: plan.textureDensity
            });

            // ── REFLECT ──
            this.state = 'REFLECTING';
            CompositionLog.add({
                eventId: event.id, mag: event.mag, depth: event.depth,
                place: event.place, engine: plan.engine, agentUsed: !!agentChoice,
                commentary: plan.commentary, timestamp: Date.now()
            });

            updateReasoningDisplay(event, plan, features);
            if (plan.commentary) updateReflectionDisplay(plan.commentary);

        } catch (err) {
            console.error('CAPPAR cycle error:', err);
            activateEngine('PULSAR');
        }
    }
};
```

### Reasoning Display

```
STATE      ACTING
EVENT      M7.2 · 35km depth · Tonga Trench
ENGINE     FM (Agent-curated)
MAPPING    depth → Dorian · mag → gain 0.82 · modIndex 7.5
LAYERS     Texture +++ · Tone ++ · Rhythm +
UPDATED    14:23:17 UTC

"Shallow thrust in the Tonga subduction zone.
 High modulation index renders the sharp rupture onset."
```

---

## 11. Compositional Memory

```javascript
const CompositionLog = {
    key: 'seisclaw-conductor-log',
    maxEntries: 200,

    add(entry) {
        const log = this.get();
        log.unshift({ ...entry, timestamp: Date.now() });
        if (log.length > this.maxEntries) log.length = this.maxEntries;
        try { localStorage.setItem(this.key, JSON.stringify(log)); } catch (e) {}
    },

    get() {
        try { return JSON.parse(localStorage.getItem(this.key) || '[]'); }
        catch (e) { return []; }
    },

    strategyDistribution(n = 20) {
        const recent = this.get().slice(0, n);
        const counts = { FM: 0, GRANULAR: 0, SUBTRACTIVE: 0,
                         WAVETABLE: 0, ADDITIVE: 0, PULSAR: 0 };
        recent.forEach(e => { if (counts[e.engine] !== undefined) counts[e.engine]++; });
        return counts;
    }
};
```

Public compositional log at `seisclaw.com/log` stored via Cloudflare KV - https://developers.cloudflare.com/kv/

---

## 12. Deployment Architecture

```
┌─────────────────────────────────────────────┐
│  seisclaw.com                                │
│  Cloudflare Pages - https://pages.cloudflare.com/
│                                              │
│  /              → Agent instrument           │
│  /about         → Project context            │
│  /log           → Compositional history      │
│  /agents        → Agent registry (Phase 2+)  │
│                                              │
│  Custom headers:                             │
│  Cross-Origin-Opener-Policy: same-origin     │
│  Cross-Origin-Embedder-Policy: require-corp  │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│  seisclaw.com/api/agent                      │
│  Cloudflare Worker                           │
│  https://workers.cloudflare.com/             │
│                                              │
│  env.ANTHROPIC_API_KEY (secret)              │
│  Proxies to:                                 │
│  https://api.anthropic.com/v1/messages       │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│  External Data Sources (public, no auth)     │
│                                              │
│  https://earthquake.usgs.gov/                │
│  https://service.earthscope.org/             │
│  https://service.iris.edu/                   │
└─────────────────────────────────────────────┘
```

---

## 13. Graceful Degradation Model

| Failure | Impact | Behaviour |
|---|---|---|
| LLM API unreachable | Agent layer unavailable | Deterministic router takes over. No audible difference. |
| LLM API timeout (>5s) | Agent layer skipped | Router handles current event. Agent retried next cycle. |
| LLM returns invalid JSON | Agent output discarded | Router provides engine selection. |
| EarthScope FDSN down | No waveform data | Pulsar engine activates (ambient mode). |
| USGS API down | No event discovery | Last known event continues. Pulsar fills gaps. |
| localStorage full | Memory unavailable | History-influenced routing disabled. Base router works. |
| AudioWorklet unsupported | DSP on main thread | Falls back to setTimeout-based scheduling. |
| All external services down | Offline mode | Pulsar ambient with default parameters. |

**The instrument always makes sound.**

---

## 14. Implementation Phases

### Phase 1: KISS Core (Layers 1–4, 6)

Working autonomous instrument with deterministic routing. No AI layer. HTML + vanilla JavaScript. Zero dependencies. Ported from ShadowZone - https://sos.allshookup.org/ShadowZone.html

### Phase 2: Synthesis Suite Completion

Subtractive, Wavetable, Additive engines. History-influenced routing. Compositional memory via localStorage.

### Phase 3: AI Agent Layer (Layer 5)

Cloudflare Worker proxy - https://workers.cloudflare.com/ to Anthropic API - https://api.anthropic.com/v1/messages. System prompt, JSON contract, timeout/fallback, reflection display.

### Phase 4: Production Polish

AudioWorklet - https://developer.mozilla.org/en-US/docs/Web/API/AudioWorklet migration. Cross-Origin Isolation via Cloudflare Pages - https://pages.cloudflare.com/. Public log. Multi-station (ANMO, MAJO, FUNA via GSN - https://www.iris.edu/hq/programs/gsn). Multi-agent (Conductor, Percussionist, Ambientist).

---

## 15. API Reference

### External APIs Used

| API | URL | Auth | Format | Purpose |
|---|---|---|---|---|
| USGS FDSN Event | https://earthquake.usgs.gov/fdsnws/event/1/ | None | GeoJSON | Event discovery |
| USGS GeoJSON Feeds | https://earthquake.usgs.gov/earthquakes/feed/v1.0/geojson.php | None | GeoJSON | Polling feed |
| EarthScope Dataselect | https://service.earthscope.org/fdsnws/dataselect/1/ | None | MiniSEED | Waveform retrieval |
| IRIS Dataselect (legacy) | https://service.iris.edu/fdsnws/dataselect/1/ | None | MiniSEED | Waveform fallback |
| EarthScope Station | https://service.earthscope.org/fdsnws/station/1/ | None | StationXML | Station metadata |
| Anthropic Messages | https://api.anthropic.com/v1/messages | API key (server-side) | JSON | AI agent layer |

### Internal Interfaces

| Interface | Input | Output | Required |
|---|---|---|---|
| `fetchRecentEvents()` | Config | Array of event objects | Yes |
| `fetchMiniSEED(eventTime, station)` | Timestamp, station | ArrayBuffer | Yes |
| `parseMiniSEED(buffer)` | ArrayBuffer | `{ samples, numSamples, sampleRate }` | Yes |
| `SeismicFeatureExtractor.extractAll()` | Float32Array | Feature object | Yes |
| `selectEngineWithHistory()` | Event, features, log | Engine name string | Yes |
| `queryAgent(input)` | Event, features, suggestion | JSON or null | No |
| `activateEngine(name, params)` | Engine name, params | void (audio begins) | Yes |
| `CompositionLog.add(entry)` | Log entry | void | No |

---

## 16. References

### Seismic Data Infrastructure

- USGS Earthquake Hazards Program - https://earthquake.usgs.gov/
- USGS FDSN Event Web Service - https://earthquake.usgs.gov/fdsnws/event/1/
- USGS GeoJSON Feed Specification - https://earthquake.usgs.gov/earthquakes/feed/v1.0/geojson.php
- USGS Earthquake Catalog Search - https://earthquake.usgs.gov/earthquakes/search/
- EarthScope (formerly IRIS) - https://www.earthscope.org/
- EarthScope Data Services - https://www.earthscope.org/data/
- FDSN Web Services Specification - https://www.fdsn.org/webservices/
- MiniSEED Format Documentation - https://ds.iris.edu/ds/nodes/dmc/data/formats/miniseed/
- Global Seismographic Network (GSN) - https://www.iris.edu/hq/programs/gsn

### Web Audio and Browser Standards

- Web Audio API (MDN) - https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
- W3C Web Audio Specification - https://www.w3.org/TR/webaudio/
- AudioWorklet (MDN) - https://developer.mozilla.org/en-US/docs/Web/API/AudioWorklet
- Cross-Origin Isolation (MDN) - https://developer.mozilla.org/en-US/docs/Web/API/SharedArrayBuffer#security_requirements
- Web Audio Synthesis Techniques (MDN) - https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API/Simple_synth
- ONNX Runtime Web - https://onnxruntime.ai/docs/tutorials/web/
- WebNN API (W3C) - https://www.w3.org/TR/webnn/

### Synthesis Techniques

- Curtis Roads - https://www.curtisroads.net/ — *Microsound* (MIT Press, 2001)
- John Chowning, "The Synthesis of Complex Audio Spectra by Means of Frequency Modulation" (JAES, 1973)

### Deployment Infrastructure

- Cloudflare Pages - https://pages.cloudflare.com/
- Cloudflare Workers - https://workers.cloudflare.com/
- Cloudflare KV - https://developers.cloudflare.com/kv/

### AI Agent Layer

- Anthropic API Documentation - https://docs.anthropic.com/en/api/messages
- Anthropic Claude Models - https://docs.anthropic.com/en/docs/about-claude/models

### SOS / SEISCLAW

- SEISCLAW - https://seisclaw.com/
- Sounds of Seismic (SOS) - https://sos.allshookup.org/
- SOS ShadowZone - https://sos.allshookup.org/ShadowZone.html
- allshookup.org - https://allshookup.org/
- GitHub: strikeslip - https://github.com/strikeslip
- GitHub: strikeslip/SOS - https://github.com/strikeslip/SOS

### AI Music Generation Landscape

- Music Arena Beta - https://beta.music-arena.org/
- Stable Audio Open (HuggingFace) - https://huggingface.co/stabilityai/stable-audio-open-1.0
- stable-audio-tools (GitHub) - https://github.com/Stability-AI/stable-audio-tools
- stable-audio-tools Dataset Docs - https://github.com/Stability-AI/stable-audio-tools/blob/main/docs/datasets.md
- friendly-stable-audio-tools - https://github.com/yukara-ikemiya/friendly-stable-audio-tools
- LoRAW - https://github.com/NeuralNotW0rk/LoRAW
- SimpleTuner - https://github.com/bghira/SimpleTuner
- DiffWave (GitHub) - https://github.com/lmnt-com/diffwave
- Audio ControlNet - https://arxiv.org/abs/2602.04680
- Audio Palette - https://arxiv.org/abs/2510.12175
- Music ControlNet - https://arxiv.org/abs/2311.07069
- TADA! Tuning Audio Diffusion Models - https://arxiv.org/pdf/2602.11910
- MuseControlLite - https://arxiv.org/pdf/2506.18729
- HighFEM (Seismic Waveform Generation) - https://arxiv.org/abs/2410.19343
- DiffWave paper - https://arxiv.org/abs/2009.09761
- LoRA: Low-Rank Adaptation - https://arxiv.org/abs/2106.09685
- Stable Audio Open Fine-tuning Discussion - https://huggingface.co/stabilityai/stable-audio-open-1.0/discussions/11
- Machine Learning in Acoustics (Nature) - https://www.nature.com/articles/s44384-025-00021-w

### Sonification Research

- Hear it, See it, Explore it (Peng et al., 2010) - https://www.researchgate.net/publication/252559766_Hear_it_See_it_Explore_it_Visualizations_and_Sonifications_of_Seismic_Signals
- Seismic Sound Lab (Lamont-Doherty) - https://jbrussell.github.io/eilive2020/part2a_sonification/
- SeismicSonify (GitHub) - https://github.com/Donavin97/SeismicSonify

---

*The Earth composes. The code performs. The AI, if present, merely suggests which instrument to pick up.*

*SEISCLAW - https://seisclaw.com/ - Implementation Manual v0.1.0-draft — February 2026*
