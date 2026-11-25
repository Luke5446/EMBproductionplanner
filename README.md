# EMB production

Embroidery and transfer jobs for the EMB room, the sales office and the warehouse, replacing the
Google Sheets. Live at https://luke5446.github.io/EMBproductionplanner/ (GitHub Pages, branch main).

## Where the jobs come from
No second Sage feed. The Order tracker's daily Sage paste (`ordertracker_live.sql`, published in
`SOPtracker/data.json`) already carries every live sales order line, the LOGOAPPLICATION /
TEXTAPPLICATION charge lines, the transfer (ID/TX) lines and the free text logo rows. This app reads
that file, ties each marker to its garment lines the way the Order tracker does, and makes **one job
per order per kind** (EMB, Transfer). An order whose reference starts `EMB:` / `DTF:` / `PRINT:`
with no line markers is a job on every garment. A job leaves when Sage shows every garment despatched.

Embroidery-only and DTF works orders raised in the Production Plan (special makes, branded codes)
are read from the planner's `data.json` and the warehouse log and shown in the same queue; their
pick, machine and done marks stay in the planner.

## Tabs
- **Orders** (sales office, each PC with the token): by default only jobs that still need
  something: a logo and a placement, or a works order file uploaded. Filter by the Sage user who
  put the order on. Open a job to type the logo and placement (a branded code the planner knows,
  e.g. EGOCICJ0193XS01, brings its logo, placement and image from the style master), or **Upload
  works order** (the sales office's own WO, PDF/image/xlsx) which goes to `wo/` in this repo and
  marks the job actioned. Print the EMB works order.
- **Pick list** (warehouse): stock is worked out per company and code, oldest promised date
  first, respecting Sage's allocation, then the back-to-back PO, the planner's works order for the
  line or code, a container, else short. Four groups: **Ready to pick in full** (tick jobs, print
  pick lists in bulk; printing = Picking), **Picking**, **Waiting on a production works order**,
  **Waiting on stock**. Bins come from BinFinder's `stock.csv`. Mark each garment picked (or All
  picked); a job with some garments short stays on the list as **Part picked** until the rest is in.
- **EMB queue** (EMB room): every live job, picked first then by due date. Type the stitch count
  per garment (offered to remember against the job's logo, so the next order gets it), choose the
  machine (1 to 4 six heads, 5 single head, DTF twin or single press), mark **Done**.
- **Done**: done jobs awaiting despatch, with undo.
- **Logos**: the logo memory (name, customer, placement, stitch count); images come in phase 3.
- **Settings**: stitches per minute (900), set-up minutes per product (10), lead time in working
  days (Tibard 10, OH 7). Due date = Sage order date + lead time.

Time estimate = garments x stitches / 900 + 10 min per product, shared over the machine's heads.

## Marks, the token and PINs
Marks (actioned, picks, machine, done, printed), logos and users are kept in `emb.json` in this
repo, written through the GitHub contents API with a fine-grained token for this repo
(`emb_token` in the browser; **Set up this PC**, or a share link `…/#token=XXXX` which is
removed from the address bar once taken). Everyone can view. Editing needs the token **and** a
PIN sign-in (top right): the EMB manager edits the EMB queue, Logos and Settings; the sales
office edits Orders; the warehouse edits the Pick list; Admin manages users and PINs
(Settings → Users). PINs are SHA-256 hashed like the CRM's. Default PINs on first set-up:
EMB manager 1111, Sales office 2222, Warehouse 3333, Admin 0000 - change them straight away.

## Data model (for the on-prem system)
`jobs[key]`: co, so, kind, cust, logo, place, stitches, colours, notes, actionedAt/By,
wo{path,name,at,by}, printedAt, pickPrintedAt, picks{lineKey:{qty,at,by}}, machine, machineAt,
doneAt/By. `logos[]`: name, acct, cust, place, stitches, stitchesAt, file (phase 3).
Works order files: `wo/<TIB|OH>-<SO>-<kind>-<file name>`. `users[]`: key, name, edits[], pinHash.

## Changelog
- v0.4 (25 Nov 2026) Actioned tab with Undo; works order files per garment (or one for all with the tick); a job is actioned when every garment has a file, or logo + placement are saved, or Mark actioned is pressed. `jobs[key].wos[]` = {path,name,at,by,lines[]} (an old `wo` is read as covering every garment).
- v0.2 (25 Nov 2026) Orders shows only jobs needing action, sales person filter, works order
  file upload; stitch count moved to the EMB queue and remembered per logo; pick list works out
  stock (oldest promised first, b2b POs, planner WOs, containers), bins from BinFinder, ready list
  with bulk pick-doc print; planner works orders bring logo, placement and image from the style master.
- v0.1 (25 Nov 2026) phase 1: jobs from the Order tracker paste, Orders, EMB WO print, Pick list,
  EMB queue with machine and time estimate, Done, emb.json marks.
