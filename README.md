# Truss Analysis Lab

A single self-contained HTML file that teaches the analysis of **simply supported
plane trusses** by the **method of joints** and the **method of sections**. No build
step, no framework, no local dependencies — open `index.html` in any browser, online
or off. The only external request is the Google Fonts stylesheet, and there is a
system-font fallback if it does not load.

It follows the vector-addition → force-systems → beam-reactions labs, and continues
their conventions: up and counter-clockwise positive, angles from the +x axis, a
worked solution that shows formula → substitution → value for every step.

## What it teaches

**Tension is positive, everywhere.** Every unknown bar force is drawn pulling its
joint outward. A negative answer means compression, and no arrow is ever redrawn —
which is the single discipline that keeps sign errors out of a fifty-bar truss.

The organising idea is that both methods are the same statics applied to different
free bodies:

- **Method of joints** — a joint is a concurrent force system, so it gives two
  equations and no moment equation. The march must therefore begin where only two
  members are unknown, and the order of joints is not free. Click any joint and the
  worked solution rebuilds the shortest legal chain of joints that reaches it.
- **Method of sections** — a cut portion is a *rigid body*, so it gains the moment
  equation a joint never has. Take moments where the other two cut members intersect
  and both vanish: one equation, one member, no marching. When the other two are
  parallel chords there is no intersection, and summing forces perpendicular to them
  says the same thing — the diagonal carries the shear.

The check block closes the loop: the same member solved both ways must agree.

## Layout

Three tabs, as in the other labs.

- **Explore** — the sandbox. Drag joints to reshape the truss and watch the forces
  follow; drag the tail of a load arrow; drag the section cut. Presets for a triangle,
  king post, Pratt, Howe, Warren, pitched roof, and a randomiser. Span, depth and
  panel count are editable, and the depth box is the point of the whole lab: chord
  forces are a bending moment divided by the depth.
- **Walkthrough** — ten numbered steps from "every member is a two-force member" to
  "a truss is a beam turned inside out", each driving the sandbox and narrating it
  with live numbers from the truss actually on screen.
- **Practice** — seven generators, marked, with a revealable worked solution:
  reactions, a support joint, zero-force members, a marched chain, a chord by
  sections, a diagonal by sections, and a design question on truss depth. Wrong
  answers are diagnosed by name — entering the shear instead of the diagonal force,
  putting the moment centre at a support, dividing by the member length instead of
  its perpendicular arm — rather than being told merely "incorrect".

Copy-as-text and print buttons export the same worked solution, including the full
member force table.

## Under the hood

- One `<style>` block, one ES5 IIFE. `var` only, no arrow functions, no classes.
- Theme tokens on bare `:root`, redefined for `prefers-color-scheme: dark`, again for
  `[data-theme="dark"]`, and a fourth ink-friendly set inside `@media print`. The
  theme button persists the choice in `localStorage`.
- A single `state` object; `render()` builds the whole SVG as a string and assigns
  `innerHTML`, then updates the numeric panels through a `setInput` helper that skips
  `document.activeElement` so typing is never clobbered.
- Reactions come from whole-truss rigid-body equilibrium — the beam problem
  unchanged. Member forces come from marching joint to joint, exactly as a student
  would, so the narration and the answers share one derivation. If the march stalls
  (no joint left with fewer than three unknowns) the app says so, explains that this
  is what sections are for, and finishes by solving all `2j` joint equations at once.
- Determinacy is checked as `m + r = 2j`, and the three failure modes are treated as
  real outcomes rather than as errors: indeterminate, mechanism, and coincident
  supports.

## Verified, not eyeballed

Before delivery the lab was driven headlessly over HTTP:

- **155 generated practice problems re-solved independently.** A separate solver,
  written to the prose in the visible prompt and assembling all `2j` joint equations
  as one matrix, reconstructed each truss from the prompt text alone and fed its
  answers back through the app's own Check button. 155/155 accepted, across all seven
  generators — which validates the generators, the prompts' completeness, the stored
  answers, the tolerance, and the marching solver against an independent method.
- **~490 randomised states** across every preset, both methods, every overlay toggle,
  extreme spans and depths, zero and enormous loads, and dragged-into-degeneracy
  geometry: no thrown errors and no `NaN`, `Infinity` or stray `undefined` in the
  emitted SVG, worked solution, checks or member table.
- **100 layout measurements** of every SVG `<text>` via `getBBox()` against the
  `viewBox`, at desktop and at a 375 px mobile viewport: nothing clipped, and no
  horizontal page overflow.
- **Every text label on the stage across 40 practice problems**, to prove no member
  force, reaction or section value leaks onto the locked diagram before the student
  presses Check. It did, at first — the section arrows were drawn ungated, printing
  the answer next to the question.

The process also caught: label collisions between the joint free body and the member
labels behind it; a walkthrough step calling a bottom chord a diagonal; a stage hint
that stopped tracking the method once the walkthrough switched it; and a claim about
truss depth that the numbers contradicted — it is the *verticals*, not the diagonals,
that depth leaves untouched. Section results were also checked by hand against the
joint results for a Pratt and a Warren.
