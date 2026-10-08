---
name: to-aur-ticket
description: >-
  Create Juzmatch Jira tickets in the Atlassian Cloud site
  juzmatch-tbi.atlassian.net via the Jira MCP, using the AUR card standard set
  by Aod (Dev Manager) in AUR-331 for project AUR, or the legacy CP template for
  project CP. Use whenever a PM wants to write, draft, file, or "make a ticket /
  story / task / bug" for a feature, a piece of dev work, or a defect — even if
  they just describe what they want built or the bug they saw without saying the
  word "ticket". The skill picks the right type, fills the matching template,
  attaches the right Epic, drafts for confirmation, then creates it and returns
  the link. Trigger on "create a ticket", "file a bug", "write a user story",
  "make a task for the team to…", "log this issue", or a pasted bug report /
  feature ask meant for the dev team.
---

# Juzmatch Jira ticket (AUR card standard)

Turn a PM's plain-language ask into a Jira ticket the dev team can act on
without asking follow-up questions. Since **02/10/2569** every ticket filed to
project **AUR** (Aurora) must follow **AUR-331** "[Template] Card template
มาตรฐานสำหรับ AUR" by Aod. Dev reads cards by these sections and expects
**values, not prose**. Project **CP** keeps the older User story / SOW / AC
template (kept at the end of this file).

> Shared skill. Install with `npx skills add juzmatch/ai-agent-management-public --skill to-aur-ticket`.
> It assumes the Atlassian MCP (Rovo) is connected in your Claude Code session.

## Workflow

1. **Understand the ask.** Read what the PM wrote. If it touches Juzmatch
   business logic (asset statuses, KYC, OVD, teams, financials, the RTO flow),
   pull authoritative detail from the `juzmatch-context` skill if available, or
   the project's knowledge base / confirmed spec. Never guess domain terms.
2. **Pick the project.** Aurora work → `AUR` (board 206). Legacy / BAU platform
   work → `CP`. Override only if the PM names another project.
3. **Pick the type.**

   | Type | Jira issue type | Use when the PM is describing… |
   |------|----------------|--------------------------------|
   | **Story** | `Story` | A business / user-journey outcome with a clear end user. |
   | **Task** | `Task` | A direct, concrete ask to the team (refactor, data, config, integration). |
   | **Bug** | `Bug` | Something broken: an as-is and an expected. |

   On AUR, Story and Task both use the **AUR card** below. Bugs keep the Bug
   template. If genuinely ambiguous, ask one short question; otherwise infer and
   state the choice in the draft.
4. **Find the Epic.** If the PM names a key, use it. Otherwise search with
   `searchJiraIssuesUsingJql`:
   `project = AUR AND issuetype = Epic AND statusCategory != Done ORDER BY created DESC`
   (swap `CP` for CP work), match by name, show the candidate and confirm. If
   none fits, ask whether to leave it unparented. Never invent an Epic.
5. **Draft the ticket** (summary + description) with the matching template.
   Run the tone checklist (below) on the body. **Show the draft in chat before
   creating anything.** Filing a ticket is outward-facing; always confirm first.
6. **Create it** with `createJiraIssue` (`contentFormat: "markdown"`, `parent`
   = Epic key). Wire any dependency as a real issue link (see Conventions).
7. **Return the link** `https://juzmatch-tbi.atlassian.net/browse/<KEY>` and
   note the type and Epic it was filed under.

## The AUR card (AUR-331) — 14 topics, 8 mandatory

| # | Topic | Required |
|---|-------|----------|
| 0 | Title `[System][Module] <สิ่งที่ทำ>` | ✅ |
| 1 | Header line: Phase · Blocked by (reason) · Related · ⚠️ update banner | when relevant |
| 2 | Background / ปัญหา — what happens today + table + current workaround | ✅ |
| 3 | Goal / Expected result — one measurable sentence | ✅ |
| 4 | User Story (As / I want to / So that) | features with a clear user only |
| 5 | Scope — In scope **and** Out of scope, always both | ✅ |
| 6 | Business Rules — table หัวข้อ → กติกา | ✅ |
| 7 | Role & Permission — table Role → เห็น → กด/แก้ได้ (+ API 403 note) | when roles differ |
| 8 | UI Changes / States / ข้อความ — verbatim strings | new or changed UI |
| 9 | Data model / Field mapping — Field → Type/Source → หมายเหตุ | new/changed field or enum |
| 10 | Acceptance Criteria — table given / when / then / Remark, success **and** fail | ✅ |
| 11 | Figma / Screenshot / QA URL — success node, fail node, QA URL | ✅ for UI work |
| 12 | Dev Open Questions — checkboxes + ไฟล์/service ที่คาดว่าต้องแก้ | ✅ |
| 13 | References / Note / Change log | optional |

### Card body template

```markdown
> **Phase:** MVP / 2 · **Blocked by:** AUR-… (เหตุผลสั้น ๆ) · **Related:** AUR-…
> ⚠️ (only after an edit) อัปเดต dd/mm/yyyy — ส่วนนี้ทับฉบับเดิม ประวัติอยู่ในคอมเมนต์

### Background / ปัญหา
ตอนนี้… (1–2 sentences: what happens today, who suffers, how often)

| กรณี / ช่องทาง | สิ่งที่เกิดตอนนี้ |
| --- | --- |
| <who wants what, with volume> | <manual step + time cost / wait> |

**Workaround ที่ใช้ตอนนี้:** …

### Goal
<one sentence, measurable: actor + action + result + constraint>

### User Story
**As** … **I want to** … **So that** …

### Scope
**In scope**
- …

**Out of scope**
- … (name the adjacent things people will ask for)

### Business Rules
| หัวข้อ | กติกา |
| --- | --- |
| <limit / format / PII / rounding / who-can> | <exact value QA can check> |

### Role & Permission
| Role | เห็น | กด/แก้ได้ |
| --- | --- | --- |
(API ต้องตอบ 403 สำหรับ role ที่ไม่มีสิทธิ์)

### UI Changes / States
- ปุ่ม: "…" · loading: "…" · success toast: "…" · error toast: "…" · disabled เมื่อ: …

### Data model / Field mapping
| Field | Type / Source | หมายเหตุ |
| --- | --- | --- |

---
### Acceptance Criteria
| given | when | then | Remark |
| --- | --- | --- | --- |
| <role + data state with real values> | <action> | <exact result: file name, count, time bound, text> | success / fail / 403 / PII |

### Figma / Screenshot
* success: <Figma node link> · fail: <Figma node link> · QA URL: …

### Dev Open Questions
- [ ] <real engineering choice, bounded by the functional rule in plain words>
- [ ] ไฟล์/service ที่คาดว่าต้องแก้: …

### References / Note
- Field dictionary · related cards · "ถ้าทีมอื่นขอเพิ่ม X ให้เปิดการ์ดใหม่"
```

Title: `[System][Module] <สิ่งที่ทำ>`, e.g. `[JMS][Agent] อนุมัติใบสมัคร agent จากหน้า BD review`.
The System/Module prefix is mandatory so the board filters by it.

Never fabricate or embed an image. Leave the Figma / screenshot link for the
PM to fill.

## How an AUR card must read (tone, not just headings)

Learned from Aod's worked example in the first comment of AUR-331 (a mock
Export-Excel card). Check every draft against this list before showing it.

- **Thai spec register.** No ครับ, no first person, no รบกวน / ขอความกรุณา /
  ฝากด้วย. English nouns for system and tech terms stay in English (filter,
  toast, endpoint, role, enum, stream, paginate). It is a spec, not a message.
- **Background is a quantified pain, not a feature description.** "copy มือ
  24 หน้า ใช้เวลา ~40 นาที", "รอ 1–2 วัน". Every row has a who, a what-happens-now
  and a cost. Use the table even for two rows. Name the current workaround in
  one line.
- **Goal is one measurable sentence.** "ผู้ใช้ OMS กด Export แล้วได้ไฟล์ .xlsx
  ตาม filter ที่เลือกอยู่บนหน้าจอ ภายใน 30 วินาที ไม่ต้องพึ่งทีม Data". Never
  "เพื่อเพิ่มประสิทธิภาพ".
- **Out of scope is specific.** "Export หน้าอื่น เช่น Deal, Buyer, Payment",
  "ส่งทางอีเมล", "CSV/PDF". Generic "อื่น ๆ" is not an out-of-scope. Add the note
  "ถ้าทีมอื่นขอเพิ่ม ให้เปิดการ์ดใหม่".
- **Rules are values.** 10,000 แถว · `assets_YYYYMMDD_HHmm.xlsx` · dd/mm/yyyy พ.ศ.
  · ทศนิยม 2 ตำแหน่ง · **ไม่ export** เลขบัตร / เบอร์ / อีเมล. If QA cannot check a
  row, it is not a rule yet.
- **UI strings verbatim**, in quotes, including error text. Nobody should have
  to ask what the toast says.
- **Data model names the real source path.** `asset.assetCode`,
  `deal.buyerFinalPrice`, with an example value and a dated note when a column
  was added later.
- **AC are scenarios with numbers.** "Role BD กรอง สถานะ = Active (1,200 รายการ)
  กด Export → ได้ไฟล์ `assets_20261005_1430.xlsx` 1,200 แถว ภายใน 30 วินาที".
  Fail rows carry the exact error text. Include the 403 row, the double-click
  row, and the PII-absence row where relevant. Table preferred; bullets of the
  same shape are accepted.
- **Dev Open Questions are engineering choices**, never "TBD": server-stream vs
  browser, reuse endpoint vs new. List the repos or services expected to change
  (e.g. `juzmatch-backend-php`, `juzmatch-api`).
- **No SOW steps.** The old "1. ทำหน้า… 2. ต่อ API…" list is gone. Rules + UI
  states + data model + AC replace it.
- **Where content comes from:** Background ← pain points in the workshop canvas
  or confirm deck · Business Rules + Out of scope ← the functional spec's
  resolved decisions and out-of-scope sections · AC values ← the field sheet
  and confirmed rules · Dev Open Questions ← the spec's Dev Open Questions.
- **Change discipline.** Edits after filing go as a ⚠️ banner line at the top
  with the date, with history in comments. Scope requests from other teams →
  new card, never widen the existing one.

### Anti-patterns (old CP habits that AUR cards must not have)
- Free-text SOW steps instead of rules + AC.
- AC without values ("ระบบต้อง export ได้ถูกต้อง").
- Missing Out of scope.
- Background that describes the feature instead of the current pain.
- Thai bureaucratic register (ดำเนินการ, เพื่อให้สามารถ…ได้) or English service-speak.
- Message register inside the card (ครับ, รบกวน, ฝากด้วย).
- Abbreviated roles (TL, SM). Spell out: หัวหน้าทีม, Sale Manager. Domain
  acronyms everyone knows (OTP, AMLO, PDPA, KYC) are fine.

## Bug template (AUR and CP)

```markdown
## Pre-Requisite
<setup, environment, or data needed to reproduce — links to sheets/records if any>

## Test Steps
1. <step to reproduce>
2. <…>

## Actual Result
<what happens now>

## Expected Result
<what should happen after the fix>

## Technical Note
-
```

## Conventions

- **Language.** Match the PM's input language for the summary. Card bodies are
  read by the Thai dev team, so Thai is the norm. UI copy or anything
  customer-facing inside a ticket is Thai-first.
- **🔒 PII.** Never put real customer PII (names, phone numbers, 13-digit
  national IDs, addresses) into a ticket. Use placeholders; mask IDs as
  `*********1234`. If the raw text contains real PII, flag it and scrub it in
  the draft before creating.
- **Formatting.** Currency `฿1,234,567`; dates `DD/MM/YYYY` Buddhist Era
  (พ.ศ.) for anything user-facing.
- **Address fields** (Aurora): always the 12 split parts, never a free-text
  address box. See the Aurora knowledge base §12.5 for the list.
- **Don't over-ask.** Infer type, tag, rules, AC and Epic. Surface the
  inferences in the draft for the PM to correct. One confirm step, not an
  interrogation.
- **Issue links are real links, not prose** (dev-team request, Peng 14/07).
  - *Blocked by / blocks*: work that cannot start until another ticket is done
    gets a real *is blocked by* link. Never leave "Blocked by: AUR-18" only in
    the description.
  - *Relates to*: tickets in the same flow or feature relate to each other or
    the umbrella story. If they already share a parent Epic, that is enough.
  - Tool: `getIssueLinkTypes` for the exact type name, then `createIssueLink`.
    Linking is outward-facing, so include it in the same confirm step.

## Reference

- **Site / cloudId:** `juzmatch-tbi.atlassian.net` (the MCP tools accept the
  site URL as `cloudId`).
- **Projects:** `AUR` = Aurora (board 206; WS-0 umbrella AUR-17). `CP` = Core
  Platform for legacy / BAU work.
- **Issue type names** (`issueTypeName`): `Story`, `Task`, `Bug`.
- **Epic linking:** set the Epic as `parent` on `createJiraIssue`. If rejected,
  fall back to `editJiraIssue` setting `parent`, or `createIssueLink`.
- **Jira MCP tools** (names may carry a server prefix in your session):
  `createJiraIssue`, `searchJiraIssuesUsingJql`, `getJiraIssue`,
  `getJiraProjectIssueTypesMetadata`, `editJiraIssue`, `createIssueLink`,
  `getIssueLinkTypes`.
- **Worked example:** read AUR-331 and its first comment with `getJiraIssue`
  when in doubt about tone.

## Legacy CP templates (project CP only)

### Story
```markdown
## User story
As a **<role>**, I want **<goal>** so that **<benefit>**.

<1–3 sentences of business context. No implementation detail.>

## AC
- <observable, business-level acceptance criteria>

## Technical Note
-
```

### Task
```markdown
## User story
<1–2 sentences: what is wanted and why.>

## SOW
1. <concrete step the dev should do>
2. <…>
- Reference any spec / sheet / Confluence link provided.

## AC
- [ ] <testable condition that means "done">
- [ ] <error / edge handling if relevant>

## Technical Note
-
```

CP summaries carry a bracketed area tag, e.g. `[e-kyc] …`, `[OCR-Contract] …`.
Never file a CP Task with empty SOW or AC.
