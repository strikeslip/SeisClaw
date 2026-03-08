# SEISCLAW / SOS — Autonomous AI Research Build Plan
### Using Claude Sonnet + autoresearch-macos on Mac M4 Mini 24GB
**(SHOOK) — sos.allshookup.org / seisclaw.com <br>
**REF: https://github.com/miolini/autoresearch-macos <br>
*Generated March 8, 2026*

---

## 1. The Concept: What autoresearch-macos Is

The `autoresearch-macos` repo (miolini's Apple Silicon port of Karpathy's `autoresearch`) implements a single elegant loop:

> Give an AI agent one file to modify, a fixed evaluation metric, and a time budget. Let it run overnight. Wake up to a log of experiments and a better system.

The human's only job is to write `program.md` — a Markdown research charter that encodes intent, aesthetic direction, and constraints. The agent reads it, modifies the target file, runs an experiment, evaluates the result, keeps or discards the change, and repeats. Approximately 100 experiments overnight on a single machine.

**The key insight for SEISCLAW:** this architecture was designed for LLM training research but maps directly onto autonomous sonic research. Instead of minimising `val_bpb` (validation bits per byte), SEISCLAW agents evolve the translation layer between Earth's seismic activity and electronica music — guided by a human-authored aesthetic constitution.

---

## 2. Working Proof of Concept

Two live deployments already exist and prove the core pipeline works:

- **https://seisclaw.com/** — SEISCLAW frontend (autonomous electronica music agents interpreting live seismic data)
- **https://sos.allshookup.org/flow.html** — SOS (Sounds of Seismic) live seismic sonification system

These are the baseline. The autoresearch loop exists to autonomously improve and evolve what is already running.

---

## 3. Hardware Platform

**Mac M4 Mini, 24GB Unified Memory**

| Capability | Relevance to SEISCLAW |
|---|---|
| Apple Neural Engine (ANE) | Accelerates any ML-based spectral analysis |
| MPS backend (Metal) | PyTorch audio DSP without CUDA dependency |
| 24GB unified memory | Full experiment history in RAM, no swap pressure |
| Efficiency + performance cores | Seismic data ingestion on efficiency cores, agent loop on performance cores |
| macOS CoreAudio | Low-latency audio output via `sounddevice` |

The M4 Mini is the correct machine for this. 24GB unified means the seismic stream buffer, audio render pipeline, and Claude API context window all coexist in RAM simultaneously across a full overnight run (~48 experiments).

---

## 4. Critical Technical Decision: No ObsPy

**ObsPy is not used. It is not needed.**

The proven approach — already running in SOS and SEISCLAW — is:

```
Raw MiniSEED binary → direct binary decode → waveform array → synthesis engine
```

Pure MiniSEED binary parsing is leaner, faster, has zero C dependency issues on macOS, and the SOS pipeline already demonstrates it works at production quality. Any documentation, code, or dependency list referencing ObsPy for SEISCLAW/SOS development is incorrect.

---

## 5. Three-File Architecture (Adapted from autoresearch)

| Original File | SEISCLAW Equivalent | Who Edits It | Purpose |
|---|---|---|---|
| `prepare.py` | `seismic_utils.py` | Never (fixed) | USGS/EarthScope feed, MiniSEED binary decode, audio output routing, evaluation metric |
| `train.py` | `synthesize.py` | Claude agent | Granular engine, FM ratios, wavetable scanner, additive partials, subtractive filter chain |
| `program.md` | `seisclaw_program.md` | D.V. Rogers | Aesthetic intent, research direction, what "better" means, constraint boundaries |

The agent only ever touches `synthesize.py`. Everything else is fixed infrastructure. This keeps diffs reviewable and rollback trivial.

---

## 6. The Five-Layer Synthesis Architecture

Each synthesis technique maps to a distinct aspect of seismic data and occupies a defined aesthetic role:

### Granular Synthesis
- **Seismic source:** Raw MiniSEED waveform samples, sliced directly into grains
- **Parameters driven by:** Amplitude envelope, frequency content of live stream
- **Aesthetic role:** Continuous geological texture — the backbone of SEISCLAW's sound
- **Reference:** Curtis Roads (*Microsound*), Aphex Twin ambient works

### FM Synthesis
- **Seismic source:** P-wave vs S-wave frequency relationships → carrier:modulator ratios
- **Parameters driven by:** Wave type detection, event magnitude, depth
- **Aesthetic role:** Timbral character of geological events — the harmonic violence of seismic physics
- **Reference:** John Chowning (inventor of FM synthesis), Richard Devine, Autechre

### Wavetable Synthesis
- **Seismic source:** A window of MiniSEED samples loaded directly as a single-cycle waveform
- **Parameters driven by:** Waveform morphing as the earthquake evolves in real time
- **Aesthetic role:** Earth's own waveform as the oscillator — conceptually the purest SEISCLAW technique
- **Reference:** PPG Wave, Wendy Carlos, Caterina Barbieri

### Additive Synthesis
- **Seismic source:** FFT of seismic stream → partial amplitudes → resynthesis
- **Parameters driven by:** Spectral content of long-period surface waves, teleseismic events
- **Aesthetic role:** Harmonic skeleton — slow, glacial, evolving tonal structure beneath the texture
- **Reference:** Max Mathews, Suzanne Ciani, Brian Eno, Steve Reich

### Subtractive Synthesis
- **Seismic source:** Broadband seismic noise as source, filtered by spectral shape of each event
- **Parameters driven by:** P-wave arrival opens bandpass; S-wave arrival shifts resonant frequency
- **Aesthetic role:** Rhythmic punch and transient attack — the most analogue-adjacent layer
- **Reference:** Tangerine Dream, Kraftwerk, Legowelt, Severed Heads

### Synthesis Layer Summary

| Layer | Technique | Seismic Input | Role |
|---|---|---|---|
| 1 | Granular | Raw waveform samples | Geological texture |
| 2 | FM | P/S wave ratios | Event character |
| 3 | Wavetable | Waveform window | Earth as oscillator |
| 4 | Additive | FFT partials | Deep tonal structure |
| 5 | Subtractive | Broadband filtered noise | Rhythm and attack |

---

## 7. The Aesthetic Constitution — Inspirational Artists

The following artists form SEISCLAW's aesthetic DNA. They are not decoration — they encode what the musicality metric must chase.

**Pioneers of synthesis as science**
Max Mathews, John Chowning, Wendy Carlos, Suzanne Ciani, Curtis Roads
→ These people invented the techniques SEISCLAW uses. Ancestral methodology, not just inspiration.

**Systems and process music**
Steve Reich, Terry Riley, Robert Fripp, Brian Eno
→ Music as system, not composition. Patterns that evolve through their own internal logic. Earth as the process generator. SEISCLAW is a Reich phase piece where the phase relationships are tectonic.

**Kosmische / Berlin School**
Tangerine Dream, Jean Michel Jarre, Giorgio Moroder, Kraftwerk, Michael Garrison, Kosmische Wellen
→ Long-form, evolving, synthesizer-as-landscape. Geological time rendered as sound. Slow morphs, vast reverb spaces, sequencer patterns that breathe.

**Maximalist texture and complexity**
Autechre, Richard Devine, Aphex Twin, Caterina Barbieri, Skee Mask
→ Dense, rhythmically complex, engineered timbres. Where FM and granular layers earn their complexity budget.

**Emotional weight through electronics**
Boards of Canada, Oneohtrix Point Never, Igloohost, Ben Prunty, Disasterpeace, John Carpenter
→ Melody exists but feels half-remembered, slightly wrong, melancholic. The human signal inside geological noise.

**Noise and intensity as catharsis**
Fuck Buttons, Health, The Knife, Fever Ray, Clark, Sophie
→ Electronics that physically impact. When a major seismic event hits, SEISCLAW should be capable of going there.

**Noise, abstraction, and structured chaos**
CAN, Dick Hyman, Dose One, Alias, Mall Grab, Igloohost, Legowelt, SOPHIE
→ Genre-resistant, system-driven, often raw. Permission to be wrong in interesting ways.

**The Xenakis Alignment**
Iannis Xenakis — stochastic music, mass sound movements, architecture and music unified — is the closest intellectual predecessor to SEISCLAW. His concept of stochastic music, where statistical distributions of sound events replace traditional melody and harmony, is exactly what seismic sonification does natively. The USGS data *is* that structure. The synthesis layers reveal it.

---

## 8. The Musicality Metric — The Most Important Creative Decision

The autoresearch loop requires a computable evaluation score. For SEISCLAW this is not trivial — seismic data has no musical intent, so the metric measures how well the translation algorithm serves the aesthetic vision.

### The Five Qualities (derived from the artist list)

Every artist in the SEISCLAW constitution shares these five qualities:

| Quality | What It Means | Measurable Proxy |
|---|---|---|
| Time operates differently | Events unfold on their own clock, not 4/4 | Aperiodicity score — deviation from regular pulse |
| Texture IS composition | Timbre and density carry structural weight | Simultaneous synthesis layer activity — layer density index |
| System has agency | Music feels like it is doing something, not being played | Rate of parameter change — evolution gradient |
| Tension and release are geological | Long buildups, sudden ruptures, slow decay | Amplitude envelope variance over 60-second windows |
| Human emotion filtered through process | You feel something but cannot explain why | Harmonic interval consonance/dissonance ratio |

### Composite SEISCLAW Score

```
seismic_musicality_score =
  (aperiodicity_score      × 0.20) +
  (layer_density_index     × 0.20) +
  (evolution_gradient      × 0.20) +
  (envelope_variance       × 0.20) +
  (consonance_ratio        × 0.20)
```

Range: 0.0 – 1.0. Higher is better. This is the SEISCLAW equivalent of `val_bpb`.

### The Human-in-the-Loop Override

For experiments where the composite score is inconclusive, a morning listening session provides a 1–10 human rating that feeds back as an additional signal. This is slower but more honest for aesthetic judgements that resist full quantification.

---

## 9. `seisclaw_program.md` — The Research Charter

This is the document D.V. Rogers writes and iterates. The agent reads it before every experiment. It is the aesthetic and technical constitution of the research org.

Recommended structure:

```markdown
# SEISCLAW Research Program

## Context
You are an autonomous electronica composition agent. Earth is the composer.
Your role is to improve synthesize.py so that live seismic data produces
richer, more musical electronica aligned with the aesthetic constitution below.
You have a 10-minute experiment budget per run.

## The Score
- seismic_musicality_score (higher is better, range 0–1)
- Current baseline: [auto-filled each run]
- Best achieved: [auto-filled]

## What You May Change in synthesize.py
- Granular synthesis parameters (grain size, density, pitch scatter, overlap)
- FM synthesis carrier/modulator ratios and index scaling
- Wavetable window size and morphing speed
- Additive partial count and amplitude scaling
- Subtractive filter cutoff mapping and resonance
- Cross-synthesis routing (which seismic channels feed which synthesis layers)
- Effect chain parameters (reverb time, delay feedback, distortion character)
- Frequency scaling functions (how seismic Hz maps to audio Hz)

## What You Must Never Change
- The USGS/EarthScope data fetch pipeline
- The MiniSEED binary parser
- Audio output routing
- The evaluation function in seismic_utils.py

## Aesthetic Intent
Target: driving, hypnotic electronica in the tradition of Autechre, Aphex Twin
ambient works, Klaus Schulze, Tangerine Dream, and Xenakis stochastic works.
The seismic data must feel like it is composing, not triggering.
Prioritise musical tension and release over constant density.
Allow silence. Allow rupture. Avoid drone-lock and wall-of-noise.

## Experiment Log
[Agent appends results here automatically — date, score delta, change summary]
```

---

## 10. The Agent Loop — Technical Implementation

```python
# agent_loop.py — core autonomous research loop
import anthropic
import subprocess
import shutil
from pathlib import Path

client = anthropic.Anthropic()  # ANTHROPIC_API_KEY from environment

EXPERIMENT_DURATION = 600       # 10 minutes per experiment
MODEL = "claude-sonnet-4-20250514"

def run_experiment() -> float:
    """Run synthesize.py in eval mode, return musicality score."""
    result = subprocess.run(
        ["python", "synthesize.py", "--eval-mode", f"--duration={EXPERIMENT_DURATION}"],
        capture_output=True, timeout=EXPERIMENT_DURATION + 80
    )
    # synthesize.py prints SCORE:0.742 on final line
    score_line = result.stdout.decode().strip().split("\n")[-1]
    return float(score_line.split("SCORE:")[-1])

def agent_propose(program_md: str, current_code: str, history: list) -> str:
    """Ask Claude Sonnet to propose the next modification."""
    response = client.messages.create(
        model=MODEL,
        max_tokens=4096,
        system=program_md,
        messages=history + [{
            "role": "user",
            "content": (
                f"Current synthesize.py:\n\n```python\n{current_code}\n```\n\n"
                "Propose your next modification to improve seismic_musicality_score. "
                "Return ONLY the complete updated Python file, no commentary."
            )
        }]
    )
    return response.content[0].text

def research_loop(n_experiments: int = 48):
    program_md = Path("seisclaw_program.md").read_text()
    current_code = Path("synthesize.py").read_text()
    best_score = run_experiment()
    best_code = current_code
    history = []

    print(f"Baseline score: {best_score:.4f}")

    for i in range(n_experiments):
        print(f"\n--- Experiment {i+1}/{n_experiments} ---")
        proposed_code = agent_propose(program_md, current_code, history)
        Path("synthesize.py").write_text(proposed_code)

        try:
            score = run_experiment()
        except Exception as e:
            print(f"Experiment failed: {e} — rolling back")
            Path("synthesize.py").write_text(current_code)
            continue

        delta = score - best_score
        print(f"Score: {score:.4f} (Δ {delta:+.4f})")

        if score > best_score:
            best_score = score
            best_code = proposed_code
            shutil.copy("synthesize.py", "synthesize_best.py")
            print("✓ Improvement kept")
        else:
            Path("synthesize.py").write_text(current_code)
            print("✗ Rolled back")

        history.append({
            "role": "assistant",
            "content": f"Experiment result: score={score:.4f}, delta={delta:+.4f}"
        })

    print(f"\n=== Session complete. Best score: {best_score:.4f} ===")

if __name__ == "__main__":
    research_loop()
```

---

## 11. Overnight Run Profile — Mac M4 Mini 24GB

| Parameter | Value |
|---|---|
| Experiment time budget | 10 minutes |
| Experiments per hour | ~6 |
| Overnight run (8 hours) | ~48 experiments |
| Model | claude-sonnet-4-6 (claude-sonnet-4-20250514) |
| Rollback on regression | Automatic |
| Best version saved | `synthesize_best.py` |
| Memory pressure | Minimal — 24GB unified handles full history |
| MPS acceleration | Active for spectral analysis components |

Morning deliverable: experiment log with scores, best `synthesize_best.py`, musicality score delta from baseline, and agent's own notes appended to `seisclaw_program.md`.

---

## 12. Python Stack — No ObsPy

```toml
# pyproject.toml
[project]
requires-python = ">=3.11"
dependencies = [
  "anthropic>=0.40.0",      # Claude Sonnet API
  "numpy>=2.0",              # Array operations on MiniSEED samples
  "scipy>=1.13",             # FFT, signal processing, filter design
  "sounddevice>=0.4.6",      # macOS CoreAudio output
  "pedalboard>=0.9",         # Spotify DSP library — Apple Silicon native
  "librosa>=0.10",           # Spectral analysis for musicality metric
  "requests",                # USGS/FDSN HTTP feed
]
```

MiniSEED binary parsing is handled directly in `seismic_utils.py` — the same proven approach running in SOS at `sos.allshookup.org/flow.html`. No ObsPy. No C extension dependency chain. No macOS compatibility issues.

---

## 13. Build Sequence

### Phase 1 — Infrastructure (Week 1)
- [ ] Set up `seisclaw-autoresearch/` project directory on M4 Mini
- [ ] Install `uv` package manager
- [ ] Build `seismic_utils.py` — port existing SOS MiniSEED binary parser
- [ ] Implement `seismic_musicality_score` composite metric
- [ ] Verify USGS feed → binary decode → audio output pipeline end-to-end

### Phase 2 — Baseline Synthesis (Week 2)
- [ ] Build `synthesize.py` v0.1 — all five synthesis layers active at minimal complexity
- [ ] Manual tuning pass — establish baseline musicality score
- [ ] Write `seisclaw_program.md` v0.1 — encode aesthetic constitution
- [ ] First manual agent step — single Claude Sonnet proposal, evaluate

### Phase 3 — First Autonomous Run (Week 3)
- [ ] Run `agent_loop.py` for one full overnight session (~48 experiments)
- [ ] Morning review: listen to `synthesize_best.py` output, rate 1–10
- [ ] Update `seisclaw_program.md` with findings and refined intent
- [ ] Repeat weekly

### Phase 4 — Research Org Evolution (Ongoing)
- [ ] Add second agent specialised in rhythmic extraction (P-wave onset → kick logic)
- [ ] Add third agent specialised in harmonic mapping (surface wave → additive partials)
- [ ] Cross-agent evaluation — agents score each other's proposals
- [ ] `seisclaw_program.md` becomes a living document, versioned by generation

---

## 14. The Deeper Implication

Karpathy built autoresearch for LLM training. SEISCLAW applies the same architecture to **autonomous sonic research** — which is more aligned with SEISCLAW's conceptual core. The agents are not just running the system. They are evolving the translation layer between Earth and music. Each overnight run is a generation of the SEISCLAW organism.

`seisclaw_program.md` is the equivalent of Karpathy's "research org code" — the document that determines the aesthetic direction of evolution. As D.V. Rogers refines it across weeks of overnight runs, it accumulates the compressed aesthetic intelligence of the project. The code evolves. The program.md evolves. The music evolves.

Earth remains the only composer who never changes the brief.

---

*SEISCLAW — Autonomous Electronica Music Agents interpreting Earth's seismic activity*
*D.V. Rogers / allshookup.org — in collaboration with Dr. Ryan McGee, UCSB AlloSphere Research Facility*
