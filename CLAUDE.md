# CLAUDE.md — AEON Conformal Zoom Visualiser

Build context for Claude Code. Be concise. **Read `PLAN.md` for full reasoning. Use `prompts.md` for the staged build. Build one step at a time and stop for verification.**

## Mission
A single self-contained `index.html`: a seamless infinite conformal zoom where one zoom octave walks through the life of a cosmological *aeon* (spacetime-crystal critical collapse → black hole → Hawking point → conformal rebirth), narrating the physics as it passes.

## Stack (do not change)
- One `.html` file. Raw WebGL. **No Three.js. No build step. No npm.**
- Architecture: **functional core / imperative shell, unidirectional data flow.**

## The hard invariant (this is the whole game)
**The `phase = 1` state must be visually identical to `phase = 0`.** The conformal rebirth must equal the smooth start, so the infinite zoom loops seamlessly. If the wrap is ever visible, the build is wrong.

Applies ONLY to the periodic visual group. Counters (`logDepth`, `aeonCount`) are monotonic and exempt.

## Two rules that protect the invariant structurally
1. **JS owns the only clock. The shader never integrates time for the zoom.** Shader receives `phase` as a uniform. A raw `u_time` is allowed ONLY for cosmetic non-looping detail that cannot affect the seam.
2. **The shader only ever sees `phase ∈ [0,1)`.** Visual scale is driven by fractional phase, so appearance resets every Δ by construction. `logDepth` touches the HUD counter ONLY.

> Common failure: an agent "makes it work" by sneaking `u_time` into the zoom or letting the shader hold its own clock. Don't. If the wrap drifts, that's the cause.

## Per-frame flow (unidirectional)
```
dt        → clock.advance(dt)      // the ONLY mutable state
snapshot  → derive(snapshot)       // PURE: phase → full visual state
state     → gpu.render(state)      // sink 1
state     → hud.update(state)      // sink 2 (diffed, ~10 Hz text)
```
No feature reads another feature's state — every feature reads `phase`. No event bus; top-down tick is clearer.

## The contract (stabilise first; both sinks depend on it)
```js
DerivedState = {
  logDepth, aeonCount,                 // monotonic — display only
  phase,                               // [0,1)
  criticality, volume, shear, bloom,   // periodic — MUST satisfy f(0)===f(1)
  forkSign,
  stage: { id, label, confidence, caption },
  zoomRate, zoomDir                    // dir = +1 in / -1 out (v2)
}
```

## Layers (dependency order, all in one file)
1. **CONFIG** — every magic number (Δ, center, DPR cap, palette, confidence colours, pacing). Nothing hardcoded downstream.
2. **TIMELINE** — declarative stage table (drives captions + regime label + confidence). The honesty layer lives here as DATA.
3. **derive() (pure)** — `phase → DerivedState`. No random, no DOM, no GL. Testable.
4. **Clock** — owns mutable state. `phase += rateFn(phase)·dt`; on wrap `phase -= 1; aeonCount++`. Owns play/pause/scrub.
5. **GPU sink** — context, program, uniform-adapter (`state → uniforms`), fullscreen **triangle** (not quad), draw.
6. **HUD sink** — diffed DOM, throttled text.
7. **Composition root** — thin rAF loop.

## Δ and "aeon" — keep honest
- **Δ ≈ 3.44** = Choptuik echo-scaling period (real physics; the zoom/octave length & RG eigenvalue). ONE definition, in CONFIG.
- "One octave = one aeon" = a **labelled narrative choice**, not a physical claim. Don't let captions imply an aeon literally spans one Δ.

## Honesty layer (build into the UI, not a footnote)
Each stage carries a confidence tag, colour-coded:
- `solid` — peer-reviewed / standard (stages: smooth, crystallization, knife-edge, echoing)
- `contested` — disputed evidence (Hawking points; a 2024 ML reanalysis found no significant signal)
- `interpretive` — artistic/cosmological licence (conformal crossover, seed→galaxy, Weyl visual)

## Conventions
- Delta-time everywhere; never frame count. (Seamlessness is *spatial*, so varying the *time* rate for pacing can't tear the loop.)
- `zoomDir` is one sign; v2 cosmic-web zoom-OUT is a strategy that flips it + swaps the field, reusing clock + HUD.
- Sinks are an interface `(state) → void`. Audio (deferred) is just a third sink.
- DPR-aware sizing, capped. Pause on `visibilitychange`.
- No constants outside CONFIG; no experience tuning outside TIMELINE.
- **Mobile-first.** Design for phones first, enhance up. Touch is the primary input: tap = play/pause, horizontal drag = scrub (keyboard stays as the desktop equivalent). Fluid type via `clamp()`, positions offset by `env(safe-area-inset-*)`, `touch-action:none` so the canvas owns gestures. HUD tunables/thresholds live in CONFIG.

## Guardrails (must survive into the build)
- **Startup continuity assertion**: sample every periodic field near 0 and near 1; fail loudly if `|f(ε) − f(1−ε)| > tol`.
- **`?phase=0.42` URL param**: jump to any phase (stages 5–6 only flash in normal playback).
- **`DEBUG` flag**: overlay live phase + uniform values. Keyboard scrub.

## Aesthetic direction (for the HUD/shell — step 3)
Scientific-instrument / observatory readout. Dark, atmospheric (depth, not flat black). Distinctive technical typography — **NOT** Inter/Roboto/Arial/Space Grotesk. Cohesive palette via CSS variables; the confidence colours are sharp accents on a restrained base. Refined minimalism executed precisely > maximalism here. The zoom is the spectacle; the HUD is a quiet, exact instrument around it.

## Build order (see prompts.md)
0. Lock phase model (done in PLAN) → 1. Seamless zoom on STATIC field (acceptance: invisible wrap) → 2. phase-driven morph smooth→crystal→collapse → 3. HUD + confidence captions → 4. crossover physics (Hawking bloom + Ricci→0 / Weyl shear closing the seam) → 5. optional (pacing, holographic frame *if it earns it*, audio, v2 zoom-out).

**Do not jump ahead. Step 1's invisible-wrap test is the gate for everything.**
