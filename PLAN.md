# AEON — Conformal Zoom Visualiser

**Reference / architecture doc. Written before implementation. Read before cutting code.**

A single, self-contained HTML file: a seamless infinite conformal zoom in which one zoom octave walks through the full life of a cosmological *aeon*, narrating the physics as it passes. Visualises the recent **spacetime-crystal / critical-collapse** result and stitches it into **Penrose's Conformal Cyclic Cosmology (CCC)** and several adjacent ideas — all of which are, at root, the same scale transformation seen from different angles.

- **Status:** design locked. Build not started.
- **Stack (confirmed):** single `.html` file, raw WebGL, no Three.js, no build step.
- **Architecture (confirmed):** functional core / imperative shell, unidirectional data flow.

---

## 1. Concept

### The unifying principle
Everything here is organised by **scale-invariance, conformal symmetry, self-similarity, and cyclicity**. That single thread is what lets one infinite zoom carry many distinct phenomena without becoming a grab-bag: each phenomenon is assigned to a distinct *phase* of one repeating cycle.

### The spine
A seamless infinite zoom where **one echo-period of the zoom = one narrative cycle (one aeon)**. The camera falls inward forever. Because critical collapse is *discretely self-similar*, zooming by exactly the echo period Δ returns a rescaled copy of the start — so the loop is genuinely seamless (Droste-style), and the next octave *is* the next aeon for free.

> **Honesty note on Δ:** Δ ≈ 3.44 is the **Choptuik echo-scaling period** of critical collapse (for a massless scalar field) — real physics, and the reason the zoom can be seamless. It is **not** the physical duration of a cosmological cycle. Mapping "one zoom octave = one aeon" is a deliberate, *labelled* storytelling choice, not a physical claim.

---

## 2. Physics → visual mapping (with epistemic status)

The honesty layer is **data, not a footnote**: every stage carries a confidence tag, colour-coded in the UI.

| Confidence | Meaning | UI colour (indicative) |
|---|---|---|
| `solid` | Peer-reviewed / standard physics | calm (e.g. cyan) |
| `contested` | Published but disputed evidence | amber |
| `interpretive` | Artistic/cosmological licence | violet |

### Core phenomena and where each lives
- **Spacetime crystal / critical collapse** *(solid)* — smooth spacetime freezes into a regular lattice near the black-hole threshold (the water→ice analogy). The exact-solution result derived via the large-dimension limit.
- **The knife-edge bifurcation** *(solid)* — at criticality, a sliver of energy ε tips the lattice either to dispersal (subcritical) or collapse (supercritical). The basin boundary between fates is fractal.
- **Choptuik discrete self-similarity / echoing** *(solid)* — the solution repeats at ever-smaller scales; period Δ. **This is the zoom engine**, and the stage where the zoom is named as a renormalization-group (RG) step toward the critical fixed point — Δ is the RG eigenvalue.
- **Microscopic black hole → Hawking point** *(contested read)* — a micro-BH forms and, over ~10¹⁰⁰ yr, evaporates into a single concentrated burst. In CCC this is a *Hawking point* seeding the next aeon's sky. Evidence (An/Penrose, 99.98% claimed) is **disputed** — a 2024 ML reanalysis (HawkingNet) found no significant signal after accounting for unusually bright spots. The *tension itself* is renderable.
- **Conformal crossover (CCC boundary)** *(interpretive)* — with all mass gone, scale loses meaning; the scaleless far-future is rescaled and stitched to the next Big Bang.

### Persistent / secondary layers
- **Weyl curvature** *(interpretive, but principled)* — the conformally-invariant part of curvature, i.e. exactly what survives the crossover. Rendered as **anisotropic shear** (distortion at constant area), distinct from **Ricci = isotropic volume change**. At the crossover, drive volume→0 and keep shear: what visibly carries across the boundary is the conformally-invariant piece.
- **RG flow / critical universality** — not a separate visual; it's the *interpretation* of the zoom. Named in the echo stage.
- **Holographic frame (large-D / AdS-CFT)** *(optional, candidate to cut)* — a boundary ring showing a 1D radial encoding of the 2D bulk (boundary-encodes-bulk).
- **Discrete scale invariance in nature** — log-periodicity also appears in earthquakes, fracture, turbulence, markets. Justifies a *universal* echo rhythm and gives any future audio layer a principled beat.

### Deferred to v2 (each is quietly a second project)
- **Cosmic-web / JWST zoom-*out* mode** — reverse the camera to reveal seed black holes growing into filaments and early galaxies. Whole separate visual system.
- **Audio (sonification)** — frequencies spaced by Δ. Last; off by default (autoplay rules + annoying uninvited).

---

## 3. Refined design decisions (the fixes that de-risk the build)

1. **Δ and "an aeon" are separated.** Δ = zoom/echo period (physics). "One octave = one aeon" = labelled narrative choice. The echo-stage caption states Δ's true meaning; nothing pretends an aeon spans one Δ.
2. **One master clock drives everything.** Every visual feature is a pure function of a single parameter `phase = fract(logDepth / Δ)`. As you zoom, `logDepth` rises; `phase` cycles 0→1→0 forever. Stage sequencing and loop-seamlessness both fall out automatically.
3. **Seamlessness is load-bearing, not polish.** Discrete self-similarity *is* the claim that the system looks identical under Δ-rescaling. A genuinely seamless zoom is the piece embodying the physics — promoted to a hard requirement and proven first.
4. **Variable zoom rate for pacing.** `dPhase/dt` varies — slow through crystallization/collapse, fast through smooth space. Safe, because seamlessness is a property of *spatial* self-similarity; varying the *time* rate can't tear the loop.
5. **Weyl/Ricci made concrete.** Ricci = radial volume pulse; Weyl = area-preserving shear. (See §2.)
6. **Honesty baked into captions** (see §2 table), not a separate disclaimer.

### The one hard invariant
> **The `phase = 1` state must be visually identical to the `phase = 0` state.** The conformal rebirth must equal the smooth start. This is designed in from stage 6 → stage 1, not discovered later. It applies **only to the periodic visual group**, never to the monotonic counters.

---

## 4. Architecture

### 4.1 Governing model: one clock → derive → two sinks

Unidirectional data flow with a single source of truth. Per frame:

```
dt        → clock.advance(dt)      // the ONLY mutable state
snapshot  → derive(snapshot)       // PURE: phase → full visual state
state     → gpu.render(state)      // sink 1: WebGL uniforms + draw
state     → hud.update(state)      // sink 2: diffed DOM update
```

- **Functional core / imperative shell.** The `phase → state` math is a pure, testable core; WebGL and the DOM are the dirty shell at the edges.
- **No feature reads another feature's state** — every feature reads `phase`.
- **No event bus / observer.** The flow is simple and synchronous; a top-down "tick then render both sinks" is clearer and has no ordering hazards. Pattern restraint is the point.

### 4.2 The state contract (the spine)

The single interface both renderers depend on. Stabilise this **first** — sinks can be built against the contract before all producers exist.

```js
DerivedState = {
  // monotonic — display only — NOT subject to the seam invariant
  logDepth, aeonCount,

  // periodic visual params — MUST satisfy f(0) === f(1)
  phase,        // [0,1) master phase
  criticality,  // 0 smooth → 1 collapse (lattice blend)
  volume,       // Ricci term (isotropic scale)
  shear,        // Weyl term (anisotropic distortion)
  bloom,        // Hawking-point intensity
  forkSign,     // knife-edge ±1

  // declarative, from the timeline table
  stage: { id, label, confidence, caption },
  zoomRate,
  zoomDir       // +1 in / -1 out (v2 seam)
}
```

> **The split is the important part: counters are monotonic, visuals are periodic.** The seam invariant applies only to the periodic group.

### 4.3 Layers (top-to-bottom dependency order in the single file)

1. **CONFIG** — every magic number: Δ (one definition), center point, DPR cap, pacing curve, colour palette, confidence colours. Nothing hardcoded downstream.
2. **TIMELINE (data)** — the stage table as a declarative array `[{phaseStart, phaseEnd, id, label, confidence, caption}]`. Drives captions *and* regime label. The honesty layer lives here as data. (Config-registry pattern.)
3. **Math / derivation (pure)** — `derive(phase) → DerivedState`. All periodic functions. No `Math.random`, no DOM, no GL. Unit-testable in isolation.
4. **Clock** — owns the only mutable state. Integrates `phase += rateFn(phase)·dt`; on wrap, `phase -= 1; aeonCount++`. `logDepth = (aeonCount + phase)·Δ` derived for display. Owns play/pause/scrub.
5. **GPU renderer** — owns context, program, uniform locations, fullscreen triangle, draw. A **uniform-adapter** maps `DerivedState` keys → uniform setters, so adding a uniform is a one-line change in one place.
6. **HUD renderer** — same `state`; **diffs** so text isn't rewritten every frame (throttle text to ~10 Hz).
7. **Composition root** — the `requestAnimationFrame` loop wiring the four steps. Thin.

### 4.4 Two load-bearing decisions (these prevent the seam bug *structurally*)

1. **JS clock is authoritative; the shader has no clock of its own.** The shader receives `phase` as a uniform and never integrates time for the zoom. A raw `u_time` is permitted *only* for cosmetic, non-looping detail (faint noise drift) that can't affect the seam. Two clocks = guaranteed drift = visible tear; one clock removes the failure mode.
2. **The shader only ever sees `phase ∈ [0,1)`.** Visual scale is driven by fractional phase, so appearance resets every Δ by construction. The ever-growing `logDepth` touches *only* the HUD counter. Infinite zoom, bounded shader input, seam guaranteed rather than tuned.

> **Enforce the invariant as a runtime guard.** At startup, a dev assertion samples every periodic derivation near 0 and near 1 and fails loudly if `|f(ε) − f(1−ε)| > tol`. Continuity becomes an automated check, not a hope.

### 4.5 Shader-side architecture

Internal GLSL structure; uniforms grouped by origin (clock / derived-state / static config):

```
pixel → centered uv → complex-log:  (angle, fract((log|z| + phase)/Δ))
      → apply Weyl shear + Ricci volume to coords        // conformal terms
field = mix(smoothField, latticeField, criticality)      // + collapseField
      → bloom overlay (Hawking point)
      → tone-map → output
```

- Each field is a named function taking phase-driven params; composition by `criticality`.
- **Fullscreen triangle, not quad** — no diagonal seam, one primitive.

---

## 5. Conventions

- **Delta-time everywhere.** Pacing integrated against real `dt`, never frame count — identical on 60/120 Hz.
- **Sign-flip for modes.** `zoomDir` is a single sign; the v2 cosmic-web zoom-*out* becomes a strategy that flips it and swaps the field, reusing clock + HUD untouched. Architect the seam now, build the mode later.
- **Sinks are an interface.** GPU and HUD both implement `(state) → void`. Audio (deferred) is just a third sink; nothing structural changes when added.
- **DPR-aware sizing**, capped; resize + `visibilitychange` (pause when hidden) handled in the input layer.
- **No hardcoded constants outside CONFIG; no experience tuning outside TIMELINE.**

---

## 6. Robustness & dev tooling

- Graceful failure on context-creation; surface shader compile/link errors; survive `webglcontextlost` / `restored`.
- `DEBUG` flag overlays live phase + uniform values.
- `?phase=0.42` URL param jumps to any phase for visual inspection.
- Keyboard scrub.
- These make every stage independently inspectable — important since stages 5–6 only appear briefly in normal playback.

---

## 7. The spine — timeline table (one octave, all phase-driven)

| phase | stage | what's drawn | confidence |
|---|---|---|---|
| 0.00–0.15 | Smooth spacetime | gently curved grid, light bending | solid |
| 0.15–0.35 | Crystallization | grid snaps into a repeating lattice | solid |
| 0.35–0.50 | Knife-edge | lattice balanced; ε injected; fork | solid |
| 0.50–0.70 | Echoing | self-similar repeat; **Δ named**; zoom = RG step | solid |
| 0.70–0.85 | BH → Hawking point | micro-BH forms, evaporates to a burst | contested |
| 0.85–1.00 | Conformal crossover | Ricci→0, Weyl shear persists, burst seeds next sky | interpretive |
| →0.00 | (seam) rebirth ≡ smooth start | identical to phase 0 | — |

**HUD throughout:** zoom depth (log), aeon counter, current regime label, live Δ value, one-line "what you're seeing," pause/scrub control.

---

## 8. Build sequence (ship incrementally)

0. **Lock the phase model on paper** — define every feature as f(phase); confirm phase 1 ≡ phase 0. *(Cheap; prevents the seam bug.)*
1. **Shader core** — seamless infinite log-zoom over a *static* self-similar field. **Acceptance test: the phase wrap is invisible.** De-risks the whole project.
2. **Phase-driven morph** — smooth → crystal → collapse keyed to `phase`.
3. **Overlay** — HUD + confidence-tagged captions on the shared clock.
4. **Crossover physics** — Hawking bloom + Ricci→0 / Weyl-shear handoff that closes the seam.
5. **Optional layers** — variable zoom-rate pacing, holographic frame (*if it earns its place*), audio, then the v2 zoom-out cosmic-web mode.

---

## 9. Constants & glossary

- **Δ (Delta) ≈ 3.44** — Choptuik echo-scaling period (massless scalar field; field-dependent). The zoom octave length and RG eigenvalue. *One definition, in CONFIG.*
- **γ (gamma) ≈ 0.37** — Choptuik mass-scaling exponent. Not needed for the core loop; noted for accuracy.
- **Criticality** — 0 (smooth) → 1 (collapse); drives the smooth↔lattice blend.
- **Ricci term (`volume`)** — isotropic volume change; → 0 at crossover.
- **Weyl term (`shear`)** — anisotropic, area-preserving distortion; the conformally-invariant piece that persists across the boundary.
- **Aeon** — one cosmological cycle in CCC; here, one zoom octave (a labelled narrative mapping).
- **Hawking point** — concentrated remnant of an evaporated supermassive black hole from the prior aeon, seeding the next aeon's CMB. *Contested.*
- **Crossover** — the conformal boundary joining one aeon's scaleless future to the next aeon's Big Bang.

---

## 10. Open items to confirm at each step

- Does the holographic frame earn its place, or is it cut? (Default: cut unless it clearly adds.)
- Audio: confirm it stays off-by-default and last.
- v2 cosmic-web mode: confirmed deferred; seam (`zoomDir`) is in the design now.
