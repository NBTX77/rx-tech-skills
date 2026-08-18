---
name: project-kickoff-readiness
description: Run a ConnectWise project kickoff readiness review — assemble what a PM needs before kickoff and, more importantly, name what is missing. Use when asked for a "kickoff review", "kickoff readiness", "project readiness", "readiness report", "review the new projects", "what's on the project board", "are these projects ready", "10 New cohort", or when handed a project number or RXS number and asked whether it is ready to start. Also use before a Monday project meeting, or when someone asks what to chase on a project. Produces a flags-first report, optionally as a PDF.
category: project-management
---

# Project kickoff readiness review

Assemble what a PM needs to walk into kickoff — **and name what is missing.** The second half is the
point. A document list is not worth producing; the value is in the flags.

**Lead with what's wrong. Rank worst-first. Say what to chase and who to ask.**

## Before you start

Requires the **RxTech** MCP connector. If `rx-project-kickoff-context` is in your tool list, use it —
it does most of the assembly. If it is not (a session opened before it deployed will not see it), fall
back to the manual sequence below, which is what it does internally.

Never assemble this from memory or from a previous run. **Every figure is read live.**

## 1 · Resolve the cohort — never assume it

```
cw-list projects  filter {status: {name: "10 New"}, closedFlag: false}
```

⚠ **The cohort churns hard.** It was 12 on 2026-07-27, 4 on 2026-08-02, and 1 by that evening. Projects
move out mid-conversation. Resolve it at the moment you run.

**Carry forward projects that moved out in the last few days** if their readiness issues are unresolved.
A forked BOM does not stop mattering because the status changed. Put them in their own section.

For a single project, take `projectId` or the RXS number and skip to step 2.

## 1a · Always include a second cohort — `20 Incomplete handoff`

```
cw-list projects  filter {closedFlag: false}      # then group by status.name
```

Open project statuses observed: `10 New`, **`20 Incomplete handoff`**, `30 Assigned to Project Manager`,
`33 Assigned to Service`, `34 Assigned to IT Installation`, `50 Completed`.

**`20 Incomplete handoff` means the sale closed but something needed to run the job never arrived**, so it
never reached a PM or a crew. These are the projects nobody is looking at — they carry no PM attention by
definition. **Give them their own section in every report.**

For each one, read the notes and answer the only question that matters:

```
cw-list project-notes projectId=N
```

| Column | Where it comes from |
|---|---|
| Project / RXS | `id`, `name` |
| Client, PM | `company.name`, `manager.name` |
| Waiting since | `estimatedStart` → days elapsed. **Not `lastUpdated`** — a note edit is not progress. |
| **Why it is stuck** | the notes — see below |

### Grade the reason, do not just quote it

The convention is note `type: "Internal Kick Off"`, usually `flagged: true`. Three grades, and say which:

- **Documented** — states *what is missing* **and** *what releases it*. Project 818 is the model:
  *"Incomplete reason: awaiting RFIs and pending changes in the SiteOwl system. Action: hold incomplete
  status until RFI changes are resolved, then follow up. PM Troy Jordan."*
- **Partial** — names a gap with no owner and no date. *"kick off meeting skipped. No agreement created."*
  ⇒ ask **who owes it and by when**.
- **Outcome only, or nothing** — *"project kickoff was unsuccessful"* records that something failed without
  recording what, which four weeks later is the same as silence. **Zero notes is the worst case and it
  happens** (project 862, 42 days, no note of any kind).

⚠ **Do not soften this.** A project parked with no written reason is invisible work, and the report is the
only thing that will surface it. Name the PM and say the reason is absent.

### Look across the set, not just down it

Cluster by PM, client and board before writing. Live example: 818 and 862 are both Tri-Tech, both Troy
Jordan, both VA medical centres, both parked — **plausibly one shared blocker, not two**, and worth asking
as a single question. Three of four sat on Systems Integration.

Also: **none of these carry an `actualStart`**, and estimated ends run a year out. Those dates are as
fictional as the ones on a `10 New` project — do not report them as schedule.

### Running this cohort on its own

"Incomplete handoff review", "what's parked", or "project board only" means run **just** this section as a
standalone report — skip the `10 New` kickoff sections entirely. Give each project the full per-project
gather in §2 (documents, folder contents, procurement), because the most valuable finding is usually
**"nothing is actually missing"**. Live example: project 862 had a signed work order, a signed VA
acknowledgement, a SOW, 17 MB of plans, three vendor quotes, a PM folder created 4 days earlier and a BOM
edited 2 days earlier — and had been parked 42 days with **zero notes**. Work was continuing off-status.

⇒ Add one line to the at-a-glance table: **"Real blocker?"** — *yes and written down* / *was real, may be
cleared* / *unknown* / *none visible*. That column is the report.

⇒ And always ask: **is work still happening while this sits parked?** Compare document
`lastModifiedDateTime` against the date the project stalled. If someone edited a BOM last week on a project
waiting since July, the status is stale, not the project.

## 2 · Per project, gather in this order

| What | Call |
|---|---|
| Identity, status, PM, dates | `cw-list projects filter {id: N}` |
| Provenance | `cw-list conversions projectId=N` |
| Documents + SharePoint links | `cw-list documents recordType=Project recordId=N` |
| Folder contents for each link | `m365-list-resources resourceType=drive-items shareUrl=<the link>` |
| Products actually on the project | `cw-list project-products projectId=N` |
| Phases, tickets, budget hours | `cw-list project-work-plan projectId=N` |
| Ticket owner / SLA, if handed off | `cw-list tickets filter {id: <ticketId>}` |

Read the folder contents. **A link you did not resolve is not a document you have seen.**

## 3 · The flags

Every one of these is a real observed failure, not a hypothetical.

| Flag | What it means |
|---|---|
| **No documents at all** | Nothing attached, no link. See the "missing vs unlinked" rule below. |
| **No BOM** | Field team has no parts list. A distributor quote is **not** a BOM — it is a price list. |
| **No SOW** | Nothing states what "done" looks like. |
| **Quote not signed** | Signed quotes tend to carry `signed` in the filename. Absence is worth asking about, not asserting. |
| **Competing BOM versions** | Two BOMs for one job. Never present one as "the BOM". |
| **Forked duplicate** | Same filename in two folders with **different sizes or modified dates** — a copy that has diverged. Compare size and `lastModifiedDateTime`, not just the name. |
| **Late scope change** | A quote or spec added *after* the original quote date. Ask whether it was re-quoted. |
| **Products not on the project** | `project-products` empty while the opportunity carries lines — provenance and billing will disagree. |
| **Labor not linked** | Labor lines carrying no project reference while material lines do. |
| **Auth-gated link** | Dropbox or similar. **Do not chase it.** Flag as human work: unpack into SharePoint or attach to the project. |
| **Unresolved SKUs** | `[VERIFY SKU: …]` placeholders that reached a customer quote. |
| **Not converted** | No conversion record — only from a proven-200 empty list, never from a failed lookup. |
| **Stale** | Untouched for days while at `10 New`. (A start date in the past is **not** a flag — see §3a.) |
| **Nothing on order** | Materials in scope, no matching PO in QuickBooks. |
| **Partial delivery** | Σ`receivedQuantity` < Σ`quantity` on the PO lines. State the % and name the open SKUs. |
| **Line closed short** | `isManuallyClosed: true`, or `Other1` = `Closed` — cancelled, will never arrive. |
| **Back-ordered line** | `Other1` = `Back Ordered` — no date and no vendor commitment. Chase before committing a project date. |
| **Ships after the project ends** | `Other1` `ETA`/`ESD` date (late end of an ESD range) later than `estimatedEnd`. |
| **No status on an open line** | `Other1` blank or holding a vendor SO number instead of a status — nobody knows when it lands. |
| **PO points at two projects** | `memo` RXS and `otherCustomField2` RXS disagree. |

## 3a · What this report is FOR — and what therefore is not a flag

This review is the **input to the kickoff meeting**. Dates are set *at* that meeting, and scheduling
happens there or later that week. The report's job is to answer one question per project:

> **Can we set dates and schedule this today — and if not, what blocks it?**

Do **not** flag these on a `10 New` project. They are the expected state, and flagging them buries the
things that actually block the meeting:

| Not a flag | Why |
|---|---|
| **Start date already passed** | Dates on a `10 New` project are placeholders carried over from opportunity conversion. Real dates are set at kickoff. |
| **Nobody scheduled** | Scheduling is an *output* of the meeting, not a precondition. An empty schedule board is normal. |
| **PM is Travis / PM unassigned** | A guide, not a rule. Travis is sometimes genuinely the PM. |

A blocker is something that would make the meeting **unable to commit to a date**: no BOM, no scope,
documents belonging to the wrong customer, nothing ordered on a materials job, or an unknown lead time.
State the verdict per project as **YES / YES-with-one-call / NO**, and for NO name the single thing.

## 4 · Four traps that will make you wrong

**SharePoint is only visible through a link recorded in ConnectWise.** There is no way to search
SharePoint by project or client name. If a folder exists but nobody pasted the link onto the project,
you cannot see it.
⇒ **Never report "no documents exist."** Report **"no documents attached — a folder may exist but is not
linked."** Confirm which before anyone acts.

**Service-ticket-originated projects have no sales package, and never did.** Check the conversion record:
if `sourceOpportunity` reads like a ticket description rather than an `RXQ…` code, the project came from
a service ticket that grew into project work. There is no missing quote or SOW to find.
⇒ Flag it as *"no written scope — service-originated"*, not *"locate the sales documents."* The remedy is
a decision about whether these need a scope document, not a search.

**`sourceOpportunity` is a name, not a foreign key.** Project 858 returns `RXQ203262-SATX-UPG-NET-ORIG`;
project 867 returns `HP Mini Replacement [New HP quoted]`. The RXQ format is a convention.
⇒ Report the name, converter and date. **Do not try to resolve it to an opportunity record.**

**A link that resolves is not a link that is correct.** Project 880 carried a link titled
*"SignedQuote- RXQ203508- WahooComfortSolutions"* that resolved cleanly to
`/Customers/Blossom Aerospace/RXQ203507` — three files, all a different customer, one digit apart on the
quote number, with a purchase order already placed against the project.
⇒ **Always check the resolved folder path and the filenames against the project's client and RXQ/RXS
number.** A mismatch is a STOP, not a note — it is also a customer-document exposure.

**Empty is not absent.** A route failure read as `[]` becomes "never converted" — a materially wrong
business fact. If a call errors, say so; do not let it become a finding.

## 5 · Procurement — read it, and report dates not just status

**"Has the gear been ordered, and when does it land?" is the most useful thing a PM wants before kickoff.**
It is answerable. Do not skip it.

### The join

QuickBooks PO **`memo`** carries `Sales Order RXS######`. That is the link to the ConnectWise project —
there is no formal key. `otherCustomField2` often carries the RXS too, and **when the two disagree, say so**
(seen live: memo `RXS203509` against reference field `RXS203503` — one is wrong, and until it is resolved
the money is attributable to either project).

```
qbd-list-resources resourceType=purchase-orders  filter {since: "<~6 weeks back>"}  includeLineItems=true
```

**Best key first: PO lines carry `payee`, and header POs carry `shipToEntity`, holding a QuickBooks
customer:job pair** — `Woods Comfort Systems:RXS203508`, `Tenoch Distribution:RXS203503`,
`Blossom Aerospace Texas:RXS203507`. That is structured, not a text convention. **Use it to confirm a match
before accusing anyone of mis-filing a PO** — it settled one live case where the memo looked wrong.

Fall back to the RXS in `memo`. **`otherCustomField2` on the header usually carries the RXS too,
and when the two disagree one of them is a typo.** Resolve it by reading the PO's line `description` fields —
buyers paste `End User Information — <client>` and the vendor quote number in there. Live example: PO
260008011-3 had memo `RXS203509` against reference `RXS203503`; a line read *"End User Information — Tenoch
Distribution"*, and RXS203503 is the Tenoch project, so **the memo was wrong**. Say which one won and why —
the memo is what the match runs on, so an uncorrected typo attaches the PO to the wrong job forever.

⇒ **Read `Other1` first (below). It is the ETA, and it is the answer to the question the PM is asking.**

### ✅ Line-level status is the real answer — always pass `includeLineItems=true`

Each PO line carries **`quantity`**, **`receivedQuantity`**, `isBilled` and `isManuallyClosed`. That is a
per-SKU fulfillment status, and it is far better than anything else available:

```
received % = Σ receivedQuantity ÷ Σ quantity   (over lines where quantity is not null)
open lines = lines where receivedQuantity < quantity   → name the SKUs still outstanding
```

Measured live 2026-08-17 — the boolean hides everything useful:

| PO | `isFullyReceived` | Line truth |
|---|---|---|
| Graybar 260008004-1 | `false` | **92% received**; only 4 SKUs open (`CF5WH-X`, `OCFC5WH-X`) |
| Amazon 260008005-4 | `false` | **99% received**; one rack `RK419WALLV-N` outstanding |
| ScanSource 260008005-5 | `false` | 50% — but the open lines are *Shipping Expense* and *Misc*, i.e. **all gear landed** |
| D&H 260008011-2 | `false` | **0% — nothing has shipped** |

"`isFullyReceived: false`" spans "one accessory short" and "nothing has arrived." **Never report the
boolean on its own.** Report `received %` and name the open SKUs.

Gotchas, all observed:

- The array key is **`lines`**, not `lineItems`, and it is absent unless `includeLineItems=true`.
- Some lines carry `quantity: null` (headers, comments). Exclude them from the ratio or it divides by zero.
- **`isManuallyClosed`** (on the header and on each line) means *closed short / cancelled*, not received.
  A closed line will never arrive — flag it, do not count it as fulfilled.
- **`isBilled`** was `false` on 63 of 63 lines — the vendor bill is entered later. It is not a delivery signal.
- `linkedTransactions` returned **empty** even with `includeLinkedTransactions=true`. Do not rely on it;
  the item-receipt `refNumber` join still works if you want dollar totals, but line quantities are better.

### ⭐ The ETA lives in the LINE-level `Other1` custom field — this is the answer

Purchasing hand-maintains a per-line status in **`otherCustomField1`** (QuickBooks `Other1`, also visible in
`customFields` as `{name: "Other1"}`). **This is the real ETA field.** It is per SKU, it is current, and it
is the single most valuable thing in the procurement read.

⚠ **Line level only.** The *header* `otherCustomField1` is buyer initials (`SK` on all 14 POs sampled) —
ignore it. Only read `Other1` off entries in `lines`.

Observed vocabulary — parse the prefix, then the date:

| Value | Meaning |
|---|---|
| `DEL 8/7/2026` | **Delivered** on that date |
| `E-DEL 8/6/2026` | Electronically delivered (licences, keys) |
| `ETA 8/21/2026` | Firm arrival estimate |
| `ESD 09/21/2026 - 10/12/2026` | Estimated *ship* window — a **range**, and the late end is the one to plan against |
| `In Process` | Vendor has it, no date yet |
| `Back Ordered` | ⚠ no date, and no commitment — chase this |
| `Reship-Factory` | Sent back / reshipping from the factory |
| `Closed` | Line cancelled, will never arrive |
| `SO # 98070657` | Buyer put a vendor SO number here instead of a status — no ETA available |

Free text, hand-entered — expect date formats to vary (`DEL  8/11/2026` has a double space) and expect
non-status values. **Match on the prefix; if it does not parse as a status, report the raw string.**

**`otherCustomField2` (`Other2`) carries tracking and vendor references** — `1Z…` UPS, `9400…` USPS,
`Delivery 8004991595`, `GB 3003108755`, `SO 9001480408`, and occasionally `Lead time 5 Days`. Report the
tracking number when a line is shipped but not delivered; it is what the PM actually needs to chase.

Live at 2026-08-17 — this changes the readiness verdicts:

| PO / project | Line status |
|---|---|
| Ingram 260008011-1 (RXS203507) | `ESD 09/21/2026 - 10/12/2026` on `P01366-B21`. **Ships as late as 12 Oct.** One line already in transit (`ETA 08/17/2026`, UPS 1Z4X67420387854400). |
| D&H 260008011-2 (RXS203508, project 880) | `In Process` ×2, **`Back Ordered`** on `KCP556SS8-16`. No date on anything. |
| ScanSource 260008011-3 (RXS203509) | `ETA 8/21/2026`, `Lead time 5 Days`. Fine. |
| Amazon 260008005-4 | The open rack line is **`Closed`** — it is cancelled, not late. |

### ⚠ The date fields on the PO header are NOT ETAs — measured, not assumed

Across 150 POs sampled 2026-08-17:

```
dueDate  − transactionDate (days): 0×100, 30×28, 60×16, other×6   → Net 0 / 30 / 60 payment terms
expectedDate − transactionDate   : 0×141 of 150                   → field is not maintained
```

`dueDate` is **payment terms**. `expectedDate` is copied from the order date and left alone. Neither is
an ETA. An earlier run of this review reported "project 880's gear arrives 25 September, after the
project ends" — that was **wrong**; 25 Sep was Net 30 on an 26 Aug order.

⇒ Never derive a schedule risk from `dueDate` or `expectedDate`. **The ETA is line-level `Other1`**
(above). Only if `Other1` is blank or carries no status is the answer "not in QuickBooks — call the vendor."

Also flag: **no PO at all** on a project with materials in scope; a PO whose RXS does not match any open
project; and long lead times (a 95-day ScanSource order is a real pattern here, not an outlier).

### If it stops working

`page` on `qbd-list-resources` was historically accepted, echoed in the envelope, and never sent — page 39
returned page 1 while reporting itself as page 39. That is fixed, and the response now carries a `walk`
block (`pagesFetched`, `stopped`, `remaining`). **If `walk` is absent or a `since` filter returns records
from 2018, stop and report the read as unreliable rather than concluding nothing was ordered.**

## 6 · Output

Default to a **flags-first summary in chat**: an at-a-glance table ranked worst-first, then one section per
project, then a method section stating what was read and what could not be seen.

**Produce a PDF when asked**, or when the report is going to someone who was not in the conversation. Use
the `pdf` skill.

### Build it with reportlab — house style

Letter, 0.7in side margins. Palette: navy `#1F3A5F` headings, accent `#0B6E99` subheads, red `#B3261E`
flags, amber `#B26A00` caution, green `#1E7A3C` clear, grey `#5F6368` method notes. Body 9.7pt/14,
table cells 8.6pt/11.4. Every table gets a 0.5pt `#C9D2DC` grid, `#F1F4F8` header row, `VALIGN TOP`,
6pt side padding.

```python
def tbl(rows, widths):
    t = Table(rows, colWidths=widths, repeatRows=1)
    t.setStyle(TableStyle([
        ('GRID',(0,0),(-1,-1),0.5,colors.HexColor('#C9D2DC')),
        ('VALIGN',(0,0),(-1,-1),'TOP'),
        ('BACKGROUND',(0,0),(-1,0),colors.HexColor('#F1F4F8')),
        ('LEFTPADDING',(0,0),(-1,-1),6),('RIGHTPADDING',(0,0),(-1,-1),6),
        ('TOPPADDING',(0,0),(-1,-1),4.5),('BOTTOMPADDING',(0,0),(-1,-1),4.5)]))
    return t
```

Wrap every cell in `Paragraph(...)` so it wraps; colour inside the cell with
`<font color="#B3261E"><b>…</b></font>`. **Give an ID column at least 1.0in** or project numbers wrap
mid-digit. One `PageBreak()` per project section.

⚠ **Always render and look at it before sending** — `pdftoppm -png -r 70 out.pdf pg` then read the PNGs.
Column-width bugs and stale headings only show up visually.

Structure that works:

1. **At a glance** — table: project, client, **"Can we schedule it today?" (YES / YES-with-one-call / NO)**, the blocker
2. **What the meeting needs to decide** — numbered, one line each
3. **One section per project** — identity table, provenance, documents table, flags in red, "what to chase",
   and a **line-status table** for every PO:

   | Item | Qty | Recv | Status (Other1) | Ref / tracking (Other2) |
   |---|---|---|---|---|
   | `MX68-HW`<br/><sub>Meraki MX68 firewall</sub> | 1 | 0 | <span style="color:green">**ETA 8/21/2026**</span> | Lead time 5 Days |
   | `KCP556SS8-16`<br/><sub>Kingston DDR5 16 GB</sub> | 1 | 0 | <span style="color:red">**Back Ordered**</span> | — |

   Colour the status: green for `DEL`/`E-DEL`/a firm `ETA`, amber for `In Process`/an `ESD` range, red for
   `Back Ordered`/`Closed`, grey for blank. **Show blank cells as "— blank —" rather than omitting the row** —
   a line nobody has annotated is itself the finding.
4. **Incomplete handoff** — the §1a table, then a short "what this says about the process" note
5. **Deliberately NOT flagged** — restate §3a so the reader knows the omissions were deliberate
6. **Carried forward** — projects that moved out but still have open issues
7. **Method and blind spots** — what was read, what cannot be seen, cohort volatility

Name flags in **red**, keep every claim traceable to a value you actually read, and put the byte sizes and
timestamps in when they are the evidence.

## 7 · Tone

Write for a PM who has ten minutes before a meeting.

- **Actions, not observations.** "Ask Brad for the BOM" beats "BOM is absent."
- **Name the person** where the data gives you one — the file's `lastModifiedBy`, the converter, the PM.
- **Do not pad a clean project.** If it is ready, say so in one line and move on.
- **Separate what you know from what you infer**, and never dress a gap in your access as a finding about
  the business.
