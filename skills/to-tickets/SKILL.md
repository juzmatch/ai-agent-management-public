---
name: to-tickets
description: Break a plan, spec, or PRD into independently-grabbable tickets on the project issue tracker using tracer-bullet vertical slices. Groups the slices under a parent overview issue (the tracker's native parent primitive) holding the summary, scope, spec link, and open-question roll-up — so the whole feature and its breakdown stay in one place. Carries any open technical questions into the tickets as explicit "discuss with dev" items (and gates those tickets), proposes an MVP slice when the breakdown is large so the PM can prioritise what ships first, and lets the PM choose how coarse or fine the tickets should be. Use when a user wants to convert a plan or a functional spec (e.g. from grill-pm) into implementation tickets.
---

# To Tickets

> Shared skill (Juzmatch). Install with `npx skills add juzmatch/ai-agent-management-public --skill to-tickets`.
> Assumes the Atlassian MCP is connected. Pairs with the `to-aur-ticket` skill for one-off tickets;
> the AUR-331 card body below is the same standard.

Break a plan into independently-grabbable issues using vertical slices (tracer bullets). Built to consume a functional spec (e.g. `<feature>.functional.md` from grill-pm) or any plan/PRD already in context.

## Issue tracker

Publishes to your project's issue tracker — no up-front setup needed. If you don't already know which tracker this project uses, ask the user once (GitHub Issues / Linear / Jira / …) and how to reach it (MCP tool, `gh` CLI, etc.), then remember it in your memory for this project so later runs don't re-ask. **Juzmatch/Aurora: file to the Aurora Jira board — project key `AUR` (Epic AUR-17 = WS-0, etc.). Legacy/BAU work stays on `CP`. Still confirm the target with the PM before publishing (filing tickets is outward-facing).**

## Process

### 1. Gather context

Work from whatever is already in the conversation context. If the user passes a reference (issue number, URL, or a path like `<feature>.functional.md`) as an argument, fetch/read its full body. If the source is a grill-pm functional spec, read **all four sections** — Resolved Functional Decisions, Out of Scope, Glossary touched, and **Dev Open Questions** (you will need the open questions in step 4).

### 2. Choose granularity (ask the PM first)

Before drafting, ask the PM how coarse or fine they want the tickets — this drives everything below:

- **Coarse** — a few large tickets (≈ epic-sized; one per major capability). Best for early scoping or when one owner takes a big chunk.
- **Standard** (default) — vertical slices, each a thin end-to-end path. Most teams want this.
- **Fine** — many small slices. Best for parallelising across devs or AFK agents.

If the PM doesn't say, default to Standard and tell them you did.

### 3. Explore the codebase (optional)

If a repo is present and you haven't explored it, do so to ground titles/descriptions in real behaviour. Use the project's domain glossary (CONTEXT.md) vocabulary and respect ADRs in the area you're touching. No repo → proceed from the spec; do not block.

### 4. Draft vertical slices

Break the plan into **tracer bullet** issues at the chosen granularity. Each issue is a thin vertical slice cutting through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

Slices are **HITL** or **AFK**. HITL needs human interaction (an architectural decision, a design review, an unresolved question). AFK can be implemented and merged without human interaction. Prefer AFK where possible — **but see the open-questions rule below.**

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones (unless the PM chose Coarse)
</vertical-slice-rules>

#### Map the open technical questions onto the slices

Every Dev Open Question from the source must end up visible and actionable in the tracker — never silently dropped:

- **Slice-specific question** → list it in that slice's `Open technical questions` section. **Force the slice to HITL** — a slice with an unresolved question cannot be started until it is resolved with dev.
- **Cross-cutting question** (not tied to one slice) → collect all such into a single discussion ticket: `[Discuss] Open technical questions — <feature>`. Link every slice it affects as **blocked by** this ticket.

The goal: a dev cannot pick up a slice without first seeing — and discussing — the open questions that bound it.

**Make every reference self-contained.** The functional spec may number its decisions (D1, D2, …) for its own bookkeeping — **never carry a bare ID like `D7` into a ticket.** A reader on the tracker doesn't have the spec open; `(bounds D7)` is noise to them. Replace each ID with the decision's gist in plain words: `(bounds the commission base — list / after-discount / buyer-paid)`. Same for blockers and acceptance criteria.

### 4b. Design-readiness check (per slice — ask the PM)

For every slice with a visual surface, ask the PM: **"Is the design for this slice confirmed/delivered?"** Do not assume. Three branches:

- **Design ready** → write it as a normal story: behavior AC + UI AC, Figma link, screens referenced by the designer-handoff **screen IDs (S1, S2…)**.
- **Design NOT ready, logic layer is substantial** (≥ a few days of dev work — calc engines, state machines, validation rules) → **split**: file a TECHNICAL ticket now covering schema + service behavior only, with **testable non-UI acceptance criteria** (API/service behavior + automated tests against the field sheet and functional rules — "เตรียม database" alone is NOT an AC). Zero UI wording in it. Mark it `🎨 UI story to follow` (label or a checklist line in the description) and record the pairing in the local draft index — the matching UI story is written at design drop (see "Design-drop intake" below). Tech tickets implement SPEC'D fields only, no speculative extras; any ambiguity → Open technical question with a dated owner, never dev's solo call.
- **Design NOT ready, UI-thin slice** (mostly presentation, little logic) → **hold it**: keep the draft local, don't file until the design arrives. A filed ticket that can only churn is worse than a draft.

### 5. Propose an MVP slice when the breakdown is large

If the breakdown is large (rule of thumb: more than ~6 slices, or the PM flags it as big), don't just hand over everything flat. Propose a **phased cut** so the PM can prioritise what ships first:

- **MVP (Phase 1)** — the minimal set of slices that delivers a thin but genuinely shippable product (the smallest end-to-end thing a real user could use).
- **Phase 2+** — everything else, grouped sensibly.

Present the cut and ask the PM to confirm or re-balance it (move slices between phases). Each published issue records its phase in the `Phase` field. This is a prioritisation aid, not a hard gate — the PM owns the call.

### 6. Quiz the user

Present the proposed breakdown as a numbered list. For each slice show:

- **Title** — short descriptive name
- **Phase** — MVP / Phase 2+ (if you proposed a cut in step 5)
- **Type** — HITL / AFK
- **Blocked by** — which slices or discussion tickets must complete first
- **Open questions** — any Dev Open Questions this slice carries
- **User stories covered** — if the source has them

Ask:

- Does the granularity feel right? (too coarse / too fine — re-run step 2 if so)
- Is the MVP cut right? Should anything move phase?
- Are the dependency relationships correct?
- Should any slices be merged or split?
- Are the correct slices marked HITL vs AFK?

Iterate until the user approves.

### 7. Create the parent overview issue

If the breakdown has **2+ slices**, create a parent overview issue **first**, so each slice can be linked to it as you publish. Use the tracker's **native grouping primitive** — Jira Epic, GitHub tracking issue, Linear parent/project, etc. — do not assume a specific tracker or hard-code "Epic".

The overview holds only what the tracker won't auto-generate. The breakdown itself is the **native parent→child links** you set in step 8 — do **not** hand-maintain a list of child tickets here (it duplicates the tracker's own view and goes stale). Only if the tracker cannot surface children under a parent should you list the slices here as a fallback.

<overview-template>
## Summary

The problem and the solution, in brief — from the functional spec.

## Design

One placeholder for the creator to drop the key design / hero mock (delete if the feature has no visual surface):

`📷 [[ screenshot: key design / hero mock of the feature — paste or delete ]]`

## Scope / Out of scope

From the functional spec.

## Functional spec

A link to `<feature>.functional.md` (the full functional context — do not duplicate it here).

## Open technical questions

Roll-up: link the `[Discuss]` ticket (if any) and note which slices are gated — HITL until their questions are resolved with dev.
</overview-template>

Single-slice feature: skip the overview; the one slice links the functional spec directly.

### 8. Publish the slices to the issue tracker

For each approved slice, publish a new issue using the template below, in dependency order (the `[Discuss]` ticket and any blockers first, so you can reference real identifiers). **Link every slice to the parent overview issue** via the tracker's parent mechanism.

**Juzmatch Jira — which card body to use (updated 02/10/2569):**
- **Project `AUR` (Aurora)** → use the **AUR card** below (standard set by Aod in **AUR-331**, distilled from AUR-266/330/35/146/292/131/16). The generic `<issue-template>` further down is the fallback for non-Juzmatch trackers only.
- **Project `CP`** → keep the `to-aur-ticket` skill's CP template (User story / SOW / AC / Technical Note).

<aur-card>
**Title:** `[System][Module] <สิ่งที่ทำ>` — e.g. `[JMS][Agent] อนุมัติใบสมัคร agent จากหน้า BD review`. System/module prefix is mandatory so the board filters by it.

> **Phase:** MVP / 2 · **Blocked by:** AUR-… (เหตุผลสั้น ๆ) · **Related:** AUR-…
> ⚠️ (only after an edit) อัปเดต dd/mm/yyyy — ส่วนนี้ทับฉบับเดิม ประวัติอยู่ในคอมเมนต์

### Background / ปัญหา
ตอนนี้… (1–2 sentences: what happens today, who suffers, how often)

| กรณี / ช่องทาง | สิ่งที่เกิดตอนนี้ |
| --- | --- |
| <who wants what, with volume> | <manual step + time cost / wait> |

**Workaround ที่ใช้ตอนนี้:** …

### Goal
One sentence, measurable: actor + action + result + constraint.

### User Story *(only when there is a clear end user; skip for refactor/bug/data work)*
**As** … **I want to** … **So that** …

### Scope
**In scope** · bullets · **Out of scope** · bullets — **both always present**; Out of scope names the adjacent things people will ask for.

### Business Rules
| หัวข้อ | กติกา |
| --- | --- |
| <limit / format / PII / rounding / who-can> | <exact value QA can check> |

### Role & Permission *(when roles differ)*
| Role | เห็น | กด/แก้ได้ |
| --- | --- | --- |
(+ note: API ต้องตอบ 403 สำหรับ role ที่ไม่มีสิทธิ์)

### UI Changes / States *(new or changed UI)*
Verbatim strings: label, loading text, success toast, error toast, disabled state. One bullet each.

### Data model / Field mapping *(new/changed field or enum)*
| Field | Type / Source | หมายเหตุ |
| --- | --- | --- |

---
### Acceptance Criteria
| given | when | then | Remark |
| --- | --- | --- | --- |
Real test values. Success **and** fail/edge rows. Permission row if roles differ. PII-absence row if data leaves the system.

### Figma / Screenshot *(mandatory for UI work)*
* success: <Figma node link> · fail: <Figma node link> · QA URL: …
Never fabricate or embed an image; leave the link for the creator.

### Dev Open Questions
- [ ] <real engineering choice, bounded by the functional rule in plain words>
- [ ] ไฟล์/service ที่คาดว่าต้องแก้: …

### References / Note *(optional)*
Field dictionary · related cards · "ถ้าทีมอื่นขอเพิ่ม X ให้เปิดการ์ดใหม่"
</aur-card>

**How an AUR card must read (learned from Aod's worked example on AUR-331 — tone, not just headings):**
- **Thai spec register.** No ครับ, no first person, no รบกวน/ขอความกรุณา. English nouns for system and tech terms stay in English (filter, toast, endpoint, role, enum). This is a spec, not a message.
- **Background is a quantified pain, not a feature description.** "copy มือ 24 หน้า ใช้เวลา ~40 นาที", "รอ 1–2 วัน". Every row: who, what they do today, what it costs. Name the current workaround in one line.
- **Goal is measurable.** "ผู้ใช้ OMS กด Export แล้วได้ไฟล์ .xlsx ตาม filter ภายใน 30 วินาที ไม่ต้องพึ่งทีม Data". Never "เพื่อเพิ่มประสิทธิภาพ".
- **Out of scope is specific.** "Export หน้าอื่น เช่น Deal, Buyer, Payment", "ส่งทางอีเมล", "CSV/PDF". Generic "อื่น ๆ" is not an out-of-scope.
- **Rules are values.** 10,000 แถว · `assets_YYYYMMDD_HHmm.xlsx` · dd/mm/yyyy พ.ศ. · ทศนิยม 2 ตำแหน่ง · **ไม่ export** เลขบัตร/เบอร์/อีเมล. If QA can't check a row, it isn't a rule yet.
- **UI strings verbatim**, in quotes, including the error text. Nobody should have to ask what the toast says.
- **AC are scenarios with numbers.** "Role BD กรอง สถานะ = Active (1,200 รายการ) กด Export → ได้ไฟล์ `assets_20261005_1430.xlsx` 1,200 แถว ภายใน 30 วินาที". Fail rows carry the exact error text. Include the 403 row, the double-click row, the PII-absence row where relevant.
- **Dev Open Questions are engineering choices** (server-stream vs browser, reuse endpoint vs new), never "TBD". List the repos expected to change.
- **Where the content comes from:** Background ← pain points in the WS canvas / confirm deck; Business Rules + Out of scope ← the functional spec's Resolved Functional Decisions and Out of Scope sections; AC values ← the field sheet and confirmed rules; Dev Open Questions ← the spec's Dev Open Questions. Pull per slice; do not leave rules only on the parent.
- **No SOW steps.** The old "1. ทำหน้า… 2. ต่อ API…" list is gone; rules + UI states + data model + AC replace it.
- Re-read the drafted body against the "How an AUR card must read" list above before showing it to the PM (no ครับ, quantified Background, measurable Goal, specific Out of scope, AC with real values).

Recommend a screenshot placeholder only where a visual genuinely reduces ambiguity (see **What to build** below). The placeholder is for the human creator to fill — **never fabricate, generate, or embed an image yourself.**

**Wire dependencies as REAL issue links, not just description text (dev-team request, Peng 14/07).** After creating the slices, set actual tracker links — do not leave a dependency living only in the `## Blocked by` section:
- **Blocked by / blocks** — for every "blocked by" relationship (a slice blocked by another slice, or by the `[Discuss]` ticket), create a real *is blocked by* link. On Jira use `createIssueLink` type `Blocks` (inwardIssue = the blocker, outwardIssue = the blocked slice); confirm the type name with `getIssueLinkTypes` if unsure.
- **Relates to** — slices that touch the same area but aren't a hard dependency get a *Relates* link so reviewers can navigate the set. If the tracker already groups them under the parent (Jira Epic children), that grouping suffices — skip redundant relates-to links.
- The `## Blocked by` section in the description is a human-readable mirror; the **real link is mandatory**. Include the links in the same confirm step as the tickets (linking is outward-facing).

<issue-template>
*(Generic fallback for non-Juzmatch trackers. For Jira project AUR use `<aur-card>` above; for CP use `to-aur-ticket`'s CP template.)*

## Parent

A reference to the parent overview issue created in step 7 (or an existing parent, if the source was one). Omit only for a single-slice feature.

## Phase

MVP / Phase 2+ (omit if no phased cut was made).

## What to build

A concise description of this vertical slice. Describe the end-to-end behaviour, not layer-by-layer implementation.

Avoid specific file paths or code snippets — they go stale fast. Exception: a prototype snippet that encodes a decision more precisely than prose (state machine, reducer, schema, type shape) — inline the decision-rich parts and note it came from a prototype.

**Screenshot (visual slices only).** If this slice has a visual surface (UI, layout, a flow clearer shown than told), add **one** placeholder for the creator to drop a real screenshot/mock:

`📷 [[ screenshot: target UI for this slice — paste or delete ]]`

If the slice **changes existing UI**, use a before/after pair instead:

`📷 [[ screenshot: current ]]` · `📷 [[ screenshot: target ]]`

Backend / data / infra-only slice → omit; no placeholder.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

## Open technical questions

Questions to resolve **with dev before building** (omit this section if none). While any remain open, this slice is HITL.

- [ ] Question 1 — and the functional constraint that bounds it, named in plain words (no bare decision IDs)
- [ ] Question 2

## Blocked by

- A reference to the blocking ticket or the `[Discuss]` ticket (if any)

Or "None — can start immediately" if no blockers.

</issue-template>

Do NOT close or modify a **pre-existing** parent issue the source referenced. (The overview issue you created in step 7 is yours to populate and link.)

## Design-drop intake (second entry point)

When the PM says a design has been delivered (e.g. "Oak ส่ง design ของ X มาแล้ว"), run this pass instead of a full breakdown:

1. Read the design against the designer-handoff screen inventory — every screen ID must map to a frame.
2. Draft the **UI stories** for the now-ready slices: behavior AC from the functional spec, UI AC from the delivered design, Figma links, screen IDs.
3. **Map and link each UI story to its technical ticket(s)**: real tracker links (`relates to`, or `is blocked by` when the tech ticket isn't done), plus tick off the `🎨 UI story to follow` marker on the tech ticket.
4. **Orphan check, both directions:** tech tickets still marked `🎨 UI story to follow` with no story after the drop → list them to the PM; screens/frames with no owning ticket → list them too (this is what caught AUR-194/195 two months late in WS-0).
5. Confirm with the PM before filing, as always.
