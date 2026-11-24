# FixVisit Report

**Property-manager-ready proof of work for payment approval.** Tagline:
**"Document your repairs. Submit with confidence. Get paid."** Descriptive
tagline (shown under the name in the app and on the landing page):
**"Property Maintenance Visit Reports."**

FixVisit Report is built for handymen and maintenance contractors who
service rental properties for landlords and property-management companies —
the customers who require documented proof of work (work order number,
unit, before/after photos, itemized charges) *before* they approve
payment. The paying customer is not paying for a PDF; they are paying to
make documenting, submitting, and retrieving evidence of completed
maintenance work easier. The tool also works for homeowner jobs, but every
design decision leads with the property-management workflow.

After each visit, the contractor sends one document: what was found, what
was fixed, before/after photos, issues that need a return trip, and an
invoice-ready charges table with the amount due. The moment the report
matters most is the moment of payment approval — so the amount due,
payment terms, and payment tracking are first-class parts of the product,
not afterthoughts.

FixVisit Report is deliberately **not** a scheduling, dispatch, or CRM
platform, and it does not try to replace the software a contractor already
runs their business on (no Jobber-replacement claims anywhere).

Pricing: **$19/month founding-member pricing** for the first 25 customers,
locked in while the subscription stays active. Standard price is
**$29/month** after the founding spots are gone. The visit builder and
completion summary are free, and every free user gets **one complete
branded PDF report free** (no credit card, no watermark — see "First
report free" below). Pro unlocks unlimited PDF reports, an evidence pack
for every saved visit, the visit history dashboard with payment tracking,
CSV export, and backup export/import.

Plain static HTML/CSS/JS — no build step, no backend.

## Files

| File | What it is |
|---|---|
| `index.html` | Landing page: promise, how it works, free/Pro features, a static sample report (a rental-property visit for a property manager), founding pricing, privacy + billing/cancellation notes. |
| `app.html` | The tool: visit record (client, property, work order / unit / PO / invoice references, times, type), client & property profiles, work items with statuses and before/after photos (IndexedDB), issues-found log, parts & charges with live amount due, payment terms + acknowledgment modes, branded PDF report (jsPDF via CDN), evidence pack with SHA-256 checksums (JSZip via CDN), visit history dashboard with payment tracking + CSV export, backup export/import. |
| `README.md` | This file. |

## Before launch

- In `index.html` and `app.html`, replace the `STRIPE_LINK` placeholder with
  the real Stripe subscription payment link for FixVisit Report ($19/month
  founding price). There are two in `index.html` (hero + pricing) and one in
  `app.html` (the `STRIPE_URL` constant, which is assigned to the Pro gate's
  subscribe button on load — the constant's *name* deliberately avoids the
  placeholder text so a find-and-replace can't mangle it).
- Contact email in the footer is `chrisla122787@gmail.com`.

## Free vs Pro

**Free (no payment, no account):**

- The full visit builder for the visit you're on: client and property,
  work order #, unit #, PO #, invoice reference, visit date and type,
  arrival/departure times, work items, issues, photos, parts & charges —
  plus a live summary showing **documentation status** and **repair
  status** separately, and a watermarked live preview of the report.
- Client & property profiles (save your regulars once, autofill every
  return visit) and property-manager mode (see below).
- **First report free:** one complete branded PDF visit report — with
  photos, issues, and the charges table, no watermark, no credit card. See
  "First report free" below.

**Pro (subscription):**

- Unlimited branded PDF visit reports: company name/logo, client/property,
  the reference line (work order #, unit, PO #, invoice #), a unique
  Report ID, date, visit type, time on site, **documentation status and
  repair status stated separately**, work items with statuses and
  before/after photos, issues with severity and recommendations, a
  "Recommended next steps" section, the charges table with the amount due
  and payment terms, and an acknowledgment block that matches how the
  report is signed off (see "Acknowledgment modes").
- **Evidence pack (ZIP) per saved visit:** the report PDF, the ORIGINAL
  work-item photos (`Work_Item_Photos/`, named per item, Before/After),
  `Visit_Record.json` (the full visit record), `Report_Manifest.txt`
  (report ID, references, both statuses, amount due, payment status, file
  list, and the SHA-256 checksum of every content file), and
  `SHA256_Checksums.txt` (checksums for every file in the pack except
  itself, including the manifest). Built entirely in the browser (JSZip
  via CDN). Once exported, the saved visit is flagged `evidenceExported`
  and the history list shows "Evidence pack exported ✓". The pack is the
  user's own backup — FixVisit Report never stores a copy, and the UI
  says so in plain words.
  - **Honest limits, printed in the manifest itself:** SHA-256 hashes
    show whether exported files changed *after* export. They do **not**
    prove when a photo was taken or that the work was performed.
- Visit history dashboard: saved visits with date, documentation status,
  repair status, references, and amount due; totals for visits this
  month, amount billed this month, visits all time, amount billed all
  time, and **amount awaiting payment**; per-visit **payment status**
  tracking (Unbilled / Submitted / Approved / Paid); **CSV export** of
  the full history; and **Start similar visit** (see below).
- Backup export (JSON, photos and profiles included) / import-restore.

Photos are deliberately **not** Pro-gated in the builder (the same choice
SurfaceProof Ledger made): a visit report without photos is meaningless,
so free users attach photos freely. Pro's value is unlimited finished
reports, evidence packs, history, and backup — not holding the camera
hostage.

## First report free

Free-tier users can generate **one** complete branded PDF report (the same
report Pro users get — photos, issues, and charges included, no
watermark). It is tracked by a localStorage counter
(`fixvisit.freereport.v1`): the first successful PDF download consumes it,
after which the PDF button asks for Pro. This is best-effort only — there
is no registration or server check, so clearing browser data resets the
counter. That is accepted at MVP: the free report is a sampling device,
not a security boundary.

## ⚠️ MVP access note (read before launch)

Pro unlocking is **client-side only** for launch validation:

- The "I already subscribed — unlock Pro" button in `app.html` simply sets
  a `localStorage` flag (`fixvisit.pro.v1 = "1"`) in the visitor's own
  browser. There is **no real payment verification** — no server checks the
  Stripe subscription, and anyone who finds the button can unlock Pro
  without paying.
- This is a deliberate MVP trade-off to validate demand cheaply on a $0
  static stack (the landing page and the app say so in plain words:
  unlocking is on the honor system during opening validation).
- Before scaling past validation, replace this with a real entitlement
  check (e.g. a small serverless function that verifies the Stripe
  subscription by email before issuing an unlock token).

## The visit logic

All core logic is in pure functions in `app.html`:

- **Work items** — `{ title, description, status, beforeIds, afterIds }`.
  Status starts **unset** (`""`) and must be set to one of *Completed*,
  *Partially completed*, *Diagnosed only*, or *Needs return visit*. An
  unset status is exactly what keeps documentation INCOMPLETE — the
  handyman has to make a call on every item.
- **Time on site** — `timeOnSiteMinutes(state)` parses the arrival and
  departure `HH:MM` values. Departure minus arrival; a departure *earlier*
  than the arrival is read as crossing midnight (an overnight visit) and
  gets +24h. Equal times, or either time missing, yields no time on site.
  Example: 09:15 → 11:45 = **150 minutes = 2.5 hrs**.
- **Charges** — `charges(state)`:
  - parts total = Σ (qty × unit price) over the parts lines,
  - labor total = labor hours × labor rate,
  - subtotal = parts + labor + trip/call-out fee,
  - tax = subtotal × tax % (optional, off by default),
  - **amount due = subtotal + tax**, every figure rounded to cents.
  - **There is no hidden margin in this tool** — deliberately different
    from the portfolio's pricing tools (JanitorMargin, ScopeAmend,
    DetailBid). The labor rate and hours are the handyman's *own charges
    to the client*, so they are printed on the client-facing report along
    with everything else in the charges table.
- **Issues found** — `{ item, severity (Low/Medium/High), recommendation,
  needsReturn }`. Issues never block the documentation COMPLETE status;
  they are findings, not paperwork.
- **Recommended next steps** — `nextSteps(state)` rolls up every work item
  whose status is *Needs return visit* and every issue flagged
  `needsReturn`, in that order, as `{ title, note }` entries (the note is
  the item's description or the issue's recommendation). The section
  appears in the preview, the PDF, and the manifest whenever it's
  non-empty.
- **Reference fields** — `workOrder`, `unit`, `poNumber`, `invoiceRef`
  live on the visit record. `refsLine(state)` renders the non-empty ones
  as one line ("Work order #: WO-2481 · Unit: 4B · PO #: … · Invoice #:
  …"). The line appears in the builder preview, the PDF header block, the
  history list, the evidence-pack manifest, and the CSV export (as
  separate columns there). The PDF and manifest also carry the record's
  unique **Report ID**.
- **Payment terms** — a mode, not free text: `due` ("Due on receipt", the
  default), `net15` ("Net 15"), `net30` ("Net 30"), or `custom` (the
  contractor's own wording; blank custom text falls back to "Due on
  receipt"). The resolved text is what prints next to the amount due.
  Records saved before terms became a mode stored only the printed text;
  the mode is inferred back from it on load ("Net 15" → `net15`, any
  other non-default text → `custom`).
- **Acknowledgment modes** — `ackMode` on the visit record:
  - `onsite` (default): "Customer acknowledgment" block — confirm text
    plus signature and printed-name lines.
  - `manager`: "Property manager acknowledgment" block — confirm text
    stating the report is approved for payment processing, plus
    property-manager signature and printed-name lines.
  - `remote`: **no signature block.** The PDF instead prints a "Submitted
    for approval" section: the report was submitted electronically, no
    signature was collected on site, and the amount due becomes payable
    on approval under the payment terms.
- **Payment tracking** — every saved visit carries a `payStatus`:
  *Unbilled* (default), *Submitted*, *Approved*, or *Paid*, changed from
  a dropdown in the history list. `historyTotals` sums **awaiting
  payment** = the amount due of every visit whose status is Submitted or
  Approved (money earned and invoiced, not yet in hand), shown on the
  dashboard next to the billed totals.

### The two statuses (read this before touching the status code)

A report carries **two separate statuses**, in the builder summary, the
preview, the PDF, the history list, the manifest, and the CSV. They
answer different questions and must never be merged:

- **Documentation status** — `computeComplete(state)`: is the *paperwork*
  complete? COMPLETE only when **all** of these hold:
  1. there is at least one work item,
  2. **every** work item has a status set, and
  3. arrival and departure produce a real time on site,
  4. **and, in property-manager mode** (see below), every *Completed*
     work item has at least one photo (before or after).
  The summary shows the supporting counts alongside: work items with a
  status (X of Y), work items completed, issues found (and how many need
  a return visit), photos attached, and the amount due. When the status
  is INCOMPLETE, the summary panel names exactly what is missing —
  including, in property-manager mode, which completed items still need
  a photo.
- **Repair status** — `repairStatus(state)`: is the *work* finished?
  Derived only from work-item statuses, in this order:
  - no work items → **"No work items recorded"**
  - no item has a status yet → **"Not yet determined"**
  - every item *Completed* → **"All work completed"**
  - any item *Needs return visit* or *Partially completed* →
    **"Return visit required"**
  - every item *Diagnosed only* → **"Diagnosis only — no repairs
    performed"**
  - otherwise (Completed mixed with Diagnosed only) → **"Work finished —
    some items diagnosed only"**

  The critical case: a visit where **every** item is *Needs return visit*
  has COMPLETE documentation and repair status **"Return visit
  required"** — it must never read as repairs complete. Both statuses are
  snapshotted onto the saved record (`complete`, `repairStatus`) at save
  time, so history shows the statuses as they stood when the visit was
  saved.

### Property-manager mode

A persisted setting (`pmMode` in `fixvisit.settings.v1`), toggled from
the visit summary card. When ON, documentation completeness additionally
requires **at least one photo on every Completed work item** — the proof
property managers actually approve payment from. The summary panel
explains the gap by name ("property-manager mode: add at least one photo
to "…"") until the photos are attached, and the status lifts the moment
they are. Diagnosed / return-visit items do not require photos (there may
be nothing photographable fixed yet); only Completed items do.

### Client & property profiles

`fixvisit.profiles.v1` holds reusable profiles —
`{ id, client, address, unit }` — for the contractor's regular accounts.
Picking a profile in the "This visit" card autofills client, property
address, and unit on the current visit. "Save current client / property"
creates a profile from the fields on screen, or updates the selected
profile (or an exact client+address+unit match) instead of duplicating
it — so editing a profile is: pick it, change the fields, save again.
"Delete selected profile" removes it. Profiles are included in backup
export and restored on import.

### Start similar visit

Each history row has a **Start similar** action for repeat visits to the
same property. It loads a fresh builder state carrying **only**:

- client, property address, unit, and the reference numbers (work order
  #, PO #, invoice reference), and
- the work items' titles and descriptions.

It deliberately does **not** copy photos, the date, arrival/departure
times, item statuses, issues, or any charges (parts, labor, trip fee,
tax) — those describe the old visit, and carrying them over would
fabricate the new visit's record. The new visit starts fresh today,
unbilled, with every item awaiting a status.

### History totals & CSV

- **History totals** — `historyTotals(visits, now)` counts visits in the
  current calendar month (by visit date) and sums `amountDue` snapshots —
  this month and all time — plus the all-time **awaiting payment** total
  (Submitted + Approved). Amounts are snapshotted onto the record at save
  time (`makeRecord`), so editing the builder later never rewrites what a
  past visit billed.
- **CSV export** — one row per saved visit: date, Report ID, client,
  address, unit, work order #, PO #, invoice reference, visit type,
  documentation status, repair status, payment status, work-item /
  issue / photo counts, and amount due. Standard CSV quoting (fields
  containing commas, quotes, or newlines are quoted, quotes doubled).

Worked example — the landing page's sample visit, reproduced in the
tests:

- Client: Harborview Property Management, 1180 Elm St, Unit 4B, work
  order WO-2481, invoice reference INV-2026-118. Payment terms: Net 15.
  Acknowledgment: property manager.
- Work items: kitchen P-trap replacement (Completed, 2 photos), toilet
  wax ring replacement (Completed, 2 photos), GFCI outlet diagnosis
  (Diagnosed only). All three have statuses, arrival 09:15 and departure
  11:45 are set → documentation **COMPLETE**, time on site **2.5 hrs**,
  repair status **"Work finished — some items diagnosed only"**. (In
  property-manager mode it stays COMPLETE because both Completed items
  carry photos.)
- Issue: GFCI outlet (Medium, needs a return visit) → appears under
  "Recommended next steps".
- Charges: parts $18.50 + $9.75 + $4.25 = **$32.50**; labor 2.5 hrs × $65 =
  **$162.50**; trip fee **$45.00**; tax 0% → **amount due $240.00**.

## Data & privacy

Everything lives only in the visitor's own browser. Nothing is sent to a
server.

**localStorage:**

| Key | Contents |
|---|---|
| `fixvisit.settings.v1` | Company name/phone/email + logo (data URL) + the property-manager mode flag (`pmMode`). |
| `fixvisit.visits.v1` | Saved visit history (Pro): full visit snapshots incl. amount due, both statuses, payment status, photo ids. |
| `fixvisit.profiles.v1` | Client & property profiles: `{ id, client, address, unit }`. |
| `fixvisit.draft.v1` | The in-progress visit, so an interruption doesn't lose work. |
| `fixvisit.pro.v1` | Pro unlock flag (see MVP note above). |
| `fixvisit.freereport.v1` | Count of free PDF reports used (0 or 1 — see "First report free"). |

**IndexedDB:** database `fixvisit`, object store `photos` — photo id →
photo data URL. Photos are deliberately **not** kept in localStorage (they
are far too big for its ~5 MB quota). Attached photos are downscaled to at
most 1280px (JPEG, 0.82 quality) where the browser supports canvas. If
IndexedDB is unavailable in a browser, the app falls back to keeping
photos in memory only and tells the user plainly in the "Your data" card
that photos will be lost when the tab closes. Deleting a work item or a
saved visit also deletes its photos from IndexedDB.

The app tells users plainly: "Your data is stored only in this browser on
this device — export a backup regularly." Backup export downloads a JSON
file with settings, profiles, saved visits, the current draft, and all
referenced photos (as data URLs); import restores all of it, photos and
profiles included. Import verifies the file is actually a FixVisit Report
backup (app name + visits array) and rejects anything else without
touching existing data.

Photo-handling guidance (also on the landing page's Privacy section, in
plain words): handymen work inside someone's home — get the owner's or
tenant's permission before photographing a property and before sharing
reports containing photos; keep personal belongings out of frame —
photograph the work and its condition, not people's things; and export
each visit's evidence pack promptly into the business's own storage,
because the browser copy is the only copy until then.

## Deploy

Any static host works — copy the three files (or at least the two HTML
files) to the host as-is:

- **Netlify / Vercel:** drag-and-drop the folder, or connect the repo; no
  build command, publish directory = this folder.
- **Cloudflare Pages:** same — framework preset "None", no build command.
- **GitHub Pages:** fine for demos/preview only — GitHub Pages terms
  prohibit primarily commercial SaaS use, so don't run the paid product
  from there.

The only external requests the app makes are the jsPDF and JSZip CDN
scripts in `app.html` (cdnjs). If jsPDF fails to load, everything except
the PDF and evidence-pack buttons still works and the user gets a clear
message; if only JSZip fails, the PDF button still works and the
evidence-pack button says so plainly.
