# SEISCLAW  — System Architecture

**Seismic Waveform Electronica**
*Three-layer granular synthesis driven by live earthquake data*

seisclaw.com · sos.allshookup.org · github.com/strikeslip/SeisClaw

---

## Overview

SEISCLAW is a browser-based seismic sonification engine that transforms real earthquake waveform data into generative electronic music. Built entirely in vanilla Web Audio API with no external libraries, it fetches live seismic data, decodes raw binary MiniSEED format in-browser, and drives a three-layer granular synthesis architecture where every musical parameter — density, frequency, stereo spread, filter cutoff — is continuously modulated by the Earth's seismic voice.

The system is structured in four layers: external data sources, data processing, granular synthesis, and effects/output.

---

## Layer 0 — External Data Sources

### USGS FDSN Event API

- Endpoint: `earthquake.usgs.gov/fdsnws/event/1/query`
- Format: GeoJSON
- Query: M6.0+ events from the last 30 days, ordered by time (most recent first)
- Extracted fields: magnitude, depth (km), longitude/latitude, origin time, USGS event URL

### EarthScope FDSN Dataselect

- Endpoint: `service.earthscope.org/fdsnws/dataselect/1/query`
- Format: Raw binary MiniSEED
- Channel: BHZ (broadband, high-gain, vertical component)
- Time window: event origin − 5 seconds → event origin + 60 seconds
- Parameter `&nodata=404` for proper empty-response handling (EarthScope AWS migration fix)

### Station Fallback Chain

Six IU (Global Seismographic Network) stations are tried in sequence. If a station returns 204/404/error or times out (15s), the next station is attempted automatically:

1. **IU.ANMO** — Albuquerque, New Mexico (34.95°N, 106.46°W)
2. **IU.COLA** — College, Alaska (64.87°N, 147.86°W)
3. **IU.GUMO** — Guam (13.59°N, 144.87°E)
4. **IU.HNR** — Honiara, Solomon Islands (9.44°S, 159.95°E)
5. **IU.MAJO** — Matsushiro, Japan (36.55°N, 138.20°E)
6. **IU.MIDW** — Midway Island (28.22°N, 177.37°W)

If all stations fail, the system continues with the synthetic waveform (see Layer 1).

---

## Layer 1 — Data Processing

### Instant-Start Strategy

SEISCLAW uses an instant-start architecture with background hot-swap — the core UX decision of v3:

1. **Immediate**: A synthetic waveform is generated from earthquake parameters (magnitude scales amplitude, depth scales decay constants) and playback begins with zero latency.
2. **Background**: Real MiniSEED data is fetched from EarthScope, walking the station fallback chain.
3. **Hot-swap**: When real data arrives, it replaces the synthetic waveform in the running granular engine with no audible gap. The `loadSeismicData()` method swaps the `seismicData` array and recalculates zero-crossings mid-playback.

### Synthetic Waveform Generator

The synthetic fallback models three seismic wave phases:

- **P-wave**: `sin(i × (0.1 + magScale × 0.1)) × exp(-i / (1000 + depthScale × 1000))` — fast onset, exponential decay
- **S-wave**: Delayed onset at sample 2000, `sin((i-2000) × 0.08) × exp(-(i-2000) / 3000) × 1.5`
- **Surface wave**: Delayed onset at sample 4000, compound sinusoid with slow decay
- **Noise floor**: `(random - 0.5) × 0.05` added throughout

Magnitude scales the overall amplitude envelope. Depth scales the decay time constants.

### MiniSEED Binary Parser

Pure JavaScript binary parser — no ObsPy, no external libraries. Handles the full SEED v2.4 fixed header format:

- **Header validation**: First 6 bytes must be ASCII digits/spaces (sequence number check)
- **Record structure**: Variable-length records (2^n bytes, typically 4096), multi-record files parsed sequentially
- **Zero-sample record skipping**: EarthScope's AWS cloud migration introduced empty records that were previously stripped — the parser now explicitly skips `numSamples === 0` records
- **Encoding support**:
  - **Steim1 (encoding 10)**: 4×1-byte, 2×2-byte, or 1×4-byte differences per 32-bit word, 16 words per 64-byte frame, nibble-controlled
  - **Steim2 (encoding 11)**: Extended compression with 5×6-bit, 6×5-bit, 7×4-bit, 2×15-bit, and 1×30-bit difference packing via double-nibble control
  - **Int16 (encoding 1)**: Uncompressed 16-bit signed integers
  - **Int32 (encoding 3)**: Uncompressed 32-bit signed integers
  - **Float32 (encoding 4)**: 32-bit IEEE 754 floats

### Normalization

All decoded samples are normalized to Float32Array in the ±1.0 range: each sample divided by the absolute maximum value. If the maximum is zero (flat-line), the parser falls back to synthetic generation.

### Waveform Analysis

Three continuous analysis functions run during playback, feeding the modulation system:

- **Amplitude**: Raw absolute sample value at the current playhead position
- **Derivative**: Windowed first-difference (window = 5 samples), measuring rate of change / signal energy
- **RMS**: Root mean square over a ±50 sample window, measuring sustained energy
- **Zero-crossing detection**: Pre-computed array of all sign-change positions in the waveform, used to drive RhythmCloud trigger timing

---

## Layer 2 — Granular Synthesis Engine

The core audio engine uses three independent granular clouds, each a stream of micro-sound grains (short oscillator bursts) whose parameters are continuously modulated by the seismic data. All three clouds feed into a shared master filter.

### Grain Architecture

Each individual grain follows the signal path:

```
OscillatorNode → BiquadFilterNode (lowpass) → GainNode (envelope) → StereoPannerNode → destination
```

- Oscillator: frequency, waveform type, detune (±25 cents random)
- Filter: lowpass with configurable frequency and Q
- Envelope: attack/sustain/release ramp via `linearRampToValueAtTime`
- Panner: StereoPannerNode with fallback GainNode for older browsers

### TextureCloud

The ambient shimmer layer — high density, short grains, wide stereo field.

| Parameter | Base Value | Seismic Modulation |
|-----------|-----------|-------------------|
| Waveform | sine | — |
| Density | 45 grains/s | 35 + amplitude × 60 |
| Grain duration | 35ms | ±50% spread |
| Base frequency | root × 4 | +(sample × 0.25) |
| Frequency spread | 1.2 octaves | +derivative × 15 |
| Pan spread | ±1.0 (full stereo) | +derivative × 8 |
| Amplitude | 0.08 | 0.06 + amplitude × 0.08 |
| Filter | 6kHz LP, Q=1 | — |

### ToneCloud

The harmonic bed layer — low density, long grains, narrow stereo, centered.

| Parameter | Base Value | Seismic Modulation |
|-----------|-----------|-------------------|
| Waveform | triangle | — |
| Density | 6 grains/s | 5 + RMS × 10 |
| Grain duration | 300ms | ±30% spread |
| Base frequency | root | — |
| Frequency spread | 0.4 octaves | +derivative × 8 |
| Pan spread | ±0.35 (center-biased) | — |
| Amplitude | 0.14 | 0.1 + RMS × 0.08 |
| Filter | 3kHz LP, Q=2 | — |

### RhythmCloud

The percussive layer — driven by zero-crossing triggers, probabilistic firing.

| Parameter | Base Value | Seismic Modulation |
|-----------|-----------|-------------------|
| Waveform | sawtooth | — |
| Density | 14 grains/s | — |
| Grain duration | 55ms | — |
| Base frequency | root × 2 | +(RMS × 0.4) |
| Trigger probability | 0.55 | 0.35 + RMS × 0.5 |
| Pan spread | ±0.7 | — |
| Amplitude | 0.10 | 0.08 + amplitude × 0.08 |
| Filter | 4kHz LP, Q=3 | — |

### Scale Quantization

All grain frequencies are quantized to the nearest pitch in the selected musical scale, calculated across octaves -3 to +4 from the root frequency (range 25–8000 Hz):

- **Pentatonic**: [0, 2, 4, 7, 9] semitones
- **Minor**: [0, 2, 3, 5, 7, 8, 10]
- **Dorian**: [0, 2, 3, 5, 7, 9, 10]
- **Harmonic**: [0, 12, 19, 24, 28, 31, 34, 36] (harmonic series)
- **Fifths**: [0, 7, 14, 21, 28] (stacked perfect fifths)

Root frequency options: A1 (55Hz), D2 (73.42Hz), E2 (82.41Hz), A2 (110Hz, default), D3 (146.83Hz).

### MUZAK Mode

Freezes the playhead at its current position and reconfigures the clouds for ambient sustained output:

- TextureCloud: density → 90, grain duration → 150ms, frequency spread → 0.15
- ToneCloud: density → 14, grain duration → 500ms
- Playhead: frozen (no advancement through seismic data)
- Visualization: note particle emitters activated on the ShadowZone canvas

---

## Layer 3 — Effects Chain & Output

### Audio Signal Graph

```
[TextureCloud] ─┐
[ToneCloud]    ─┤─→ Master BiquadFilter (LP, 2–8kHz seismic sweep)
[RhythmCloud]  ─┘         │
                           ▼
                  DynamicsCompressor (threshold: −12dB, ratio: 3:1)
                           │
                    ┌──────┼──────┐
                    ▼      ▼      ▼
              Convolver  (dry)  Delay
              (reverb)         (375ms, 38% FB)
              3.5s IR          LP filter 2.2kHz
                    │      │      │
                    └──────┼──────┘
                           ▼
                      Master Gain
                      (0 → 0.85 ramp)
                           │
                           ▼
                  AudioContext.destination
```

### Master Filter Modulation

The shared BiquadFilter sweeps its cutoff frequency continuously based on seismic amplitude:

```
targetFilter = 2000 + amplitude × 6000    (clamped 1000–10000 Hz)
```

Applied via `setTargetAtTime` with a 25ms time constant for smooth transitions.

### Convolution Reverb

Algorithmically generated impulse response — no external IR file:

- Length: 3.5 seconds at 44.1kHz (154,350 samples)
- Stereo: independent L/R channels
- Decay: `(1 - i/length)^2.3` power curve
- Early reflections: `sin(i × 0.1) × 0.25` for first 2000 samples
- Wet level: 0.45 default, depth-scaled: `0.35 + (depth/600) × 0.35`

### Delay Line

Filtered feedback delay with depth-responsive parameters:

- Delay time: 375ms default, depth-scaled: `0.3 + (depth/600) × 0.35` seconds
- Feedback: 38% default, depth-scaled: `0.35 + (depth/600) × 0.25`
- Feedback filter: 2.2kHz lowpass (darkens each repeat)
- Wet level: 0.28

### Dynamics Compressor

Prevents clipping during high-amplitude seismic events:

- Threshold: −12dB
- Ratio: 3:1
- Attack: 5ms
- Release: 200ms

---

## Visualization — ShadowZone Canvas

A real-time Canvas 2D rendering of Earth's cross-section showing P-wave propagation, shadow zones, and seismic activity. Runs independently of audio at 60fps via `requestAnimationFrame`.

### Earth Model

- **Mantle**: Full radius R, radial gradient fill
- **Outer Core**: R × 0.545 (core/mantle boundary at 54.5% of Earth's radius)
- **Inner Core**: R × 0.19
- **Shadow zones**: 103°–142° angular distance from epicenter (both sides), rendered as semi-transparent wedges with red arc strokes

### P-Wave Ray Tracing

20 rays traced from the epicenter through the Earth model:

- Rays that miss the core: straight-line to opposite surface
- Rays that hit the core: refracted path (entry → core boundary → exit → surface), with deflection angle calculated from entry/exit angles
- Shadow zone rays: reduced opacity, indicating the diffraction gap
- Wavefront animation: three phase-offset dots travel along each ray path continuously

### Dynamic Elements

- **Epicenter**: Pulsing radial gradient + 🦀 emoji, size scaled by seismic amplitude
- **Station marker**: Green triangle with antenna line, concentric pulse rings during activity, dynamically positioned and labeled with the active station name
- **MUZAK mode**: Musical note particles (♪ ♫ 🎵 🎶) emitted from epicenter and station positions, floating upward with wobble animation and decay

### Theme System

Dual-palette system toggled by double-tap on the globe or footer button:

- **DEEP** (dark): Deep blue-black background, cool mantle tones, warm core, neon accents
- **SURFACE** (light): Warm amber/gold geology illustration palette inspired by textbook cross-section diagrams

---

## User Controls

| Control | Range | Default | Effect |
|---------|-------|---------|--------|
| Scale | Pentatonic, Minor, Dorian, Harmonic, Fifths | Pentatonic | Pitch quantization grid for all three clouds |
| Root | A1–D3 (55–146.83 Hz) | A2 (110 Hz) | Base frequency for all clouds |
| Stretch | 0.25×–3.0× | 1.0× | Playhead speed through seismic data |
| Reverb | 0–100% | 45% | Convolution reverb wet level |
| Layer toggles | Texture / Tone / Rhythm on/off | All on | Individual cloud enable/disable |
| MUZAK | Toggle | Off | Freeze playhead, ambient sustain mode |
| Theme | DEEP / SURFACE | DEEP | Dark/light visual palette |

---

## Technical Specifications

- **Runtime**: Pure browser — vanilla HTML/CSS/JavaScript, zero dependencies
- **Audio**: Web Audio API (AudioContext at 44.1kHz)
- **Rendering**: Canvas 2D (DPR-aware, max 2×)
- **Data format**: Raw binary MiniSEED with Steim1/2 decompression
- **Browser compatibility**: Safari 14+, Chrome 80+, Firefox 78+, Edge 80+
- **Mobile**: iOS/Android responsive, touch-optimized, safe-area-inset aware
- **Fallbacks**: `webkitAudioContext`, `StereoPannerNode → GainNode`, `aspect-ratio → padding-bottom`, `oninput → onchange`

---

## Data Flow Summary

```
[USGS Earthquake API] → GeoJSON event (mag, depth, coordinates, time)
         │
         ├─→ [Synthetic Waveform Generator] → Float32Array → [Granular Engine] → SOUND
         │                                                          ↑
         └─→ [EarthScope FDSN] → MiniSEED binary                   │
                    │                                                │
                    └─→ [Station Fallback: ANMO→COLA→GUMO→HNR→MAJO→MIDW]
                              │
                              └─→ [MiniSEED Parser + Steim Decoder]
                                         │
                                         └─→ [Normalize Float32] ──→ HOT-SWAP (no gap)
```

---

*SEISCLAW — SOS · Sounds Of Seismic*
*Copyright © 2026 SHOOK · MIT License*
*Sweet Love For Planet Earth*
