# homefolk — pilot reframe design

**Date:** 2026-06-03
**Status:** Approved direction; pending final detail review before implementation plan.
**Applies to:** `/Users/leyichang/Desktop/freshy/index.html` (single self-contained file, no build step).

---

## 1. Problem & goal

The current site presents as "drop a tiny home on a Hackney site and step inside" — which reads as *finished homes you can move into now*. That is the wrong frame.

homefolk is at an earlier stage: it is **communicating and building support for a pilot / research proposal**. The pilot is intended to be tested first with students (esp. Masters/PhD / academic communities) before scaling to wider residential use.

**Goal of the reframe:** make every visitor feel they are *helping design, test, and validate a pilot* — moving it concept → pilot → research-backed proof → future scaling. The correct message is: **"Help us test the pilot so this model can become credible, fundable, and scalable,"** never "apply to live here now."

### Success criteria
- No visitor should believe finished homes are being offered to the public.
- Each of the three audiences (student, researcher, council/community) gets a journey that speaks to *their* main question.
- The experience feels like a gamified civic design journey, ending in a tailored "pilot readiness" summary that captures the visitor's support.
- Built lean: stays in one HTML file, no new framework/build step, reuses existing code.

### Non-goals
- Not a public housing application or waitlist.
- Not a rebuild of the map / 3D builder (reused as-is).
- Not a multi-page app or a new tech stack.

---

## 2. Audiences (three roles)

Each role has one driving question:

| Role | Door label | Main question |
|---|---|---|
| 🎓 Student / pilot participant | "I might live in it" | *Would I help test this pilot?* |
| 🔬 Research / academic partner | "I might study it" | *Is this pilot credible enough to support?* |
| 🏛️ Community / council / planning | "I might host it" | *Should this pilot be supported here?* |

Framing per role:
- **Student** — not an ordinary housing application. Student-led pilot participation: willing to test, give feedback, live with some trial-and-error, help prove the model.
- **Research** — credibility to test/support/fund/study: evidence, feasibility, evaluation, legal structure, affordability, carbon, public-health framing, research outcomes.
- **Council/community** — a controlled trial/research stage (not an uncontrolled permanent rollout): safety, local trust, public space, consultation, greyfield land use, accessibility, governance, wider-community benefit.

---

## 3. Overall flow (Option C — problem-first, role-detected)

```
LANDING (hopeful hook)
  vision → problem + affordability stats → "this is a pilot, not finished homes" → three doors
        │
        ├─ 🎓 ── shared journey ──┐
        ├─ 🔬 ── shared journey ──┤── + role detour ── readiness summary (score + reflect + CTA)
        └─ 🏛️ ── shared journey ──┘
        │
        └─ optional, anytime: 🏠 try the pod builder (existing map + 3D)
```

The role choice is framed as a natural reaction to the problem ("I might live in it / study it / host it"), **not** a cold "select your category."

---

## 4. Landing hook (also fixes "point 4")

- **Tone:** hopeful possibility — warm, aspirational, then grounded.
- **Headline (working):** "What if a forgotten car park became somewhere to belong?"
- **Body:** underused Hackney plots sitting empty → homefolk wants to *test* whether they could become small, community-owned, low-carbon tiny-home villages → starting with a student-led pilot.
- **Stats:** focus on **affordability / rent burden only** (one sharp angle). Must be **real, citable UK / London figures** (e.g. share of the maintenance loan eaten by average student rent). **Do not fabricate** — source and cite during build; show source on hover/footnote.
- **Honesty line (required):** an explicit "This isn't finished housing. It's a pilot — and we need help to test it." This is the guardrail against the "are these ready to move into?" misread.
- **Three doors** lead into the journeys.

---

## 5. The journey engine (shared core + detour)

A single **data-driven engine** runs all three roles. Questions, scoring and summaries live in compact config objects; there is **one** journey runner, not three hand-built flows.

### 5.1 Shared spine (every role, one decision each)
Five quick chapters, one question per chapter, each framed for the active role:

1. **Site** — where should the pilot go? (e.g. greyfield / car park / garage court)
2. **Design** — what are the homes like? (size, shared vs private, eco features)
3. **Safety** — how do we keep it safe? (fire, access, security, oversight)
4. **Community trust** — how do we win local trust? (consultation, temporary/reversible, benefit to area)
5. **Evidence** — how would we know it works? (what to measure, who evaluates)

Each chapter presents 2–3 options; each option carries a small score contribution (see 5.3).

### 5.2 Role detour (1–2 extra questions after the spine)
- 🎓 **Student** — living-in & feedback (e.g. "what would you most want to test / feed back on?").
- 🔬 **Research** — evidence & legal (e.g. "what would make this credible enough to back?").
- 🏛️ **Council/community** — consultation & governance (e.g. "what assurance would let you support this locally?").

### 5.3 Scoring model
- Each chosen option contributes points toward a **"pilot readiness" score** (0–100).
- Per-role **weights** tilt the score toward what that role cares about (researcher weights evidence/legal higher; council weights safety/consultation; student weights participation/feedback).
- Score is a *reflection of the visitor's own choices*, framed as "in your view, this pilot is …" — never a real-world claim.

### 5.4 Config shape (illustrative, not final)
```js
const CHAPTERS = [
  { id:'site', frame:{student, research, council},
    options:[ {label, blurb, points} ... ] },
  // design, safety, trust, evidence ...
];
const ROLES = {
  student:  { door, mainQ, weights:{...}, detour:[...], cta:{...} },
  research: { ... },
  council:  { ... },
};
```
One runner reads `CHAPTERS` + the active `ROLES[role]`, renders each question, accumulates score, then renders the summary.

---

## 6. Readiness summary (the payoff — integrated single screen)

One screen combining all three:
1. **Score meter** — "Your pilot readiness ▓▓▓▓▓░ 82%".
2. **Reflection** — a short, tailored sentence + a recap of the choices that drove the score ("because you chose: greyfield site, strong safety plan, early consultation…").
3. **Role-specific call-to-action** — one clear next step that captures support:
   - 🎓 Register to help test the pilot
   - 🔬 Express research / funding / collaboration interest
   - 🏛️ Register support / request the briefing

### Support-capture mechanism
Default: **Google Form** (one form with a hidden/role field, or one per role) — free, no code, responses land in a spreadsheet. Alternatives if a native look is wanted: **Formspree** (looks native, free tier) or **`mailto:`** (zero setup, clunky). Mechanism is swappable; the journey is designed around "there is a sign-up here." **Decision deferred to build; default = Google Form.**

---

## 7. Optional shared activity — the pod builder

The existing **map + 3D pod builder is reused as-is**, relinked as an optional "try the pilot / play with it" activity reachable from the landing and/or summary. Reframed in copy as *"help us test where & how"*, not "claim a home." No functional changes required in this phase.

---

## 8. Build approach (credit/token-efficient — user instruction)

- Stay inside the single **`index.html`**; no new framework, build step, or libraries.
- Reuse the existing **`showScreen()`** screen-switching system and brand tokens.
- **Data-driven**: three roles run off one engine + compact config — less code to write and maintain.
- Reuse the map + 3D builder unchanged.
- No sub-agents / heavy tooling; straightforward edits.

---

## 9. Related visual fixes (separate, later phase)

Tracked separately, to follow the reframe (see also points 1–3 of the original brief):
1. **Logo swap** — replace the chunky 3-bar `#aframe` SVG (used in landing + both banners) with the brand logo. *Blocker:* `light-a.png` is a studio screenshot, not a logo — confirm the actual logo file. Original site uses a thin single-line A-frame + "homefol" + coral "k".
2. **Map clarity** — keep Leaflet; swap base tiles to a muted/greyscale basemap (CartoDB Positron / Stadia Alidade Smooth / Stamen Toner-Lite); strengthen polygon contrast. `.lyrx` files = ArcGIS styling + data *pointer* only (no geometry; Leaflet can't read them) — useful as colour reference and as a sign real Hackney GIS data exists that could be exported to GeoJSON.
3. **Site counts show "–"** — legend counts only populate when the live Overpass API returns data; Overpass is frequently throttled. Likely reliability, not a layout bug (confirm by running). Robust fix: replace live Overpass with a **pre-baked static GeoJSON** (ideally from the `.lyrx` data) — fixes counts + map clarity + load speed at once.

---

## 10. Open details to resolve in this spec / early build
- Exact wording of each chapter's question + options (must keep pilot framing).
- Final per-role detour questions.
- Real affordability stats + sources.
- Support-capture mechanism choice (default Google Form).
