# prompts.md — staged build prompts for Claude Code

Paste these **one at a time**. After each, run it in a browser and verify before moving on. The whole method is: prove the seam on a trivial field first, then layer richness on top of a foundation you trust.

> Before starting, optionally install the shader skill:
> `/skill install shader-dev@MiniMax-AI/shader-dev`
> (If the path 404s, clone the repo and copy its skill folder into `.claude/skills/`. **Read its SKILL.md first** — confirm it's GLSL/WebGL, not WebGPU.)

---

## Step 1 — Seamless zoom core (the gate)
```
Read CLAUDE.md and PLAN.md. Implement STEP 1 ONLY in index.html, building on the
existing scaffold.

Replace the placeholder fragment shader with a seamless infinite log-zoom over a
STATIC self-similar field (e.g. layered value noise sampled in complex-log /
log-polar coords). The JS Clock owns phase; the shader receives `phase ∈ [0,1)`
as a uniform and must NOT integrate its own clock for the zoom.

ACCEPTANCE TEST: as phase runs 0→1 and wraps to 0, the transition is visually
INVISIBLE — no jump, no flash, no drift. Verify with the ?phase= param at 0.0 vs
0.999 and by watching several wraps. Keep the startup continuity assertion passing.

Do not add fields, stages, HUD, or physics yet. Stop here.
```
**Verify:** scrub `?phase=0.0` and `?phase=0.999` — identical. Watch 3–4 wraps live — no seam. If it drifts, suspect a stray `u_time` in the zoom.

---

## Step 2 — Phase-driven morph
```
STEP 2 ONLY. Drive a `criticality` value (0→1→0 across the octave, from derive())
that blends the static field: smooth spacetime → regular lattice (crystal) →
collapse. The lattice and collapse are GLSL field functions composed by
`criticality`. Add the knife-edge fork (forkSign) as a subtle bifurcation cue.

The seam invariant still holds: criticality(0) must equal criticality(1). Re-run
the continuity assertion. Don't touch HUD or crossover yet. Stop.
```
**Verify:** the morph reads as smooth→crystal→collapse; wrap still invisible; assertion still green.

---

## Step 3 — HUD + confidence captions
```
STEP 3 ONLY. Add the HUD/overlay shell per the aesthetic direction in CLAUDE.md
(scientific-instrument readout; distinctive technical type, NOT Inter/Roboto/
Space Grotesk; cohesive dark palette via CSS vars; confidence colours as accents).

Drive captions + regime label + confidence tag from the TIMELINE table on the
shared clock. Show: zoom depth (log), aeon counter, live Δ, current stage label,
one-line "what you're seeing", confidence tag. Diff the DOM; throttle text ~10 Hz.
Wire play/pause/scrub. Stop.
```
**Verify:** captions track the stage; counters advance; text isn't rewritten every frame; design isn't generic.

---

## Step 4 — Crossover physics (closes the seam meaningfully)
```
STEP 4 ONLY. Implement the conformal crossover that makes the seam physical:
- Hawking-point bloom in the BH→evaporation stage (bloom from derive()).
- At the crossover, drive the Ricci term (volume, isotropic) → 0 while keeping the
  Weyl term (shear, anisotropic, area-preserving). What carries across the boundary
  is the conformally-invariant shear. The bloom seeds the next octave's field.

The phase=1 → phase=0 handoff must remain visually identical (the rebirth IS the
smooth start). Re-run the continuity assertion. Stop.
```
**Verify:** evaporation→bloom→rebirth feels continuous; volume→0 while shear persists; wrap still invisible.

---

## Step 5 — Optional layers (only what earns its place)
```
STEP 5, pick-and-choose. Candidates, in order:
- Variable zoom-rate pacing (slow through crystallization/collapse, fast through
  smooth space) — integrate against dt; must not affect the spatial seam.
- Holographic frame (boundary ring encoding a 1D radial projection of the 2D bulk)
  — ONLY if it visibly adds; otherwise cut.
- Audio sink (frequencies spaced by Δ) — last, OFF by default.
- v2 cosmic-web zoom-OUT mode — flip zoomDir, swap field; reuse clock + HUD.
Implement one at a time. Stop after each.
```

---

## If the seam ever tears (debugging order)
1. Is a `u_time` feeding the zoom transform? Remove it (cosmetic-only `u_time` is fine elsewhere).
2. Is the shader integrating its own clock? It must read `phase` only.
3. Does a periodic field in derive() satisfy `f(0)===f(1)`? The assertion should already catch this — check it's still wired.
4. Is `logDepth` (monotonic) leaking into a shader uniform that drives appearance? Only `phase` should.
