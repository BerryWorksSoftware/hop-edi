# Quick Start: turn an X12 purchase order into a business summary

In about 5–10 minutes, read a fictional X12 850 purchase order, calculate its
merchandise value, and flag it for review. The output is ordinary CSV that an
application or spreadsheet can use. No database, email account, or coding is
needed to run the demo.

## Before you start

Use Apache Hop 2.19.0 with Java 21 and BerryWorks EDI Input 0.1.0 installed.
See the [installation instructions](../README.md#install-through-hop-marketplace).
Download this repository using GitHub **Code → Download ZIP**, extract it, and
keep the entire `quick-start` folder together. Run it from a writable folder.
The demo includes its own X12 850/004010 model.

## Run it in Hop

1. Open Hop GUI and open `pipelines/po-summary.hpl` inside this folder.
2. Click **Run**. Choose your **local** pipeline run configuration.
3. On the run dialog's **Parameters** tab, set `QUICK_START_HOME` to the
   **absolute filesystem path of this quick-start folder**, without quotes or
   a `file:` prefix. For example, `/Users/you/Downloads/hop-edi-main/quick-start`
   or `C:\Users\you\Downloads\hop-edi-main\quick-start`.
4. Leave `REVIEW_THRESHOLD` at `10000` and run the pipeline.
5. Confirm there are no errors. Open `output/order-summary.csv` and
   `output/order-lines.csv` in a text editor or spreadsheet.

Each run overwrites these two output files. The `expected` folder holds the
reference results for the default threshold. If you see “models directory does
not exist,” check the folder parameter; it must point to `quick-start`, not
its `pipelines` subfolder.

## What you should see

The order summary contains one row:

| PO | Date | Buyer | Ship to | Currency | Lines | Merchandise value | Review required |
|---|---|---|---|---|---|---|---|
| PO-10042 | 2026-10-09 | Acme Retail | Chicago DC | USD | 3 | 12450.00 | Yes |

The line output explains the total:

| Product | Quantity | Unit | Unit price | Line value |
|---|---:|---|---:|---:|
| WIDGET-A | 100 | EA | 75.00 | 7500.00 |
| WIDGET-B | 50 | EA | 90.00 | 4500.00 |
| WIDGET-C | 30 | EA | 15.00 | 450.00 |

Now run again with `REVIEW_THRESHOLD=15000`. The total stays the same and
`review_required` becomes `No`. The rule is **strictly greater than** the
threshold, so an order equal to the threshold is not flagged.

For a second experiment, edit the first PO1 line in `input/po-10042.edi`,
changing quantity `100` to `200`. Run again at threshold `10000`: the first
line becomes `15000.00`, the order becomes `19950.00`, and review is `Yes`.
Restore the quantity when finished.

## How the pipeline works

Read the canvas from left to right:

1. **Read X12 purchase order** parses the EDI using the supplied model.
2. **Extract PO business fields** keeps PO1 line items, carrying the order
   header and party names onto each line.
3. **Calculate line value** multiplies quantity by unit price. It copies
   each line to both output branches.
4. **Write order lines** saves line detail. **Summarize purchase order**
   groups by the order header, counts the lines, and sums their values.
5. **Flag high value order** applies the threshold, then **Write order
   summary** saves the result.

For someone new to EDI: `BEG` supplies the PO number and date; `CUR` supplies
currency; `N1` with `BY` identifies the buyer and with `ST` the ship-to party;
`PO1` supplies quantity, unit price, and product identifier. These are
business records within an X12 interchange, rather than CSV columns.

For someone new to Hop: a transform is one processing step; an arrow carries
rows between steps. Double-click a transform to inspect its settings. You can
preview **Extract PO business fields** to see the business rows before they
are totaled. The JavaScript transforms are already configured; editing their
code is optional.

## Scope of this example

This is a deliberately small teaching example, not a general trading-partner
map or an EDI validator. It assumes USD, nonnegative quantities/prices, buyer
and ship-to names before the lines, and a vendor product identifier (`VP`) in
PO1 positions 06/07. It calculates quantity × unit price without price-basis
or unit conversions. The merchandise total excludes tax, freight, discounts,
and allowances; it is not an invoice amount or a reconciliation against CTT.
The review flag is data you could later route to a notification or approval
process. This demo sends no email.

## Optional: make it a Hop project

You can instead register a Hop project whose home folder is this `quick-start`
folder and whose project configuration is the supplied `project-config.json`.
It includes a `local` run configuration under `metadata`.
When that project is active, `QUICK_START_HOME` defaults to `${PROJECT_HOME}`;
you can leave it at its default. The explicit folder parameter above lets you
try the demo in your existing project first.
