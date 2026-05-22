# SEISMIC MICROTONAL MUSIFICATION MANUAL

**Microtonal tuning for the SeisClaw / SOS musification engine.**
Moving the Earth's voice off the 12-TET grid — while preserving geological fidelity.

> Part of the **SE1STR0N1CA** suite. Single-file HTML, Web Audio, **no dependencies, ever.**
> Standard SOS palette only: `#ff6f00` / `#ffaa00` / `#ff8f33` on `#101015`.
> Surgical edits only. Do not alter compositional structure unless instructed.

---

## Philosophical Foundation — Naivety is King

We approach this work with deliberate naivety. Traditional musical theory is largely set aside.
The guiding principle is simple: **the less you know, the cleaner it sounds.** By staying
musically innocent, we allow the seismic data to express itself more directly — free from the
weight of scales, chords, and historical expectations. Naivety is not a limitation; it is the
method.

This governs *intent*, not the toolkit. The tuning systems below — 19-TET, 24-TET, 31-TET, just
intonation, maqam — are not here to make the Earth "sound correct." They are pure intervals the
data may *lean toward*, never rules imposed upon it. We do not compose. We let the Earth choose,
and the listener completes the meaning. Every future addition to SOS is measured against this:
does it keep the experience innocent and let the data speak — or does it start composing?

The purest form of this principle is `tuningStrength = 0`: raw seismic voice, no grid. Everything
that follows is a dial away from that purity, used sparingly and in service to it.

> *Naivety is King. The less I know, the cleaner it sounds. Let the Earth sing without the burden
> of musical knowledge. Knowledge contaminates. Innocence reveals.*

---

## 0. The Unit: Cents (¢)

All tuning in this manual uses **cents** as the single primitive. Understand the unit first —
nothing else parses without it.

A **cent (¢)** measures the size of a musical interval (the distance between two pitches):

```
1 octave          = 1200 cents
1 12-TET semitone =  100 cents
```

The name is borrowed from currency (100 cents to a dollar → 100 cents to a semitone),
formalized by Alexander Ellis in the 1880s so non-Western tunings could be compared against
Western intervals on one common scale — exactly what we do here with seismic data, maqams,
and JI side by side.

**Cents are logarithmic** — equal cent-distances sound like equal pitch-steps anywhere in the
range. Conversion (core to the engine):

```js
cents = 1200 * Math.log2(ratio)
ratio = 2 ** (cents / 1200)
```

**Worked example — the perfect fifth.** A pure fifth is the ratio 3:2:

```
1200 * log2(3/2) = 701.96¢
```

So a pure fifth sits at 701.96¢; the 12-TET fifth sits at exactly 700¢. The difference,
`700 − 701.96 ≈ −1.96¢`, is why we say the 12-TET fifth is "~2¢ flat."

**Why this matters for SeisClaw.** Every system — 19-TET, 24-TET, 31-TET, JI, maqams — collapses
into a single list of cent values. One data structure, one engine, handles all of them. Adding
a new scale is just adding numbers, never new code. That is the whole KISS payoff.

| Interval | Cents (¢) |
|---|---|
| 31-TET step | 38.71 |
| Quarter-tone (24-TET step) | 50 |
| 19-TET step | 63.16 |
| 12-TET semitone | 100 |
| Pure major third (5:4) | 386.31 |
| Pure perfect fifth (3:2) | 701.96 |
| Octave | 1200 |

---

## 1. Why Leave 12-TET

12-TET is convenient but compromises the pure intervals:

| Interval | Pure Ratio | 12-TET Error |
|---|---|---|
| Perfect Fifth | 3:2 (701.96¢) | ~2¢ flat |
| Major Third | 5:4 (386.31¢) | ~13.7¢ sharp (beaty) |

Seismic data is **continuous and inharmonic** — it never sat on any grid. A finer or unequal
grid honors the raw signal better than rigid 100¢ steps.

---

## 2. Core Systems (all = cents[] arrays)

```js
const SCALES = {
  '12tet': Array.from({length:12}, (_,i) => i * 100),
  '19tet': Array.from({length:19}, (_,i) => i * (1200/19)),  // ~63.16¢ — warm / exotic
  '24tet': Array.from({length:24}, (_,i) => i * 50),         // ~50¢ — maqam / neutral
  '31tet': Array.from({length:31}, (_,i) => i * (1200/31)),  // ~38.71¢ — harmonic richness
  'ji5':   [0, 203.91, 386.31, 498.04, 701.96, 884.36, 1088.27],
  'rast':  [0, 200, 350, 500, 700, 900, 1050]                // Maqam Rast approx
};
```

- **19-TET** — Warm meantone, better major thirds, alien-yet-consonant. Good intermediate-depth color.
- **24-TET** — Bridge to Arabic / Turkish maqams; neutral intervals fit shallow, chaotic events.
- **31-TET** — **Harmonic specialist.** Best 5-limit and 7-limit approximation of any practical
  EDO here. Owns the deep / sustained branch (see below).
- **JI5** — Beatless purity for static drones and the Freeze chord.
- **Rast** — Neutral third (~350¢) for melodic / tremor ambiguity.

### Why 31-TET earns the deep branch

31-TET is the formalization of quarter-comma meantone (Huygens, 17th century), revived in the
20th for extended harmony. Its strengths are specifically *harmonic*:

- Major third (5:4): **+0.78¢** error → near-perfect ringing consonance.
- Harmonic seventh (7:4): **−1.13¢** error → the deep 7-limit color 19/24-TET cannot reach.
- Fine ~38.71¢ steps resolve spectral centroids and waveform partials with nuance.

That makes it ideal for **deep teleseismic events and sustained surface waves**, where rich,
living harmony mirrors Earth's slow, complex vibrations. It is *not* the right tool for shallow
violent events — there 24-TET's neutral ambiguity fits the chaos better. So 31-TET is promoted
to a **role**, not crowned a global default: the Earth still chooses by event type.

> ⚠️ **Correction to a common GROK/AI claim.** It is often stated that "stacking 19 steps gives
> `2^(19/12) ≈ 2.996`, near a pure 3:1." That formula stacks 19 *12-TET semitones*, not 19 steps
> of 19-TET, and conflates the two systems. 19 steps of 19-TET equal a pure octave (`2^(19/19) = 2.0`)
> by definition. The real benefit of 19-TET is a **better major third** (~6.3¢ sharp vs 12-TET's
> ~13.7¢) and a warm meantone fifth. This box stays in the manual so the error does not creep back in.

---

## 3. Core Engine (~40 lines, drop-in)

```js
function centsToMultiplier(cents) {
  return Math.pow(2, cents / 1200);
}

function snapToScale(freq, rootFreq, scale, strength = 0.5) {
  const cents = 1200 * Math.log2(freq / rootFreq);
  const oct   = Math.floor(cents / 1200);
  const local = ((cents % 1200) + 1200) % 1200;

  let nearest = scale[0], best = Infinity;
  for (const deg of scale) {
    const d = Math.abs(deg - local);
    if (d < best) { best = d; nearest = deg; }
  }

  const snapped = local + (nearest - local) * strength;
  return rootFreq * centsToMultiplier(oct * 1200 + snapped);
}
```

**Tuning Gravity** — `strength 0` = full raw seismic drift, `strength 1` = hard snap to scale.
Everything between is where SeisClaw lives. The general sweet spot is **0.3–0.8**; in 31-TET you
can pull harder (**0.6–0.75**) before it sounds quantized, because the steps are finer. Gravity,
not quantization, is the point — hard-snapping every grain throws away the inharmonic drift that
makes seismic material interesting.

---

## 4. Let the Earth Choose (SeisConduct integration)

Tuning is an *output of the data*, not a user preset — that is the SOS thesis. A clean depth
ladder routes each event; the `|| fallbacks` keep a missing field from cascading into NaN.

```js
function selectTuning(event) {
  const rootFreq = clamp(event.spectralCentroid || 60, 35, 95); // low rumble register

  let scale, strength;
  if (event.depth > 250) {            // deep / sustained / long-period
    scale = SCALES['31tet']; strength = 0.68;   // harmonic richness, 7-limit depth
  } else if (event.depth > 70) {      // intermediate
    scale = SCALES['19tet']; strength = 0.55;   // warm meantone
  } else {                            // shallow / energetic
    scale = SCALES['24tet']; strength = 0.35;   // neutral / maqam spread
  }

  const spreadCents = 32 + (event.magnitude || 0) * 28;
  return { rootFreq, scale, strength, spreadCents };
}
```

> Note: routing is by **depth only**. An earlier draft added `|| magnitude < 4.8` to the deep
> branch — but since most quakes are below M4.8 (and a missing magnitude reads as 0), that clause
> routed nearly every event to 31-TET and collapsed the depth logic. Removed deliberately.

| Seismic feature | Drives | Result |
|---|---|---|
| Spectral centroid | Root frequency | Earth sets the key |
| Focal depth | Tuning system + gravity | Deep → 31-TET; mid → 19-TET; shallow → 24-TET |
| Magnitude | Spread width + register | Bigger quake → wider, higher |
| Zero-crossings | Scale-degree stepping | Raga-style melodic motion |

---

## 5. Apply to the Three Clouds

- **Texture Cloud** — spread grains across scale degrees + small random detune (±4–10¢) for
  organic beating shimmer that evolves with seismic coda. In 31-TET the fine steps give complex,
  living texture.
- **Tone Cloud** — high `strength` toward JI/31-TET for ringing, beatless sustains with 7-limit
  depth during long-period surface waves.
- **Rhythm Cloud** — zero-crossings step *through* scale degrees (raga-like motion); 31-TET adds
  unexpected tonal color.
- **Freeze / Muzak** — lock all grains to a pure JI / 31-TET-derived chord from the event's
  spectrum. Maximum consonance snapshot of the Earth at one instant.

---

## 6. Implementation Order (strict KISS)

1. Add `SCALES`, `centsToMultiplier`, `snapToScale`. **Test alone.**
2. Add a single `tuningStrength` UI control. Audition 0 → 1 on a live event.
3. Insert `selectTuning()` into the Reason stage. Confirm depth actually moves the sound.
4. Update the three clouds one-by-one (surgical).
5. Optional: cent-deviation readout in the ShadowZone visualizer (orange palette only).

---

## 7. Glossary

| Term | Meaning |
|---|---|
| **Cent (¢)** | Logarithmic pitch unit; 100¢ = one semitone, 1200¢ = one octave |
| **12-TET / EDO / TET** | Equal divisions of the octave; 12-TET is the standard 12 × 100¢ grid |
| **31-TET** | 31 equal divisions; best practical EDO here for 5-limit and 7-limit harmony |
| **Just Intonation (JI)** | Tuning by exact integer ratios (3:2, 5:4); beatless |
| **Maqam** | Arabic/Turkish melodic mode, often with neutral (quarter-tone) intervals |
| **Neutral interval** | Between major and minor (e.g. neutral third ≈ 350¢) |
| **Tuning gravity** | This engine's term: strength of attraction toward a scale degree |
| **Spectral centroid** | Spectral center of mass; used here as the dynamic root pitch |

---

## Standing Rules (do not violate)

- Single-file HTML + Web Audio only. **No external dependencies** (no numpy, no ObsPy, no libraries).
- Pure MiniSEED binary parsing only.
- EarthScope endpoint: `service.earthscope.org`.
- SOS palette only: `#ff6f00` / `#ffaa00` / `#ff8f33` on `#101015`. Never invent per-module schemes.
- Surgical edits. **DO NOT CHANGE compositional structure** unless explicitly instructed.

---

## Credits

- Concept & practice: **SHOOK / D.V. Rogers** — SeisClaw / SOS (Sounds of Seismic).
- Microtonal source research generated with GROK; corrected, expanded, and adapted to the
  SeisClaw architecture here. 31-TET deep-branch role integrated and routing bug fixed.
- Cent unit: formalized by Alexander Ellis, 1880s. Scale archives: Scala, Xenharmonic Wiki.

*Earth composes. Code translates. You complete it.*