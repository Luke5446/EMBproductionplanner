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
- **Orders** (sales office): new jobs with the Sage text under them. Confirm a job: logo name,
  placement, stitch count per garment, thread colours, notes. Print the EMB works order.
- **Pick list** (stores): every confirmed job with each garment's free stock. Mark each garment
  picked; a job with some garments short stays on the list as **Part picked** until the rest is in.
- **EMB queue** (EMB room): confirmed jobs, picked first then by due date. Choose the machine
  (1 to 4 six heads, 5 single head, DTF twin or single press), mark **Done**.
- **Done**: done jobs awaiting despatch, with undo.
- **Settings**: stitches per minute (900), set-up minutes per product (10), lead time in working
  days (Tibard 10, OH 7). Due date = Sage order date + lead time.

Time estimate = garments x stitches / 900 + 10 min per product, shared over the machine's heads.

## Marks and the token
Confirmations, picks, machine, done and printed marks are kept in `emb.json` in this repo
(`jobs` keyed `company|SO|kind`, `logos`), written through the GitHub contents API with the
fine-grained token already on the warehouse and EMB PCs (`tibard_cutlog_token` in the browser);
the token must be allowed to write this repo. Without the token the page is view only.
`emb.json` is read through GitHub Pages by everyone.

## Data model (for the on-prem system)
`jobs[key]`: co, so, kind, cust, logo, place, stitches, colours, notes, confirmedAt/By,
printedAt, picks{lineKey:{qty,at,by}}, machine, machineAt, doneAt/By.
`logos[]`: name, acct, file (path under `logos/`), place, stitches (phase 3).

## Changelog
- v0.1 (25 Nov 2026) phase 1: jobs from the Order tracker paste, Orders, EMB WO print, Pick list,
  EMB queue with machine and time estimate, Done, emb.json marks.
