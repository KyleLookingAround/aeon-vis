# AEON — Conformal Zoom Visualiser

A single self-contained `index.html`: a seamless infinite conformal zoom that walks through the life of a cosmological *aeon* — spacetime-crystal critical collapse → microscopic black hole → Hawking point → conformal rebirth — narrating the physics as it goes, with each stage tagged by how solid the science is.

This package is set up to be **built in Claude Code, one step at a time.** It ships as a runnable scaffold; the visualiser logic is intentionally stubbed for you to fill in.

## Files
| file | what it is |
|---|---|
| `index.html` | **Runnable scaffold.** WebGL boot + placeholder shader + the full architecture wired up with stubs and TODOs. Open it — it runs. |
| `CLAUDE.md` | Project memory for Claude Code: the rules, the hard invariant, the contract, conventions, aesthetic direction. Read automatically as context. |
| `PLAN.md` | The full reference doc — all the reasoning behind every decision. |
| `prompts.md` | Copy-paste build prompts, one per step, each with a verification gate. |

## Run it
No build step. Either:
```bash
# any static server, then open the printed URL
python3 -m http.server 8000
# or
npx serve .
```
…or just open `index.html` in a browser directly. (A server is nicer for live-reload while iterating.)

You'll see a dark instrument view with a reticle that pulses with `phase`, a HUD readout, and stage captions cycling. **Controls:** `space` play/pause, `←/→` scrub, `?phase=0.42` jump to a phase, `?debug` show live state.

## Workflow with Claude Code
1. Open this folder in Claude Code (it picks up `CLAUDE.md` automatically).
2. *(Optional)* install the GLSL skill:
   `/skill install shader-dev@MiniMax-AI/shader-dev`
   — if that 404s, clone the repo and copy its skill folder into `.claude/skills/`. **Read its SKILL.md first** to confirm it's GLSL/WebGL, not WebGPU.
3. Work through `prompts.md` **in order**, verifying each step in the browser before the next.

The skills split the work: `shader-dev` → the GLSL (zoom transform, fields, Weyl/Ricci distortion); `frontend-design` → the HUD/shell and overall non-generic look.

## The one thing that matters
**`phase = 1` must look identical to `phase = 0`** so the infinite zoom loops seamlessly. Two rules protect it: the **JS owns the only clock** (the shader never integrates its own time for the zoom), and the **shader only ever sees `phase ∈ [0,1)`**. A startup continuity assertion and the `?phase=` param are your guardrails. If the wrap ever visibly tears, that rule got broken — see the debugging order at the bottom of `prompts.md`.

## Build order
Scaffold (here) → **step 1: seamless zoom on a static field** (gate: invisible wrap) → step 2: smooth→crystal→collapse morph → step 3: HUD + confidence captions → step 4: crossover physics (Hawking bloom, Ricci→0 while Weyl shear persists) → step 5: optional layers.

---
*Physics honesty: the critical-collapse / echoing mathematics is solid; Hawking points are contested evidence; the conformal-crossover and seed-of-galaxies visuals are interpretive. The UI labels which is which — keep it that way.*
