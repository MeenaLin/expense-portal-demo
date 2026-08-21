# BV Workplace — Expense Reimbursement Workflow Demo

A standalone, single-file interactive prototype of a Digital Workplace Portal, built around a complete
**Expense Reimbursement Workflow**: Requester → Manager → General Manager (conditional) → Finance Reviewer
→ Cashier → Paid / Closed.

This package contains everything needed to review the assignment without any development environment:

- `index.html` — the entire Portal. Double-click it (or open it in Chrome/Edge) and it runs. No server,
  no build step, no internet connection required.
- `README.md` — this file (setup, accounts, architecture, trade-offs, test evidence).

---

## 1. Setup / how to run

1. Open `index.html` directly in the current version of **Chrome or Edge** (double-click, or drag into a
   browser window). Everything — UI, business logic, seed data — is in this one file.
2. No installation, no `npm install`, no network access is required. Fonts are loaded from Google Fonts
   as a progressive enhancement only; the layout and every function work identically offline with the
   built-in system-font fallback.
3. State is saved to `localStorage` after every action, so it survives a page reload in the same browser
   (per the assignment's "can persist Demo state during a User Session" requirement and the spec's
   Trade-off 3). Use **Reset Cases** (Expense Workspace) at any time to deterministically restore Cases
   A–D and overwrite the saved snapshot.
4. To host it as a "Live Demo URL," upload `index.html` as-is to any static host (GitHub Pages, Netlify
   drop, Azure Static Web Apps, or a SharePoint document library) — it needs no backend.

## 2. Portal sign-in & demo / test accounts

Opening `index.html` always lands on a **Portal sign-in page** first (this gate is never skipped or
remembered across reloads, by design). Instead of a free-text account field, the sign-in page shows a
**dropdown of the currently active identities** — pick one and enter its demo password:

| Field | Value |
|---|---|
| Identity | Choose from the dropdown, e.g. "Alex Chen — Requester" |
| Password | The identity's first name, lowercase (see table below) |

Both fields are required (native HTML5 validation); an incorrect combination shows an inline error. Once
signed in, the account avatar (top-right of the banner) shows the signed-in identity and opens a merged
menu — styled after the Microsoft Entra admin center pattern — with **My Account** (a visual-only profile
screen for whichever identity is currently signed in), **Switch to another user** (signs out and returns
to the sign-in page so a different identity can log in), and **Sign out**.

There is no separate "admin" account layered on top of the five workflow identities — signing in *is*
choosing which of the five you're acting as. Switching identity therefore always goes through the sign-in
screen: click the avatar → **Switch to another user** → pick a different identity from the list → enter
its password → sign in. The "Log in as this identity" shortcuts on **My Work** / **Demo Guide** pre-select
an identity on the sign-in screen as a convenience, but a password is still required to complete the
switch.

| Account | Name | Role | Demo password | Minimum demo permission |
|---|---|---|---|---|
| 1 | Alex Chen | Requester | `alex` | Create requests, view own requests, fix & resubmit returned items |
| 2 | Jordan Lee | Manager | `jordan` | View and act on the Manager queue only |
| 3 | Morgan Patel | General Manager | `morgan` | View and act on the GM queue only |
| 4 | Riley Nakamura | Finance Reviewer | `riley` | Approve / Reject / Return; policy & accounting review |
| 5 | Casey Romero | Cashier | `casey` | Record payment after Finance approval; cannot edit approved amount |

Every stage is a **distinct named identity** — there is no single generic "Approver" account, satisfying
the role-separation requirement. Deactivating an identity from Administration → Approval Matrix removes it
from the sign-in dropdown immediately (and signs it out if it was the active session) — if every identity
is ever deactivated at once, the sign-in page offers a **Reset Cases** escape hatch so the demo is never
permanently locked out.

## 2a. UI language & branding

- **EN / 中文 toggle** in the top banner switches every label, button, table header, notice and the
  Architecture/Demo Guide copy. Switching to 中文 forces the interface font to **Microsoft JhengHei**
  (falling back to PMingLiU/Heiti TC/sans-serif if unavailable on the OS); switching to English forces
  **Arial**. Data fields (Request IDs, amounts, timestamps) keep a monospace treatment in both languages
  for legibility — a deliberate, disclosed exception to the interface-font rule.
- **Sidebar branding**: white background, brand teal `#3BBCB3` for the navigation text/icons and active
  state, with a subtle border/shadow separating it from the content area, per the visual spec.
  Administration sits as the last nav item, directly above the sidebar footer.
- **Login page** is centered on the full viewport width (not just within the sidebar-less shell), on both
  desktop and narrow layouts.

## 3. Technology choices

- **Vanilla HTML / CSS / JavaScript**, one file, no framework and no build pipeline — chosen specifically
  so the "offline backup package" requirement (a single HTML file a reviewer can open with zero setup) is
  met by construction, not as an afterthought.
- State is a plain in-memory JS object rendered with template-string views and one delegated click
  listener (no virtual DOM, no external UI kit) — easy to read/modify live in an interview if asked.
- **LocalStorage persistence** (per the spec's Trade-off 3): all requests, audit history, configuration,
  notifications and system events are saved to `window.localStorage` after every state change, and
  restored automatically the next time the same browser opens the file — satisfying the "no dev
  environment needed to open" delivery requirement while still surviving a page reload. If LocalStorage
  is unavailable (e.g. a locked-down preview frame), the app detects this and falls back to in-memory-only
  operation automatically, with a small notice in the sidebar footer.
- **Font Awesome 6** (CDN) supplies the member sign-in/out icon (<i class="fa-solid fa-user"></i>) in the
  top banner — a progressive enhancement like the Google Font; the button still works if the CDN is
  unreachable offline, it simply loses the glyph.

## 4. Implemented scope

- Full Expense Reimbursement state machine: **Draft → Submitted → Pending Manager → Pending GM (Level
  2/3 only) → Pending Finance → Pending Payment → Paid / Closed**, plus **Reject** and **Return** as two
  distinct statuses (not merged into one). Reject always sends the request back to the Requester for
  correction; Return (Finance only) can target either the Requester or an earlier approver directly (see
  below). Both resume at the correct stage on Resubmit.
- **Role-scoped Expense Workspace**: the request list is filtered to what's actually relevant to the
  signed-in identity — a Requester sees only their own requests; an approver role (Manager/GM/Finance/
  Cashier) sees only requests whose computed route actually includes that role, so e.g. GM never sees a
  Level 1 request's workspace entry, matching what GM would never receive as a task in the first place.
- **Approval Route stepper**: five stages — Requester → Manager → GM → Finance → Cashier — with the old
  separate "Paid" step merged into the Cashier step (it pulses as current while awaiting payment and
  turns into a checkmark once actually paid), since processing payment is Cashier's job, not a separate
  stage.
- **Configuration-driven routing, with FX conversion**: USD/JPY amounts are converted to TWD (static
  demo exchange rates — not a live feed) before the Level 1/2/3 thresholds are evaluated, so a request's
  route is always judged on its TWD-equivalent value. Level 1 ≤ NT$5,000 (GM auto-skipped), Level 2
  NT$5,001–20,000 (GM required), Level 3 > NT$20,000 (GM required + auto High-Value flag). Thresholds are
  centrally editable in Administration and apply to new submissions only.
- **Save as Draft, with full re-editing**: a new request can be saved without entering the approval
  flow. From the draft's detail page, the owner sees an **Edit** button that reopens the same form,
  fully pre-filled, where they can either save the changes back as a draft again or submit it for
  approval (route/FX conversion is computed at the moment of actual submission, not when the draft was
  first saved).
- **Fix & Resubmit opens the full edit form**: a Rejected/Returned request's "Fix & Resubmit" button pops
  up the same "Edit Expense Request" form used for drafts — fully pre-filled, with the rejection/return
  reason shown for context — rather than a bare comment box. Every field can be corrected, not just the
  attachment; the primary button reads **Fix & Resubmit** instead of Submit in this mode, and a
  resubmission note is required.
- **New Expense Request validation**: all required fields are marked with a red asterisk; the Amount
  field rejects zero, negative, and non-numeric input in real time (red border + red inline message).
  Receipt handling is asymmetric by design: choosing **"No receipt available"** never triggers any
  file-related check — the form submits normally with no missing-receipt error. Choosing **"attach a
  receipt"** requires an actual file upload (PDF/JPEG/PNG/DOCX) before Submit succeeds; when editing a
  draft or a returned request that already has a file on record, re-uploading is optional (the existing
  file is kept unless replaced). Only a Requester identity can open the form — it is disabled/read-only
  for every other role.
- Five role-scoped, named demo identities. Switching identity always goes through the **sign-in screen**
  (see §2) rather than an instant in-app switcher, and role-aware "My Work" queues reflect whichever
  identity is currently signed in. On the My Work page itself, only the currently active identity's card
  is shown as active — the other four are greyed out and point to the account menu's "Switch to another
  user," so there's a single, consistent way to change identity.
- **My Work → View** opens the request in a modal dialog rather than navigating away from the queue, so
  reviewers stay in context.
- **Flexible Return routing**: Finance's Return action lets the reviewer choose the recipient — the
  Requester (for correction) or any earlier approver in the route (e.g. Finance can return a request
  directly to the Manager for re-review, without forcing a full Requester correction cycle). Approve,
  Reject, Return and Resubmit all require a comment.
- Append-only Approval History per request (Actor, Role, Action, Timestamp, Comment, Resulting Status)
  with per-request and full-log export (JSON / CSV via in-browser download).
- Notifications tied to real state transitions (new approval, return/reject, payment).
- **Administration**: editable routing thresholds; a fully managed Approval Matrix (Role, User Name,
  Entra Group, Active toggle) where every row can be activated/deactivated and new role assignments can
  be added on the fly via **+ Add New Role** (with the Entra Group auto-derived from the selected Role);
  Workflow Health indicators; a Handled-Exception simulation (Correlation ID, Owner alert, idempotent
  Retry); and a System Events log. Deactivating an identity immediately removes it from every "Switch
  Identity" list across the Portal (and force-switches you away from it if it was the active identity).
- Architecture page covering Components, **Data Model (six controlled lists)**, State Machine,
  Permissions, Exceptions, Deployment/Operations, Trade-offs, and a Demo-vs-Production gap list.
- One-click **Reset Cases** (stays on the Expense Workspace), deterministically reseeding the historical
  Paid example plus Cases A–D.
- Responsive layout (collapsing sidebar under ~900px), keyboard-operable native buttons/inputs, visible
  focus outlines, semantic headings and tables.

## 5. Known limitations (Demo scope, not Production)

- The Portal sign-in is a demo convenience, not real authentication — each identity's "password" is a
  predictable, documented convention (first name, lowercase) checked entirely in the browser, not against
  Entra ID (see Architecture → Trade-offs).
- No real virus scanning of uploaded receipt files — the file is accepted and its name is recorded, but
  its bytes are never actually stored or transmitted anywhere in this Demo.
- FX rates used for currency routing are static demo constants, not a live feed (documented in
  Administration and Architecture → Data Model).
- A draft can be saved with any subset of fields filled in; the Edit screen lets the requester complete
  or change anything before either re-saving as a draft or submitting, but there is no field-level
  "diff" or version history for draft edits — only the final saved state is kept.
- All authorization is enforced in this file's `canAct()` function; there is no server to enforce it
  independently, so the Architecture page explicitly calls out where production enforcement must move.
- Data persists to `localStorage` (see §3), but the sign-in state itself intentionally does **not** —
  every fresh page load returns to the sign-in screen even though the workflow data is restored.

## 6. AI-tool disclosure

This prototype (application code, UI copy, and this document) was drafted with the assistance of
**Claude** (Anthropic), based directly on the assignment brief. AI assistance was used for: scaffolding
the single-file app structure, generating the CSS design system, drafting the workflow/state-machine
logic, and writing the Architecture/Trade-offs copy. All resulting design decisions, security assumptions,
and code were reviewed against the assignment's checklist before delivery, and the author remains
responsible for explaining or modifying any part of it live.

## 7. Architecture summary

*(Full detail — with a component diagram, permission model, and exception-handling design — is inside the
Portal itself under **Architecture**, in eight tabs: Component Diagram, Data Model, State Machine,
Permissions, Exceptions, Deployment & Ops, Trade-offs, and Demo-vs-Production.)*

**Target Microsoft 365 mapping**

| Layer | Production service | What it owns |
|---|---|---|
| Experience | SharePoint Portal (or equivalent role-aware, task-oriented UI) | Presentation only — no business-rule truth |
| Data | Microsoft Lists / Dataverse | Expense Requests, Approval History, Approval Matrix, Workflow Configuration |
| Process | Power Automate | Routing, approvals, notifications, exception scopes, safe retry |
| Identity | Entra ID Groups | Least privilege, separation of duties |
| Operations | Service-owned connections | Monitoring, support ownership, ALM, environment separation |

**Data model (six controlled lists, field names as specified)**

| List | Key fields |
|---|---|
| 1. Expense Requests | Request_ID, Requester, Department, Expense_Date, Expense_Type, Amount, Currency, Purpose, Project_Cost_Center, Payee, Attachment_Status (Boolean), Route_Level, Status, Payment_Reference, Is_Locked (Boolean) |
| 2. Approval History (append-only) | History_ID, Request_ID, Actor_Name, Role, Action (Submit/Approve/Reject/Return/Resubmit/Payment), Timestamp, Comment_Reason, Resulting_Status |
| 3. Workflow Configuration | Config_Key (e.g. `Level_1_Threshold`), Config_Value (e.g. `5000`) |
| 4. Approval Matrix | Role_Name (Manager/GM/Finance/Cashier), Assigned_Identity |
| 5. User Profile | User_ID, Full_Name, Photo_URL, Birthday, Country_Region, Language, Regional_Format, Email, Phone — backs the "My Account" screen |
| 6. Member Account | Account_ID, Account, Password_Hash (masked — plaintext is never stored), Role (drawn from the same enum as Table 2's Role field, assignable from Administration → Approval Matrix), Is_Active (defaults true; deactivating an account removes it from the sign-in dropdown and from every "Switch Identity" surface — managed live from Administration → Approval Matrix) |

Workflow Configuration is read once at submission and stamped onto the request's Route_Level, so later
threshold changes never retroactively re-route an in-flight request.

**Why hiding a button is not security** — the Portal hides unauthorized actions in the UI for clarity, but
every transition is re-validated by a single authorization function (`canAct`) before it is applied — the
same shape of check a Power Automate "Authorization" scope would run server-side against Entra ID group
membership and Dataverse row/column security. In production this check must live in the Data/Process
layer, never only in client-side JavaScript.

**Exception handling** — Try/Catch scopes per flow action, a Correlation ID per run, a dedicated Exception
Record (reason, last valid state, timestamp), Flow Owner notification (not the business approver), and
idempotent retry keyed on Request ID + intended transition so a retried action can never duplicate an
approval, a notification, or a payment.

**Trade-offs (see Architecture → Trade-offs for full risk/next-step detail; bilingual in the Portal)**

1. **Simulated Identity vs. Real Entra Authentication** — a per-identity sign-in screen (pick an
   identity, enter its demo password) stands in for real authentication. Risk: the "password" is a
   predictable, documented demo convention, not a real credential — a malicious user could still guess it
   or call the API directly. Next step: remove the demo password scheme, read the real Entra ID
   credential, enforce Item-level Permissions at the data layer.
2. **Rapid Prototyping vs. Centralized Relational Data** — Microsoft Lists (flat, NoSQL-like tables) are
   the fastest path to an interactive UI. Risk: poor performance/scalability for complex joins (e.g.
   multi-line expense items). Next step: migrate to Dataverse or Azure SQL Database when extending to
   multi-line forms such as Purchase.
3. **Local Browser State vs. Persistent Cloud Data** — this build persists to `localStorage` to satisfy
   the standalone-HTML delivery requirement. Risk: data can be lost if browser data is cleared, and it
   cannot demonstrate true multi-user collaborative approval. Next step: replace `localStorage` with a
   real cloud API (Microsoft Graph API or a custom backend REST API) before production.

## 8. Test evidence matrix

| Case | Setup | Expected proof | Result |
|---|---|---|---|
| **A** | NT$3,000, receipt complete | Manager approves → routes directly to Finance; GM never receives a task; Finance approves; Cashier pays; final state Paid/Closed. | ✅ Verified — `computeRoute` returns Level 1 with stages `[Manager, Finance, Cashier]`; GM is structurally excluded from the route, not merely hidden. |
| **B** | NT$12,000, receipt complete | Manager, GM, Finance, Cashier each act from a distinct named identity; all four appear in Approval History. | ✅ Verified — Level 2 route includes GM; each transition is logged with the acting identity's name and role. |
| **C** | NT$50,000, receipt complete | Level 3 + High-Value auto-flagged; full route enforced; Cashier cannot edit approved amount; record locks after payment. | ✅ Verified — `highValue` flag set automatically above the Level 2 threshold; the payment form exposes only Payment Reference/Date fields, never Amount; `locked=true` after payment disables all further actions. |
| **D** | NT$8,000, missing receipt | Manager rejects with a reason; Requester attaches evidence and resubmits; original rejection/comment preserved; flow resumes at the correct stage. | ✅ Verified — Reject requires a non-empty comment; the rejection event is appended (never overwritten); Resubmit sets `currentStage` back to `returnedFrom` (Manager), not to Draft. |
| **New E2E** | Any new request created via **+ New Expense** as Alex Chen (Requester) | Route is shown before submit; each stage's queue gains/loses the task correctly; payment completes with reference, Paid/Closed status, locked amount, and full audit history. | ✅ Verified — `buildRequest()` computes and displays the route pre-submission; each `applyAction` call both updates `status`/`currentStage` and fires the matching queue notification, confirmed by walking a fresh request through all five identities. |
| **FX routing** | New request, 200 USD (≈ NT$6,300 at the demo rate) | Routes as Level 2 (GM required), not Level 1, because the TWD-equivalent amount is used for the threshold check, not the raw USD figure. | ✅ Verified — `buildRequest`/`applyAction('submit-draft')` both call `toTWD()` before `computeRoute()`; the on-screen route preview updates live as currency/amount change. |
| **Flexible Return** | Any Level 2/3 request at Pending Finance | Finance's Return action offers Manager (and GM, if applicable) as recipients, not only the Requester; choosing Manager sends the request straight back to `PendingManager` without a Requester correction step. | ✅ Verified — the Return modal's target dropdown is built from `priorStagesOf(req)`; `applyAction('return', {target})` sets `currentStage`/`status` directly for a non-Requester target. |
| **Draft** | Save a new request as Draft, then open it and Submit for approval | Draft appears in the Expense Workspace with a Draft badge and no route; submitting computes the route (with FX conversion) and enters Pending Manager. | ✅ Verified — `buildDraftRequest()` produces `status:'Draft'`, `routeLevel:null`; `submit-draft` stamps the route and pushes a `Submit` history event. |
| **Draft edit** | Save a Draft, reopen it via **Edit**, change the amount/currency/purpose, then Save as Draft again | The form opens fully pre-filled with the draft's current values; saving updates the same request in place (same Request ID) rather than creating a duplicate. | ✅ Verified — `edit-request` loads the request into `state.editingRequestId`; `handleSaveDraftExpense`'s edit branch mutates the existing request object instead of calling `buildDraftRequest()`. |
| **Receipt validation asymmetry** | (a) New request, select "No receipt available", Submit. (b) New request, select "attach a receipt", leave the file empty, Submit. | (a) Submits successfully with no file-related error. (b) Submit is blocked with a red "please choose a file" message. | ✅ Verified — `resolveAttachment()` short-circuits to `{missing:true}` for case (a) before any file check runs; for case (b) it returns `null`, which `handleSubmitNewExpense` treats as a hard block. |
| **Approval Matrix governance** | In Administration → Approval Matrix, deactivate the current identity's row, then click **+ Add New Role** and fill in a new Manager | The deactivated identity disappears from every Switch Identity list immediately and the active identity auto-falls-back to another active one; the new row's Entra Group auto-fills from the chosen Role and the new identity becomes selectable as soon as a name is entered. | ✅ Verified — `handleToggleUserActive()` re-points `state.currentUserId` to `activeUsers()[0]` when the active identity is turned off; `handleAddNewRole()` pushes a new `custom:true` entry that `entraGroupForRole()` and `activeUsers()` both handle like any other identity. |
| **Reject vs. Return statuses** | (a) Case D: Manager Rejects. (b) A Level 2/3 request: Finance Returns to the Requester | (a) Status badge and filter both read "Rejected", not "Returned". (b) Status badge and filter both read "Returned". Both still appear in the Requester's My Work queue and can be Fix & Resubmit'd. | ✅ Verified — `applyAction('reject', …)` sets `status:'Rejected'`; `applyAction('return', {target:'Requester'})` sets `status:'Returned'`; `canAct`/`requestsForRole`/`statusBadge` all branch on both values. |
| **Merged stepper step** | Any request, walked from submission through payment | The Approval Route shows 5 nodes (Requester/Manager/GM/Finance/Cashier); the Cashier node pulses as current while Pending Payment and turns into a checkmark once Paid — there is no separate 6th "Paid" node. | ✅ Verified — `renderStepper()`'s `order` array has 5 entries; `isPaid` marks the final Cashier node `done` directly rather than indexing a removed `'Paid'` entry. |
| **Role-scoped Expense Workspace** | Log in as Morgan Patel (GM), open Expense Workspace, with a Level 1 request (e.g. Case A) present | Case A does not appear in GM's Expense Workspace list or counts, because GM is not part of a Level 1 route; a Level 2/3 case does appear. | ✅ Verified — `visibleRequestsForRole()` filters to `r.stages.includes(u.role)` for approver roles, and to `r.requesterId===u.id` for the Requester role. |
| **Login-gated identity switching** | Click the account avatar → **Switch to another user**, pick a different identity, enter its demo password | Redirects to the sign-in screen with an identity dropdown (not a free-text account field) pre-selecting the current identity; entering the wrong password shows an error and does not switch; the correct password signs in as the newly chosen identity. | ✅ Verified — `handleGotoLoginAs()` sets `authenticated:false` and `loginSelectedId`; `handleLogin()` validates the selected identity's password via `demoPasswordFor()` before setting `authenticated:true` and `currentUserId`. |

All four cases are recreated deterministically by **Reset Cases** (which now keeps you on the Expense
Workspace), and can be run in any order — each depends only on its own request's state, never on prior
demo history.
