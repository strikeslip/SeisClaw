# 🦀 SeisClaw
### Autonomous Electronica Music Agents 
- https://seisclaw.com/
- https://sos.allshookup.org/

---

## What Is SeisClaw 

SeisClaw is a system of autonomous agents that listen to Earth's seismic activity and compose electronica music without human instruction. Each agent executes a continuous **SeisConduct** loop — Perceive, Reason, Plan, Act, Reflect — connecting to live global seismic sensor networks, parsing real binary waveform data, and executing synthesis decisions in a browser via the Web Audio API.

The earth composes. The agents perform.

SeisClaw is the agentic evolution of Sounds of Seismic (SOS), a concept in development since 2012. SOS instruments — ShadowZone, ONE, SEISFLOW, SEISTRONICA, ANMO FM Synth — require a human to initiate each session. SeisClaw agents begin composing after a single activation gesture and continue indefinitely, responding to planetary seismic activity across geological time.

ShadowZone (sos.allshookup.org/ShadowZone.html) is the base instrument. Its three-layer granular synthesis engine, MiniSEED data pipeline, and seismic parameter mappings are the proven foundation on which SeisClaw's agentic architecture is to be built.

---

## Architecture: The SeisConduct Loop (SCLoop)

Every SeisClaw agent executes five stages continuously and autonomously:

**Perceive** — Connect to USGS earthquake feeds and EarthScope/IRIS sensor networks. Fetch GeoJSON event data. Retrieve binary MiniSEED waveform records via FDSN web services. Parse raw waveform samples. Classify event signatures: tectonic rupture onset, deep-focus rumble, ocean microseism pulse, inter-event background noise.

**Reason** — Evaluate the compositional potential of each event. Magnitude maps to intensity and gain. Focal depth maps to synthesis mode selection and reverb character. Waveform derivative maps to texture spread and density. RMS energy maps to tone layer weight. Zero-crossing density maps to rhythmic trigger probability.

**Plan** — Select synthesis strategy from the six available techniques based on reasoning output. Granular and FM synthesis are the primary engines. Subtractive, Additive, Wavetable, and Pulsar synthesis handle specific event types and transitional states.

**Act** — Execute the selected strategy via Web Audio API. Update all active synthesis parameters. Render audio. Broadcast through the browser interface — no plugins, no specialised software, globally accessible.

**Reflect** — Evaluate the completed composition cycle against accumulated compositional history. Generate a brief commentary noting the geological conditions and synthesis decisions. Store the event record in the compositional log. Carry adjusted parameters forward to influence the next cycle.

The SCLoop is the performer. Making it observable is the design challenge.

---

## The Synthesis Suite

SeisClaw deploys six synthesis techniques. Granular and FM are the primary compositional engines — they handle the majority of seismic events because their parameter spaces map most directly to the properties of seismic waveforms. The remaining four handle specific geological conditions and provide the compositional range needed for a system that must remain musically coherent across geological time.

### Granular Synthesis (Primary Engine)

Granular synthesis is the heart of ShadowZone and the primary engine of SeisClaw. It operates by generating a continuous stream of short audio events — grains — each with its own frequency, amplitude, duration, pan position, and envelope. The synthesis texture emerges from the collective behaviour of many grains rather than from any individual sound.

SeisClaw implements three distinct granular clouds operating simultaneously:

**Texture Cloud** — High-density, short-duration grains producing continuous textural wash. Parameters in SeisClaw: density 35–95 grains/second (driven by waveform amplitude), grain duration 35ms (spread ±50%), frequency spread ±1.2 octaves around root×4, pan spread ±1.0 (full stereo field), sine waveform, lowpass filter at 6kHz. The texture cloud's density and frequency spread are directly modulated by the instantaneous amplitude and derivative of the seismic waveform sample at the current playhead position.

**Tone Cloud** — Low-density, long-duration grains producing sustained harmonic content. Parameters: density 5–15 grains/second (driven by RMS energy), grain duration 300ms (spread ±30%), frequency spread ±0.4 octaves around root, pan spread ±0.35, triangle waveform, lowpass at 3kHz. The tone cloud provides the harmonic foundation — it responds to sustained energy in the waveform rather than instantaneous peaks.

**Rhythm Cloud** — Probability-triggered grains whose timing is derived from zero-crossing events detected in the seismic waveform. Parameters: density 14 triggers/second, grain duration 55ms, sawtooth waveform, trigger probability 0.35–0.85 (driven by RMS energy), lowpass at 4kHz with Q=3. Zero-crossings in the seismic data are detected at load time and stored as fractional positions; these become the rhythmic skeleton of the composition.

All three clouds pass through a shared master BiquadFilter (lowpass, 1kHz–10kHz range driven by instantaneous amplitude) before reaching the compressor and reverb/delay chain.

In SeisClaw's expanded architecture, granular synthesis handles: coda wave sequences (complex multi-arrival waveforms), sustained seismic background, moderate-magnitude events (M4.5–6.0), and inter-event ambient states using microseismic background noise as waveform material.

### FM Synthesis (Primary Engine)

FM (Frequency Modulation) synthesis generates complex timbres by modulating the frequency of a carrier oscillator with a modulator oscillator. The ratio of carrier frequency to modulator frequency (C:M ratio) determines the harmonic character; the modulation index (depth of frequency deviation) determines spectral complexity and brightness. Low modulation index produces clean harmonic tones; high modulation index produces dense, inharmonic, metallic textures.

SeisClaw's ANMO FM Synth sibling instrument demonstrates the geological mapping: shallow earthquakes (brittle crust rupture, high-frequency P-wave content) map to high modulation index and wide spectral spread; deep-focus earthquakes (smooth rupture through ductile mantle material, attenuated high frequencies) map to low modulation index and narrow, fundamental-rich timbres.

In SeisClaw, FM synthesis handles: shallow crustal events (depth < 70km, where rupture characteristics produce rich harmonic content), P-wave onset events (sharp initial arrival, high derivative in waveform), significant events (M6.0+, where the spectral complexity of FM matches the geological violence of the rupture), and transition states between other synthesis modes.

The geological-to-FM parameter mappings:

```
Focal depth (km)    → modulation index (shallow = high index, inharmonic)
Magnitude           → carrier amplitude and output gain
P-wave derivative   → modulation ratio (C:M) — fast-changing waveform = wider ratio
Waveform RMS        → operator feedback amount
Event significance  → number of FM operators (2-operator for background, 4-operator for M6+)
```

### Subtractive Synthesis

Subtractive synthesis begins with a harmonically rich source (sawtooth, square, or noise) and sculpts timbre by removing frequency content through filtering. It is the appropriate engine for sustained, slowly-evolving geological conditions — deep-focus earthquakes and mantle events where seismic energy is attenuated into smooth, low-frequency rumble by the time it arrives at the sensor.

In SeisClaw, subtractive synthesis handles: deep-focus events (depth > 300km, where the waveform is dominated by low-frequency energy), slow-onset events (low derivative, gradual energy buildup), teleseismic arrivals (events recorded from distant sources, characterised by long-period surface waves), and extended inter-event drone states.

Parameter mappings:

```
Focal depth         → filter cutoff frequency (deep = low cutoff, darker timbre)
Waveform RMS        → resonance (Q) — higher energy = more resonant character
Magnitude           → oscillator mix (sawtooth vs noise ratio)
Surface wave period → LFO rate modulating filter cutoff
```

### Additive Synthesis

Additive synthesis constructs complex timbres by summing multiple sine wave oscillators, each at a specific frequency and amplitude. Where subtractive synthesis removes, additive synthesis builds from fundamentals up. It is computationally more expensive but offers precise control over harmonic content — particularly useful for events with clear harmonic structure in their waveforms.

In SeisClaw, additive synthesis handles: harmonic tremor events (volcanic or tectonic tremor with clear periodic structure), events where spectral analysis reveals dominant frequency peaks that can be directly reproduced as partials, and MUZAK mode — the freeze state inherited from ShadowZone, where the waveform is held and the synthesis shifts to a more sustained, pitched character. In freeze mode, the additive engine constructs a chord from the current scale and root frequency, voiced according to the geological parameters of the frozen event.

Parameter mappings:

```
Waveform spectral peaks     → partial frequencies and relative amplitudes
Number of dominant peaks    → number of additive partials (2–16)
Waveform RMS                → overall amplitude envelope
Event duration              → partial amplitude decay rate
```

### Wavetable Synthesis

Wavetable synthesis cycles through stored single-cycle waveforms, interpolating between them to create evolving timbres. In SeisClaw, the wavetables are constructed directly from seismic waveform data — a short window of the actual MiniSEED samples is extracted, normalised, and used as the waveform shape for an oscillator. The geological event literally becomes the waveform.

This technique is used for: transitional states between major synthesis modes (the wavetable morphs from one event's waveform to the next as the agent moves between events), mid-magnitude events (M4.5–5.5, where the waveform has sufficient complexity to produce interesting timbres but not the extreme energy of major events), and as a textural element layered beneath FM or granular output during complex events.

The seismic-to-wavetable extraction process:

```
1. At event onset, extract a window of N samples from the parsed MiniSEED array
2. Find a zero-crossing near the centre of the window as the loop point
3. Normalise the window to [-1, +1] range
4. Use this normalised array as the waveform shape for a WaveShaper or 
   PeriodicWave oscillator node
5. Playback frequency is set by the scale/root mapping (same as granular)
6. As the seismic playhead advances, the wavetable is periodically refreshed 
   from the new waveform position — the timbre evolves as the agent moves 
   through the recorded event
```

### Pulsar Synthesis

Pulsar synthesis generates trains of brief acoustic events (pulsars) at varying rates, producing textures that range from tonal (high pulse rate, fusing into pitched sound) to rhythmic (low pulse rate, perceived as discrete events) to noise-like (irregular spacing). The technique was developed by Curtis Roads and is particularly effective for microseismic textures — the continuous low-level seismic noise generated by ocean waves and atmospheric pressure variations.

In SeisClaw, pulsar synthesis is the ambient engine — the mode that keeps the agent composing and broadcasting during inter-event seismic quiet. Microseismic background noise has a characteristic 4–14 second period (the primary and secondary microseism bands generated by ocean swell interactions). This periodic structure drives the pulsar rate, creating a subtle rhythmic underpinning to the ambient texture that is geologically accurate — the pulse of the ocean encoded in seismic noise, rendered as sound.

Parameter mappings:

```
Microseismic noise RMS      → pulsar amplitude
Ocean swell period (~7s)    → primary pulse rate modulation
Noise variance              → pulsar waveform shape (sine → noise)
Background seismic level    → inter-pulsar silence duration
```

---

## Seismic Data Pipeline

This section documents the complete pipeline from network query to normalised Float32Array, based on the proven ShadowZone implementation.

### Stage 1 — Event Discovery: USGS GeoJSON

SeisClaw queries the USGS FDSN event service, not the static GeoJSON feed endpoints. This allows parameterised queries:

```javascript
const CONFIG = {
    minMagnitude: 6.0,
    fetchDays: 30,
    station: { net: "IU", sta: "ANMO", loc: "00", cha: "BHZ" }
};

async function fetchLatestEvent() {
    const end = new Date().toISOString();
    const start = new Date(Date.now() - CONFIG.fetchDays * 86400000).toISOString();
    const url = `https://earthquake.usgs.gov/fdsnws/event/1/query`
              + `?format=geojson`
              + `&starttime=${start}`
              + `&endtime=${end}`
              + `&minmagnitude=${CONFIG.minMagnitude}`
              + `&orderby=time`;
    
    const res = await fetch(url);
    const data = await res.json();
    return data.features[0]; // most recent qualifying event
}
```

The returned GeoJSON feature contains everything the Reason stage needs:

```javascript
const quakeInfo = {
    place:  event.properties.place,              // "47km NNE of Tonga"
    mag:    event.properties.mag,                // 6.1
    depth:  event.geometry.coordinates[2],       // 47.3 (km)
    time:   new Date(event.properties.time).toISOString(), // ISO timestamp
    lon:    event.geometry.coordinates[0],       // longitude
    lat:    event.geometry.coordinates[1]        // latitude
};
```

The event timestamp is the key: it becomes the time window anchor for the MiniSEED waveform request.

### Stage 2 — Waveform Retrieval: EarthScope FDSN

Using the event timestamp, ShadowZone requests 60 seconds of waveform data from EarthScope's FDSN dataselect service, starting 5 seconds before the event origin time (to capture the pre-arrival baseline):

```javascript
async function fetchMiniSEED(eventTime) {
    const start = new Date(new Date(eventTime).getTime() - 5000).toISOString();
    const end   = new Date(new Date(start).getTime() 
                + CONFIG.seismicDurationS * 1000).toISOString();
    
    const { net, sta, loc, cha } = CONFIG.station;
    // IU = Global Seismographic Network
    // ANMO = Albuquerque New Mexico station
    // 00 = location code (primary sensor)
    // BHZ = Broadband High-gain, vertical component
    
    const url = `https://service.earthscope.org/fdsnws/dataselect/1/query`
              + `?net=${net}&sta=${sta}&loc=${loc}&cha=${cha}`
              + `&starttime=${start}&endtime=${end}`
              + `&format=miniseed`;
    
    const res = await fetch(url);
    // MiniSEED returns binary ArrayBuffer — not JSON, not audio
    return await res.arrayBuffer();
}
```

**Station selection notes.** ANMO (Albuquerque, New Mexico) is part of the Global Seismographic Network (GSN) — one of the most complete and consistently archived stations in the world. BHZ (Broadband High-gain Z-axis) captures the vertical ground motion component at 20–40 samples per second, appropriate for the frequency range of teleseismic arrivals from global M6+ events. For SeisClaw's multi-agent architecture, different agents query different stations — each station has its own tectonic character, distance relationships to global events, and noise floor profile.

### Stage 3 — MiniSEED Binary Parsing

MiniSEED is a binary format, not audio. The response `ArrayBuffer` must be parsed to extract the integer sample values. SeisClaw's parser is minimal but functional:

```javascript
function parseMiniSEED(buffer) {
    try {
        const view = new DataView(buffer);
        
        // Sanity check: minimum valid MiniSEED record is 64 bytes
        if (buffer.byteLength < 64) throw new Error("Buffer too small");
        
        // Byte offset 46 (big-endian int16): number of samples in this record
        const numSamples = view.getInt16(46, false);  // false = big-endian
        
        // Data begins at byte 64 in a standard 512-byte MiniSEED record
        // (48 bytes fixed header + 8 bytes Blockette 1000 + 8 bytes padding)
        const dataOffset = 64;
        
        if (numSamples <= 0 || buffer.byteLength < dataOffset + numSamples * 4) {
            throw new Error("Invalid sample count");
        }
        
        // Samples are stored as 32-bit integers (4 bytes each)
        // Int32Array view into the data section of the buffer
        const rawData = new Int32Array(
            buffer.slice(dataOffset, dataOffset + numSamples * 4)
        );
        
        // Normalise to [-1.0, +1.0] range by dividing by peak absolute value
        const maxVal = rawData.reduce((m, v) => Math.max(m, Math.abs(v)), 0);
        if (maxVal === 0) return new Float32Array(rawData.length);
        
        const normalized = new Float32Array(rawData.length);
        for (let i = 0; i < rawData.length; i++) {
            normalized[i] = rawData[i] / maxVal;
        }
        
        return normalized;
        
    } catch (e) {
        // Fallback: synthesise a plausible seismic waveform
        // P-wave onset + S-wave arrival + surface wave coda
        console.error("MiniSEED parse error:", e);
        return generateFallbackWaveform();
    }
}
```

**What the byte offsets mean.** A MiniSEED fixed header is 48 bytes. The critical fields for parsing are:

```
Bytes 0–5:    Sequence number (ASCII, e.g. "000001")
Byte 6:       Data quality indicator (D=data, R=raw, Q=quality-controlled)
Bytes 8–12:   Station identifier (e.g. "ANMO ")
Bytes 13–14:  Location identifier (e.g. "00")
Bytes 15–17:  Channel identifier (e.g. "BHZ")
Bytes 18–19:  Network identifier (e.g. "IU")
Bytes 20–27:  Record start time (BTIME structure: year, day, hour, min, sec, microsec)
Bytes 28–29:  Number of samples (int16, big-endian)  ← offset 28, not 46
Bytes 30–31:  Sample rate factor (int16, big-endian)
Bytes 32–33:  Sample rate multiplier (int16, big-endian)
Bytes 44–45:  Data offset within record (uint16, big-endian)  ← this is the real offset
Bytes 46–47:  Number of blockettes following header
```

**Note on SeisClaw's offset 46.** The ShadowZone parser reads `getInt16(46, false)` for numSamples. The canonical MiniSEED offset for number of samples is byte 28. Offset 46 reads the number of blockettes field, which in a well-formed single-record MiniSEED with one Blockette 1000 happens to contain a value that serves as a workable proxy. For production robustness, SeisClaw's extended parser should read numSamples from offset 28 and read the data start offset from offset 44 (which points past the blockette structure to the actual data). ShadowZone's parser works reliably in practice because EarthScope returns consistently formatted records; the extended parser addresses edge cases.

**The fallback waveform.** When parsing fails or EarthScope returns no data (HTTP 204, network error), SeisClaw synthesises a mathematically plausible seismic waveform — P-wave onset (high-frequency, exponential decay), S-wave arrival at ~2000 samples (slower, larger amplitude), and surface wave coda at ~4000 samples (long-period, slowly decaying). This ensures the synthesis engine always has material to work with. SeisClaw preserves this fallback but adds a SeisConduct state flag indicating the agent is operating on synthetic data.

### Stage 4 — Waveform Analysis

Once the Float32Array is loaded, three analytical quantities are computed and maintained as the playhead advances. These are the direct inputs to the Reason stage:

**Instantaneous amplitude** — absolute value of the current sample at the playhead position. Drives texture cloud density, master filter cutoff, and FM modulation index. This is the raw seismic signal, moment to moment.

**Derivative (rate of change)** — average absolute difference between neighbouring samples around the current index, computed over a ±5 sample window:

```javascript
calculateDerivative(idx, win = 5) {
    if (idx < win || idx >= this.seismicData.length - win) return 0;
    let sum = 0;
    for (let i = 1; i <= win; i++) {
        sum += Math.abs(this.seismicData[idx + i] - this.seismicData[idx - i]);
    }
    return sum / (2 * win);
}
```

High derivative indicates rapid waveform change — P-wave arrivals, rupture onset. Maps to texture spread, FM ratio, pan spread. Low derivative indicates smooth, sustained energy — surface waves, deep-focus arrivals.

**RMS energy** — root mean square over a ±50 sample window around the current index:

```javascript
calculateRMS(idx, win = 50) {
    const start = Math.max(0, idx - win);
    const end   = Math.min(this.seismicData.length, idx + win);
    let sum = 0;
    for (let i = start; i < end; i++) sum += this.seismicData[i] ** 2;
    return Math.sqrt(sum / (end - start));
}
```

RMS is a measure of sustained energy rather than instantaneous peaks. Maps to tone cloud density, rhythm trigger probability, and FM operator amplitude. RMS distinguishes between a single sharp spike (high amplitude, low RMS) and sustained high-energy shaking (high amplitude, high RMS).

**Zero-crossing density** — computed once at load time. The positions of all sign changes in the waveform are stored as fractional indices (crossing position / total length). These become the timing skeleton for the rhythm cloud:

```javascript
detectZeroCrossings(data) {
    const crossings = [];
    for (let i = 1; i < data.length; i++) {
        if (data[i-1] * data[i] < 0) crossings.push(i / data.length);
    }
    return crossings;
}
```

### Stage 5 — Parameter Mapping (Reason Stage)

This is where geological data becomes compositional decision. ShadowZone's `modulateFromSeismic()` is the core mapping function, called on every animation frame as the playhead advances:

```javascript
modulateFromSeismic() {
    const idx    = Math.floor(this.playhead * this.seismicData.length);
    const sample = this.seismicData[idx] || 0;
    const amp    = Math.abs(sample);
    const deriv  = this.calculateDerivative(idx);
    const rms    = this.calculateRMS(idx);
    const now    = this.ctx.currentTime;

    // Master filter: amplitude drives cutoff (louder = brighter)
    const targetFilter = 2000 + amp * 6000;
    this.filter.frequency.setTargetAtTime(clamp(targetFilter, 1000, 10000), now, 0.025);

    // Texture cloud: responds to instantaneous amplitude and waveform rate of change
    this.textureCloud.density          = 35 + amp * 60;      // 35–95 grains/s
    this.textureCloud.amplitude        = 0.06 + amp * 0.08;
    this.textureCloud.frequencySpread  = 0.4 + deriv * 15;   // high deriv = wide spread
    this.textureCloud.panSpread        = 0.5 + deriv * 8;
    this.textureCloud.baseFrequency    = this.textureCloud.rootFreq * 4 * (1 + sample * 0.25);

    // Tone cloud: responds to sustained energy (RMS)
    this.toneCloud.density             = 5 + rms * 10;
    this.toneCloud.frequencySpread     = 0.2 + deriv * 8;
    this.toneCloud.amplitude           = 0.1 + rms * 0.08;

    // Rhythm cloud: probability gate driven by RMS; pitch by RMS
    this.rhythmCloud.triggerProbability = 0.35 + rms * 0.5;  // 35–85%
    this.rhythmCloud.baseFrequency      = this.rhythmCloud.rootFreq * 2 * (1 + rms * 0.4);
    this.rhythmCloud.amplitude          = 0.08 + amp * 0.08;

    // Advance playhead
    this.playhead += CONFIG.playbackSpeed / this.stretchFactor;
    if (this.playhead >= 1) this.playhead = 0;
}
```

**Depth-to-reverb mapping.** ShadowZone maps focal depth to the reverb and delay chain separately from the real-time modulation loop, set once when the event is loaded:

```javascript
const df = Math.min(1, quakeInfo.depth / 600); // normalise depth: 0 (surface) → 1 (600km)

reverbWet.gain.setTargetAtTime(0.35 + df * 0.35, audioCtx.currentTime, 0.5); // 35–70% wet
delay.delayTime.setTargetAtTime(0.3 + df * 0.35, audioCtx.currentTime, 0.5); // 300–650ms
delayFeedback.gain.setTargetAtTime(0.35 + df * 0.25, audioCtx.currentTime, 0.5);
```

Deep events sound spacious and resonant — the geology of mantle travel time encoded as acoustic space.

---

## Step-by-Step Build Plan

This is a concrete, ordered implementation sequence. Each step produces a testable outcome. The plan is structured so that each completed step is deployable — work can stop at any step and what exists is functional.

### Step 1 — Establish the Agent Wrapper

**Objective:** Make SeisClaw autonomous. Remove the Begin button dependency. The agent composes after a single user gesture (satisfying browser AudioContext autoplay policy), then runs indefinitely without further input.

**What to build:**
- Move the `start()` call from the Begin button handler to a one-time `click` / `touchstart` listener on `document.body`
- After that first gesture, the agent initialises audio, fetches data, and begins composing
- The AudioContext is created and immediately resumed in the gesture handler — this is the browser autoplay unlock
- Remove the Begin, End, and Muzak buttons from the primary interface (retain as hidden developer controls accessible via keyboard shortcut)
- Implement an autonomous polling loop: after initial composition begins, schedule a data refresh every 45 seconds using `setInterval`. On each refresh, fetch the latest USGS event. If a new event ID is detected, queue a synthesis transition to the new waveform

**Test:** Page loads. Single click/tap anywhere. Agent begins composing without any further input. After 45 seconds, polling fires and logs a check to console. If a new event has occurred since load, synthesis updates.

---

### Step 2 — Implement the Explicit SeisConduct State Machine

**Objective:** Give the SCLoop a formal, observable structure. The agent's current phase is tracked in a state object and surfaced to the UI.

**What to build:**

A lightweight JavaScript state machine object — no external library at this stage:

```javascript
const AgentState = {
    PERCEIVING:  'PERCEIVING',
    REASONING:   'REASONING',
    PLANNING:    'PLANNING',
    ACTING:      'ACTING',
    REFLECTING:  'REFLECTING',
    AMBIENT:     'AMBIENT'   // inter-event: no qualifying event in feed
};

let currentState = AgentState.PERCEIVING;

function transition(newState, context = {}) {
    const prev = currentState;
    currentState = newState;
    logTransition(prev, newState, context);
    renderState(newState, context);
}
```

SeisConduct stage transitions map to the existing SeisClaw flow:

```
fetchLatestEvent() called          → transition(PERCEIVING)
event data received                → transition(REASONING, { event })
synthesis strategy selected        → transition(PLANNING, { strategy, params })
granularEngine.start() called      → transition(ACTING)
after N seconds of ACTING          → transition(REFLECTING)
REFLECTING complete                → transition(PERCEIVING)  [SCLoop continues]
no qualifying events in feed       → transition(AMBIENT)
```

The AMBIENT state runs pulsar synthesis for microseismic background texture rather than the main granular engine. It transitions back to PERCEIVING on the next successful event fetch.

**Test:** Open browser console. All state transitions print with timestamp, previous state, new state, and context object. UI displays current SCLoop stage label in real time.

---

### Step 3 — Build the Reasoning Display

**Objective:** Make the agent's decisions visible in real time. This is the interface surface where audiences see what the agent is doing and why — the SeisConduct loop made legible.

**What to build:**

A fixed-position panel (bottom of viewport, full width, minimal height) rendering the current SeisConduct state in monospace:

```
SCLoop     ACTING
EVENT      M6.1 · 47km · Tonga Trench  
REASONING  depth→reverb 58% · amp→texture 67grains/s · rms→tone 8grains/s
SYNTHESIS  granular:3-layer · scale:dorian · root:A2 · stretch:1.0×
FILTER     3840Hz · DERIV 0.024 · RMS 0.31
UPDATED    11:42:07 UTC
```

The display updates every animation frame for the live metrics (FILTER, DERIV, RMS) and on state transitions for the categorical fields (SCLoop stage, EVENT, SYNTHESIS).

This display is honest: every value shown is a real computed value from real data. No fabricated atmosphere. The geological event drives the numbers; the numbers are shown.

**Test:** Display updates in real time. FILTER frequency value changes continuously as the waveform playhead advances. SCLoop stage changes visibly on transitions. Values match what is audible — when grain density increases, the number increases.

---

### Step 4 — Extended MiniSEED Parser

**Objective:** Replace ShadowZone's functional-but-minimal parser with a robust implementation that handles multi-record files, reads the canonical header offsets, and provides richer waveform metadata.

**What to build:**

A complete MiniSEED reader that handles the actual header structure:

```javascript
function parseMiniSEEDExtended(buffer) {
    const view = new DataView(buffer);
    const records = [];
    let offset = 0;
    
    while (offset + 48 <= buffer.byteLength) {
        // Byte 28: number of samples (int16, big-endian)
        const numSamples = view.getInt16(offset + 28, false);
        
        // Byte 30-31: sample rate factor and multiplier
        const srFactor     = view.getInt16(offset + 30, false);
        const srMultiplier = view.getInt16(offset + 32, false);
        const sampleRate   = decodeSampleRate(srFactor, srMultiplier);
        
        // Byte 44: beginning of data offset within record (uint16, big-endian)
        const dataStartOffset = view.getUint16(offset + 44, false);
        const dataAbsOffset   = offset + dataStartOffset;
        
        // Byte 20-27: BTIME record start time
        const year    = view.getUint16(offset + 20, false);
        const dayOfYr = view.getUint16(offset + 22, false);
        const hour    = view.getUint8(offset + 24);
        const minute  = view.getUint8(offset + 25);
        const second  = view.getUint8(offset + 26);
        
        // Data encoding: Blockette 1000 at first blockette offset
        // Byte 37: number of blockettes; byte 38-39: first blockette offset
        const firstBlocketteOffset = view.getUint16(offset + 46, false);
        const encoding = view.getUint8(offset + firstBlocketteOffset + 4);
        // encoding 11 = 32-bit int (Steim-2 compressed) — decompress needed
        // encoding 3  = 32-bit int (uncompressed) — direct Int32Array
        
        if (numSamples > 0 && dataAbsOffset + numSamples * 4 <= buffer.byteLength) {
            const rawData = new Int32Array(
                buffer.slice(dataAbsOffset, dataAbsOffset + numSamples * 4)
            );
            records.push({ rawData, sampleRate, year, dayOfYr, hour, minute, second });
        }
        
        // Records are always a power-of-2 size; default 512 bytes
        offset += 512;
    }
    
    // Concatenate all records into a single sample array
    const totalSamples = records.reduce((sum, r) => sum + r.rawData.length, 0);
    const combined = new Int32Array(totalSamples);
    let pos = 0;
    for (const r of records) {
        combined.set(r.rawData, pos);
        pos += r.rawData.length;
    }
    
    // Normalise to [-1, +1]
    const maxVal = combined.reduce((m, v) => Math.max(m, Math.abs(v)), 0);
    if (maxVal === 0) return { samples: new Float32Array(combined.length), sampleRate: 40 };
    
    const normalised = new Float32Array(combined.length);
    for (let i = 0; i < combined.length; i++) normalised[i] = combined[i] / maxVal;
    
    return { samples: normalised, sampleRate: records[0]?.sampleRate || 40 };
}

function decodeSampleRate(factor, multiplier) {
    // MiniSEED sample rate encoding: positive values are direct rates/periods;
    // negative values are factors where actual rate = 1 / (|factor| * |multiplier|)
    if (factor > 0 && multiplier > 0) return factor * multiplier;
    if (factor > 0 && multiplier < 0) return -factor / multiplier;
    if (factor < 0 && multiplier > 0) return -multiplier / factor;
    if (factor < 0 && multiplier < 0) return 1 / (factor * multiplier);
    return 1;
}
```

The extended parser returns both the normalised sample array and the actual sample rate, enabling time-accurate playback speed calculation and richer event metadata for the Reason stage display.

**Test:** Parser returns correct sample count and rate for EarthScope BHZ records. Multi-record files (longer time windows) concatenate correctly. Fallback triggers cleanly on corrupt or empty responses.

---

### Step 5 — FM Synthesis Engine

**Objective:** Build the FM synthesis engine that handles shallow crustal events, P-wave onset events, and M6+ significant events.

**What to build:**

A self-contained FM engine class with a geological parameter interface:

```javascript
class FMSynthEngine {
    constructor(ctx, dest) {
        this.ctx = ctx;
        this.destination = dest;
        this.operators = [];
        this.masterGain = ctx.createGain();
        this.masterGain.gain.value = 0;
        this.masterGain.connect(dest);
    }
    
    // Configure from geological event parameters (Plan stage)
    configureFromEvent(event) {
        const { mag, depth } = event;
        
        // Shallow = inharmonic (high modIndex); deep = harmonic (low modIndex)
        const modIndex   = Math.max(0.5, 8 - (depth / 70));
        
        // Higher magnitude = more operators (richer spectrum)
        const numOps     = mag >= 7.0 ? 4 : mag >= 6.0 ? 3 : 2;
        
        // C:M ratio: fast-rupture shallow events use wider ratios
        const cmRatio    = depth < 70 ? 2.1 + (7 - depth / 10) * 0.3 : 1.5;
        
        return { modIndex, numOps, cmRatio, carrierAmp: 0.1 + (mag - 4) * 0.04 };
    }
    
    // Modulate FM parameters in real time from waveform analytics (Act stage)
    modulateFromSeismic(amp, deriv, rms) {
        if (this.operators.length < 2) return;
        const now = this.ctx.currentTime;
        
        // Modulation depth tracks waveform derivative (fast changes = wider FM)
        const modDepth = this.baseModIndex * (1 + deriv * 12);
        this.operators[1].frequency.setTargetAtTime(
            this.carrierFreq * this.cmRatio * (1 + amp * 0.15), now, 0.02
        );
        
        // Operator amplitude (index) tracks amplitude
        this.modGain.gain.setTargetAtTime(
            this.carrierFreq * modDepth, now, 0.02
        );
        
        // Output level tracks RMS
        this.masterGain.gain.setTargetAtTime(
            this.carrierAmp * (0.5 + rms), now, 0.05
        );
    }
}
```

**Test:** FM engine produces audible output on M6+ events with depth < 70km. Modulation index is perceptibly higher for shallow events than deep events. Timbre changes in real time as the seismic playhead advances.

---

### Step 6 — Pulsar Synthesis Engine (Ambient State)

**Objective:** Build the ambient engine that keeps SeisClaw composing during inter-event seismic quiet. Pulsar synthesis generates microseismic background texture with the characteristic ocean-swell period (~7 seconds).

**What to build:**

```javascript
class PulsarEngine {
    constructor(ctx, dest) {
        this.ctx = ctx;
        this.destination = dest;
        this.isActive = false;
        this.pulseRate = 0.14;       // ~7s period: 1/7 Hz
        this.pulseWidth = 0.08;      // pulse duration as fraction of period
        this.masterGain = ctx.createGain();
        this.masterGain.gain.value = 0;
        this.masterGain.connect(dest);
    }
    
    // Each pulsar: brief burst of filtered noise shaped by an amplitude envelope
    schedulePulsar() {
        if (!this.isActive) return;
        const now = this.ctx.currentTime;
        
        const noise = this.ctx.createOscillator();
        noise.type = 'sawtooth';
        noise.frequency.value = 40 + Math.random() * 80; // sub-bass range
        
        const filter = this.ctx.createBiquadFilter();
        filter.type = 'bandpass';
        filter.frequency.value = 80 + Math.random() * 200;
        filter.Q.value = 0.8;
        
        const env = this.ctx.createGain();
        const dur = this.pulseWidth / this.pulseRate;
        env.gain.setValueAtTime(0, now);
        env.gain.linearRampToValueAtTime(0.04, now + dur * 0.1);
        env.gain.exponentialRampToValueAtTime(0.001, now + dur);
        
        noise.connect(filter);
        filter.connect(env);
        env.connect(this.masterGain);
        
        noise.start(now);
        noise.stop(now + dur + 0.1);
        
        // Schedule next pulsar with slight jitter (ocean irregularity)
        const interval = (1 / this.pulseRate) * (0.9 + Math.random() * 0.2);
        setTimeout(() => this.schedulePulsar(), interval * 1000);
    }
    
    start(noiseLevel = 0.3) {
        this.isActive = true;
        this.pulseRate = 0.12 + noiseLevel * 0.05; // quieter = slower pulse
        this.masterGain.gain.setTargetAtTime(0.3, this.ctx.currentTime, 2);
        this.schedulePulsar();
    }
    
    stop() {
        this.isActive = false;
        this.masterGain.gain.setTargetAtTime(0, this.ctx.currentTime, 1);
    }
}
```

**Test:** AMBIENT state activates pulsar engine. Audible low-frequency pulse at approximately 7-second intervals with natural variation. Fades gracefully when a new seismic event triggers transition back to PERCEIVING.

---

### Step 7 — Wavetable and Additive Engines

**Objective:** Build the two remaining synthesis engines to cover mid-magnitude events (wavetable) and harmonic tremor / MUZAK freeze states (additive).

**Wavetable engine** — extracts a single-cycle waveform window from the seismic data and uses it as a `PeriodicWave` oscillator shape. The window is refreshed periodically as the playhead advances, so the timbre evolves with the seismic waveform:

```javascript
class WavetableEngine {
    constructor(ctx, dest) {
        this.ctx = ctx;
        this.destination = dest;
        this.oscillators = [];
        this.masterGain = ctx.createGain();
        this.masterGain.connect(dest);
    }
    
    buildFromWaveform(seismicData, playhead, windowSize = 256) {
        const centre = Math.floor(playhead * seismicData.length);
        const start  = Math.max(0, centre - windowSize / 2);
        const window = seismicData.slice(start, start + windowSize);
        
        // Find a zero crossing for clean loop point
        let loopPoint = 0;
        for (let i = 1; i < window.length; i++) {
            if (window[i - 1] * window[i] < 0) { loopPoint = i; break; }
        }
        
        // Build real/imag arrays for PeriodicWave from windowed FFT
        // or use the window directly as the real component
        const real = new Float32Array(windowSize);
        const imag = new Float32Array(windowSize);
        for (let i = 0; i < windowSize; i++) real[i] = window[i] || 0;
        
        return this.ctx.createPeriodicWave(real, imag, { disableNormalization: false });
    }
    
    setWave(periodicWave, frequency, amplitude) {
        this.cleanup();
        const osc = this.ctx.createOscillator();
        osc.setPeriodicWave(periodicWave);
        osc.frequency.value = frequency;
        const gain = this.ctx.createGain();
        gain.gain.value = amplitude;
        osc.connect(gain);
        gain.connect(this.masterGain);
        osc.start();
        this.oscillators.push({ osc, gain });
    }
    
    cleanup() {
        this.oscillators.forEach(({ osc }) => { try { osc.stop(); } catch(e) {} });
        this.oscillators = [];
    }
}
```

**Additive engine** — sums sine oscillators at harmonically related frequencies. For MUZAK freeze mode, builds a chord voicing from the current scale and root. For harmonic tremor events, analyses the waveform spectrum and voices partials to match detected frequency peaks:

```javascript
class AdditiveEngine {
    constructor(ctx, dest) {
        this.ctx = ctx;
        this.destination = dest;
        this.partials = [];
        this.masterGain = ctx.createGain();
        this.masterGain.connect(dest);
    }
    
    buildChord(rootFreq, scale, chordVoicing = [0, 4, 7]) {
        this.cleanup();
        const intervals = SCALES[scale] || SCALES.pentatonic;
        
        chordVoicing.forEach((degree, i) => {
            const interval = intervals[degree % intervals.length] || 0;
            const freq     = rootFreq * Math.pow(2, interval / 12);
            const amp      = 0.06 * (1 - i * 0.15); // upper partials quieter
            
            const osc  = this.ctx.createOscillator();
            const gain = this.ctx.createGain();
            osc.type = 'sine';
            osc.frequency.value = freq;
            gain.gain.value = 0;
            gain.gain.setTargetAtTime(amp, this.ctx.currentTime, 0.5);
            
            osc.connect(gain);
            gain.connect(this.masterGain);
            osc.start();
            this.partials.push({ osc, gain });
        });
    }
    
    cleanup() {
        this.partials.forEach(({ osc }) => { try { osc.stop(); } catch(e) {} });
        this.partials = [];
    }
}
```

**Subtractive engine:**

```javascript
class SubtractiveEngine {
    constructor(ctx, dest) {
        this.ctx = ctx;
        this.filter = ctx.createBiquadFilter();
        this.filter.type = 'lowpass';
        this.filter.frequency.value = 800;
        this.filter.Q.value = 2;
        this.masterGain = ctx.createGain();
        this.filter.connect(this.masterGain);
        this.masterGain.connect(dest);
    }
    
    configureFromEvent(depth, mag) {
        // Deep events: very dark, low cutoff; shallow events: brighter
        const cutoff = clamp(200 + (600 - depth) * 8, 200, 4000);
        this.filter.frequency.setTargetAtTime(cutoff, this.ctx.currentTime, 1.0);
        this.filter.Q.value = 1 + (depth / 100); // deeper = more resonant
    }
    
    modulateFromSeismic(amp, rms) {
        const now    = this.ctx.currentTime;
        const cutoff = this.filter.frequency.value;
        this.filter.frequency.setTargetAtTime(cutoff * (1 + amp * 0.3), now, 0.05);
        this.masterGain.gain.setTargetAtTime(0.15 + rms * 0.2, now, 0.1);
    }
}
```

**Synthesis router (Plan stage):**

```javascript
function selectSynthesisStrategy(event) {
    const { mag, depth } = event;
    
    // Granular: primary engine for moderate events and coda phases
    if (mag >= 4.5 && mag < 6.5 && depth >= 70)  return 'GRANULAR';
    
    // FM: shallow and/or significant events
    if (depth < 70 || mag >= 6.5)                 return 'FM';
    
    // Subtractive: deep-focus events
    if (depth >= 300)                              return 'SUBTRACTIVE';
    
    // Wavetable: mid-magnitude transitional states
    if (mag >= 4.5 && mag < 6.5 && depth < 300)   return 'WAVETABLE';
    
    // Additive: harmonic tremor or MUZAK freeze
    // (triggered by MUZAK mode, not event properties)
    
    // Pulsar: ambient/inter-event (AMBIENT state, not event-driven)
    
    // Default
    return 'GRANULAR';
}
```

The router informs the Plan stage transition. The selected strategy label appears in the SeisConduct reasoning display.

**Test:** Router selects FM for a simulated M7.2 at 35km depth. Router selects SUBTRACTIVE for a simulated event at 580km. Router selects GRANULAR for an M5.0 at 120km. All transitions display correctly in the reasoning panel.

---

### Step 9 — Compositional Memory and Reflect Stage

**Objective:** Give each agent a compositional history. The Reflect stage evaluates the current composition against accumulated history and generates LLM commentary. History influences future synthesis decisions through deterministic rules.

**What to build:**

Compositional history stored in `localStorage` as a rolling array (capped at 200 events):

```javascript
const CompositionLog = {
    key: 'seisclaw-agent-1-log',
    
    add(entry) {
        const log = this.get();
        log.unshift({ ...entry, timestamp: Date.now() });
        if (log.length > 200) log.length = 200;
        try { localStorage.setItem(this.key, JSON.stringify(log)); } catch(e) {}
    },
    
    get() {
        try { return JSON.parse(localStorage.getItem(this.key) || '[]'); } catch(e) { return []; }
    },
    
    // How many of the last N events used each synthesis strategy?
    strategyDistribution(n = 20) {
        const recent = this.get().slice(0, n);
        const counts = { GRANULAR: 0, FM: 0, SUBTRACTIVE: 0, WAVETABLE: 0, ADDITIVE: 0, PULSAR: 0 };
        recent.forEach(e => { if (counts[e.strategy] !== undefined) counts[e.strategy]++; });
        return counts;
    }
};
```

**History-influenced planning.** If the last 10 events have all used GRANULAR synthesis, the router's thresholds shift to increase probability of selecting FM or WAVETABLE for the next qualifying event. This is deterministic — not ML — and produces genuine compositional variety over time:

```javascript
function selectSynthesisStrategy(event) {
    const dist = CompositionLog.strategyDistribution(10);
    
    // If GRANULAR has been overused recently, lower its priority
    const granularBias = dist.GRANULAR > 7 ? -1.5 : 0;
    const fmBias       = dist.FM       < 2 ? +1.0 : 0;
    
    // Apply biases to magnitude thresholds
    const fmMagThreshold = 6.5 + granularBias + fmBias;
    
    if (event.depth < 70 || event.mag >= fmMagThreshold) return 'FM';
    // ... rest of routing logic
}
```

**LLM reflection.** The Reflect stage calls the LLM API with the current event, strategy, and a window of recent history. The response is a single sentence rendered as the reflection caption. The call is made through a server-side proxy to protect the API key. If the call fails or times out, the instrument is unaffected — the SCLoop continues and reflection simply doesn't update:

```javascript
async function generateReflection(event, strategy, recentLog) {
    const prompt = `You are a seismic sonification agent. One sentence only. 
    Current event: M${event.mag} at ${event.depth}km depth near ${event.place}.
    Synthesis: ${strategy}. 
    Recent history: ${recentLog.slice(0,5).map(e => e.strategy).join(', ')}.
    Comment briefly on this composition decision in geological terms.`;
    
    try {
        const res = await fetch('/api/reflect', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ prompt })
        });
        const data = await res.json();
        return data.text || null;
    } catch(e) {
        return null; // silent failure — instrument unaffected
    }
}
```

**Test:** After 5 SCLoop cycles, `CompositionLog.get()` returns an array of logged entries. `strategyDistribution()` correctly reflects the actual strategies used. LLM reflection displays within 5 seconds when API is available. Instrument functions identically when API call times out.

---

### Step 10 — Multi-Agent Architecture

**Objective:** Deploy three distinct SeisClaw agents on seisclaw.com, each with its own compositional identity, synthesis emphasis, and station data source. Each agent runs its own independent SCLoop.

**Agent 1: The Lithospheric Conductor** (base instrument, from Steps 1–9)
Station: IU.ANMO (Albuquerque, NM). Primary engines: Granular + FM. Scale defaults: Dorian, Pentatonic. Compositional character: geological and slow. This agent hears the world from a mid-continent perspective.

**Agent 2: The P-Wave Percussionist**
Station: IU.COLA (College, Alaska — seismically active, close to Pacific Rim sources). Primary engines: FM + Granular rhythm layer only (texture and tone suppressed). Compositional character: rhythmic, percussive. P-wave arrivals are treated as kick triggers. The rhythm cloud's triggerProbability is pushed to 0.9 for qualifying events. Scale defaults: Fifths, Minor. The Percussionist hears arrivals as beats.

**Agent 3: The Microseismic Ambientist**
Station: IU.MAJO (Matsushiro, Japan — sensitive station, excellent microseismic recording). Primary engine: Pulsar + Granular texture only (tone and rhythm suppressed). Compositional character: oceanic, continuous, never silent. This agent specialises in background microseismic noise. Even during seismic quiet, MAJO records the pulse of the Pacific Ocean as seismic energy. The Ambientist is always composing.

Each agent lives at its own URL (`seisclaw.com/conductor`, `/percussionist`, `/ambientist`) and in a panel on the central dashboard at `seisclaw.com/agents`. Each maintains its own `localStorage` compositional log under a unique key and runs an independent SeisConduct loop.

**Test:** All three agents operate simultaneously in separate browser tabs without audio interference. Each agent's reasoning display shows different stations, different synthesis strategies, different event responses to the same global seismic data.

---

### Step 11 — Audio Worklet Migration

**Objective:** Move DSP off the main thread. Replace the `setTimeout`-based grain scheduling with an Audio Worklet processor running at 128-sample blocks on a dedicated audio thread. This eliminates any possibility of main-thread activity (data fetches, SeisConduct state transitions, LLM API calls) causing audio stutter.

**What to build:**

A `seismic-processor.js` worklet file (placed in the `/public` directory for reliable loading — not bundled through Vite):

```javascript
// /public/audio/seismic-processor.js
// Runs in AudioWorkletGlobalScope — separate thread from main page

class SeismicProcessor extends AudioWorkletProcessor {
    constructor(options) {
        super();
        this.params = {
            grainRate:   30,    // grains per second
            grainAmp:    0.08,
            filterFreq:  4000,
            pan:         0
        };
        // SharedArrayBuffer for zero-latency parameter updates from main thread
        if (options.processorOptions?.paramBuffer) {
            this.paramView = new Float32Array(options.processorOptions.paramBuffer);
        }
        this.port.onmessage = (e) => {
            if (e.data.type === 'params') Object.assign(this.params, e.data.payload);
        };
        this.phase     = 0;
        this.grainPhase = 0;
    }
    
    process(_inputs, outputs) {
        const out = outputs[0][0];
        
        // Read latest params from SharedArrayBuffer if available
        if (this.paramView) {
            this.params.grainAmp  = this.paramView[0];
            this.params.filterFreq = this.paramView[1];
        }
        
        for (let i = 0; i < out.length; i++) {
            // Simple grain trigger: sine burst at grain rate
            this.grainPhase += this.params.grainRate / sampleRate;
            if (this.grainPhase >= 1) this.grainPhase -= 1;
            const inGrain = this.grainPhase < 0.1;
            
            this.phase += 440 / sampleRate;
            if (this.phase > 1) this.phase -= 1;
            
            out[i] = inGrain 
                ? Math.sin(2 * Math.PI * this.phase) * this.params.grainAmp
                : 0;
        }
        
        return true; // keep processor alive
    }
}

registerProcessor('seismic-processor', SeismicProcessor);
```

Loading the worklet from the main thread:

```javascript
async function initWorklet() {
    await audioCtx.audioWorklet.addModule('/audio/seismic-processor.js');
    
    const paramBuffer = new SharedArrayBuffer(16 * Float32Array.BYTES_PER_ELEMENT);
    const paramView   = new Float32Array(paramBuffer);
    
    const workletNode = new AudioWorkletNode(audioCtx, 'seismic-processor', {
        processorOptions: { paramBuffer }
    });
    workletNode.connect(audioCtx.destination);
    
    return { workletNode, paramView };
}
```

The SharedArrayBuffer requires `Cross-Origin-Isolated` headers (COOP + COEP) — configured in `_headers` for Cloudflare Pages:

```
/*
  Cross-Origin-Opener-Policy: same-origin
  Cross-Origin-Embedder-Policy: require-corp
```

**Test:** Audio plays without interruption while a USGS fetch is in progress. Grain scheduling continues at consistent rate regardless of main-thread load. SeisConduct state transitions and LLM API calls do not cause audio dropouts.

---

### Step 12 — Zod Validation, TypeScript Migration, and Architecture Hardening

**Objective:** Add runtime API validation at the network boundary and begin progressive TypeScript migration. These are robustness improvements that protect against real-world API variability.

**Zod validation** https://zod.dev/ for the USGS feed — the schema catches null magnitudes, missing coordinates, and API format changes that would otherwise silently corrupt the Reason stage:

```typescript
import { z } from 'zod';

const SeismicFeatureSchema = z.object({
    id: z.string(),
    properties: z.object({
        mag:   z.number().nullable().transform(v => v ?? 0),
        place: z.string().nullable().transform(v => v ?? 'Unknown location'),
        time:  z.number().positive(),
        type:  z.string().default('earthquake'),
        sig:   z.number().default(0),
        alert: z.string().nullable(),
    }),
    geometry: z.object({
        coordinates: z.tuple([z.number(), z.number(), z.number()])
    })
});

const USGSFeedSchema = z.object({
    type:     z.literal('FeatureCollection'),
    features: z.array(SeismicFeatureSchema)
});

async function fetchLatestEventValidated(url) {
    const res  = await fetch(url);
    const raw  = await res.json();
    const parsed = USGSFeedSchema.safeParse(raw);
    
    if (!parsed.success) {
        console.error('USGS schema validation failed:', parsed.error.message);
        return null;
    }
    return parsed.data.features[0] || null;
}
```

**TypeScript migration** proceeds file by file, starting with the data layer (`usgs.ts`, `miniseed.ts`), then the synthesis engines, then the SeisConduct state machine. The existing JavaScript instruments remain functional throughout — TypeScript compiles to JavaScript alongside them.

**Dependencies introduced at this step** (the first external packages in the build):

```json
{
  "typescript": "^5.5",
  "vite":       "^6.x",
  "zod":        "^3.23"
}
```

XState v5 is introduced as an optional upgrade path for the SeisConduct state machine — the lightweight JavaScript state object from Step 2 remains functional and can be replaced with the XState formal machine incrementally.

**Test:** `npm run build` compiles without TypeScript errors. Zod validation catches a simulated null-magnitude event and routes it to AMBIENT state rather than crashing. Validated data service returns identical results to the original fetch for well-formed USGS responses.

---

## Dependency Manifest

| Package | Purpose | Introduced |
|---|---|---|
| None | Steps 1–10: vanilla JavaScript, zero external dependencies | Steps 1–10 |
| `typescript` ^5.5 | Type safety for data and synthesis layers | Step 12 |
| `vite` ^6.x | Build system for TypeScript compilation | Step 12 |
| `zod` ^3.23 | Runtime USGS/EarthScope API validation | Step 12 |
| `xstate` ^5.x | Optional formal SeisConduct state machine (replaces Step 2 object) | Step 12+ |

No React, Vue, or Svelte. No icon libraries. No audio polyfills — Web Audio API and AudioWorklet are universally supported in 2026 browsers. Each dependency must justify itself. The JavaScript instruments from SOS remain dependency-free throughout.

---

## File Structure

```
seisclaw.com/
├── index.html               ← Lithospheric Conductor (Agent 1, root URL)
├── agents.html              ← Three-agent dashboard
├── about.html               ← Project documentation
├── log.html                 ← Compositional event log
│
├── /public
│   └── /audio
│       └── seismic-processor.js   ← AudioWorklet (not bundled)
│
├── /src                     ← TypeScript source (Step 12 onwards)
│   ├── /core
│   │   ├── types.ts
│   │   ├── machine.ts       ← XState v5 SeisConduct state machine
│   │   └── log.ts           ← Compositional memory
│   ├── /data
│   │   ├── usgs.ts          ← Validated event fetching
│   │   └── miniseed.ts      ← Extended MiniSEED parser
│   └── /synthesis
│       ├── granular.ts
│       ├── fm.ts
│       ├── subtractive.ts
│       ├── additive.ts
│       ├── wavetable.ts
│       └── pulsar.ts
│
├── /api
│   └── reflect.js           ← LLM proxy (Cloudflare Worker)
│
├── _headers                 ← COOP/COEP headers (Cloudflare Pages)
├── vite.config.ts
├── tsconfig.json
└── package.json
```

---

## Musical Scale System

Inherited from ShadowZone. All synthesis engines use the same scale quantization function — every grain, FM carrier, additive partial, and wavetable oscillator is quantized to the selected scale and root before output:

```javascript
const SCALES = {
    pentatonic: [0, 2, 4, 7, 9],
    minor:      [0, 2, 3, 5, 7, 8, 10],
    dorian:     [0, 2, 3, 5, 7, 9, 10],
    harmonic:   [0, 12, 19, 24, 28, 31, 34, 36],
    fifths:     [0, 7, 14, 21, 28]
};

function quantizeToScale(freq, scale, rootFreq) {
    const intervals = SCALES[scale] || SCALES.pentatonic;
    const freqs = [];
    for (let oct = -3; oct <= 4; oct++) {
        for (const iv of intervals) {
            const f = rootFreq * Math.pow(2, oct) * Math.pow(2, iv / 12);
            if (f >= 25 && f <= 8000) freqs.push(f);
        }
    }
    return freqs.reduce((nearest, f) =>
        Math.abs(f - freq) < Math.abs(nearest - freq) ? f : nearest
    , freqs[0]);
}
```

Root frequency options (Hz): A1 (55), D2 (73.42), E2 (82.41), A2 (110), D3 (146.83).

---

## Audio Chain

The complete signal chain, from synthesis engine output to speakers:

```
[GranularEngine / FMEngine / SubtractiveEngine / WavetableEngine / AdditiveEngine / PulsarEngine]
    ↓ (masterGain per engine)
[BiquadFilter — lowpass, seismically modulated cutoff]
    ↓
[DynamicsCompressor — threshold -12dB, ratio 3:1, attack 5ms, release 200ms]
    ↓                           ↓
[ConvolverNode — reverb]    [DelayNode — 375ms + feedback]
[GainNode — reverbWet]      [GainNode — delayWet]
    ↓                           ↓
[masterGain — global output level]
    ↓
[AudioContext.destination]
```

Reverb character: 3.5-second convolution reverb synthesised from exponentially-decaying noise with early reflection component. Reverb wet level (35–70%) is set at event load time, driven by focal depth. Delay time (300–650ms) is also depth-driven.

---

## Licence

MIT · sos.allshookup.org · seisclaw.com · github.com/SeisClaw

*The earth composes. The agents perform.*

*SeisClaw — SOS Agentic Architecture · ©© SH00K \*02026*
