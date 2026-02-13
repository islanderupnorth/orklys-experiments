# "How Screwed Are You?" — Viral Energy Campaign Plan

## Context

Orklys wants a viral marketing campaign that exposes how much people overpay on energy, generates outrage/curiosity, and funnels them toward forming Energy Communities — inspiring people to **demand to own their own electricity**. The project is scaffolded at `projects/you-are-screwed-campaign/` with empty content files ready to fill.

**Decided:**

- Campaign name: **"How Screwed Are You?"**
- Viral mechanic: **Bill photo upload** (highest virality)
- Conversion goals: **Dual** — join or create an Energy Community + email list building
- Mission: Inspire people to demand to own their own electricity

---

## Skills vs Agents (How We'll Work)

**Skills** = reusable instruction sets (`.claude/skills/`) that give Claude specialized expertise. Invoked automatically or via `/skill-name`. Think of them as "expert hats."

**Agents** = autonomous sub-processes for parallel multi-step work. We'll use them to run multiple skills simultaneously.

**Our approach: Skills for creative expertise, agents for parallel execution.**

---

## Skill Stack (4 skills)

| #   | Skill                     | Source                  | Purpose                                                      |
| --- | ------------------------- | ----------------------- | ------------------------------------------------------------ |
| 1   | `campaign-planning`       | skills.sh — **Install** | Full campaign strategy: objectives, audience, channels, KPIs |
| 2   | `viral-generator-builder` | skills.sh — **Install** | Design the bill-upload viral mechanic + shareable results    |
| 3   | `create-converting-copy`  | Already installed       | Landing page copy: hero, pain/agitation, CTA, social proof   |
| 4   | `frontend-design`         | Built-in                | Build the HTML/CSS page with high design quality             |

---

## Scope: Visual Mockup / Pitch Concept

This build is a **pure visual mockup** — a polished, compelling pitch for the campaign concept. No actual bill parsing or backend logic. The goal is to nail the copy, design, and flow so it can be shown to stakeholders, tested for reactions, and iterated on.

**What it includes:**

- Stunning landing page with full copy (hero → problem → solution → CTA)
- Visual bill upload UI (drag-and-drop area that looks real but is non-functional)
- Mock "Screwed Score" result screen (hardcoded example showing the concept)
- Dual CTA: "Join your Energy Community" + email capture
- Mobile-first design (viral sharing happens on phones)
- Shareable score card design (looks good as a screenshot)

**Geography:** Denmark-first (DKK, Danish energy context) with EU-adaptable framing (EUR references, EU energy community directives).

**Future phases (not in scope now):** Actual bill upload parsing, real savings calculations, backend integration.

---

## Execution: Copy First, Then Build

### NOW — Step 1: Landing Page Copy (this session)

**Skill:** `create-converting-copy` (already installed)
**Output:** `projects/you-are-screwed-campaign/drafts-copy/draft-copy-1.md`

Use the converting copy skill to write the full landing page copy:

- Hero: "How Screwed Are You?" headline + subheadline about energy bills
- Problem + Agitation: the energy rip-off, what people don't know
- Solution: Energy Communities — own your electricity
- How it works: upload bill → see score → join community
- Social proof: placeholder stats, EU directive references, community success stories
- Dual CTA: "Launch Your Energy Community" + "Get Your Free Report"
- Tone: Bold, provocative, conversational. Not corporate. Denmark-first, EU-ready.
- Mission thread: inspire people to **demand to own their own electricity**

### Campaign Sequence (iterative, with handoffs)

Each phase follows a **draft → feedback → refine → handoff** loop. A phase only hands off to the next when the team agrees it's in a good space. Feedback is tracked in `context/learnings-and-feedback.md`.

```
Phase 1: COPY (now)
  └─ Skill: create-converting-copy
  └─ Deliverable: Full landing page copy in drafts-copy/
  └─ Loop:
     1. Draft copy (draft-copy-1.md)
     2. Share with team → collect feedback
     3. Refine → draft-copy-2.md, draft-copy-3.md, etc.
     4. Team signs off → HANDOFF to Phase 2
  └─ Who: Us first, then team reviews

Phase 2: CAMPAIGN STRATEGY (after copy is solid)
  └─ Skill: campaign-planning (install from skills.sh)
  └─ Deliverable: Campaign plan — audience, channels, KPIs, calendar
  └─ The approved copy informs the strategy (not the other way around)
  └─ Loop: Draft plan → team input on channels + budget → refine → HANDOFF
  └─ Who: Team collaboration (marketing, partnerships)

Phase 3: VIRAL MECHANIC DESIGN (after strategy approved)
  └─ Skill: viral-generator-builder (install from skills.sh)
  └─ Deliverable: Spec for bill upload → score → share flow
  └─ Loop: Design spec → dev/design feasibility review → refine → HANDOFF
  └─ Who: Bring in developers + designers

Phase 4: WEBSITE BUILD (after copy + mechanic + strategy approved)
  └─ Skill: frontend-design (built-in)
  └─ Deliverable: Full HTML/CSS mockup in drafts-website/
  └─ Loop: Build → team review → iterate → HANDOFF
  └─ Who: Review with full team before publishing

Phase 5: PUBLISH & LAUNCH
  └─ Promote approved draft to pages/ per CLAUDE.md workflow
  └─ Who: Full team — marketing, community, partnerships
```

---

## Files to Create/Modify

| File                                                                          | Action                                                      |
| ----------------------------------------------------------------------------- | ----------------------------------------------------------- |
| `projects/you-are-screwed-campaign/context/context.md`                        | Update with finalized campaign brief                        |
| `projects/you-are-screwed-campaign/drafts-copy/draft-copy-1.md`               | Full landing page copy (via `create-converting-copy` skill) |
| `projects/you-are-screwed-campaign/drafts-website/draft-website-1/index.html` | Complete page with upload UI, score mechanic, CTAs          |
| `projects/you-are-screwed-campaign/drafts-website/draft-website-1/style.css`  | Styling — bold, provocative, mobile-first                   |
| `projects/you-are-screwed-campaign/context/learnings-and-feedback.md`         | Track iterations                                            |

## Verification (for Step 1 — Copy)

- Read through draft-copy-1.md end to end — does it tell a compelling story?
- Check tone: provocative, bold, conversational — NOT corporate or preachy
- Verify dual CTA presence: community join + email/report capture
- Confirm Denmark-specific references + EU adaptability
- Does the "How Screwed Are You?" concept come through clearly?
- Does it inspire the feeling: "I need to own my electricity"?
- Are placeholders clearly marked for real data (stats, testimonials)?
