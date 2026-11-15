---
name: whatnick-energy-monitor-elecrow-ordering
description: Package verified PCB/PCBA fabrication files and manage Elecrow quotations using an authenticated browser and Playwright. Use for Gerber/BOM/CPL uploads, quote and service-order audits, duplicate detection, build-status tracking, uploaded-file hash checks, lifecycle-board handoff, and safe approval gates before checkout, payment, or production. Preserve account privacy.
compatibility: Python with Playwright and an installed Chromium browser; KiCad CLI for fabrication validation/export.
metadata:
  org: whatnick
  domain: pcb-manufacturing
---

# Elecrow PCB/PCBA quotation and production management

Use the browser + Playwright for Elecrow fabrication submission, not email or
an invented private API. For bare PCB, the verified entry point is
https://www.elecrow.com/pcb-manufacturing.html and the cart is
https://www.elecrow.com/checkout/cart.

For PCBA, distinguish a submitted RFQ/service order from a paid shop order and
from a production release. A provisional quote with zero component cost is not
ready for payment. A displayed status is evidence only at the time checked;
record the check date.

## Authority and privacy

Separate quote/cart preparation, order placement, payment and manufacturing
authorization. A request to quote permits uploads and adding quoted jobs to
the cart, not checkout/payment or a production start. Stop at the requested
boundary. Do not describe a cart SKU as an order number or manual engineering
acceptance. Never clear an existing cart or delete unrelated items.

Use a task-specific persistent Chromium profile. Let the user sign in visibly;
do not request passwords in chat, extract ordinary browser cookies or copy
someone else's profile. A local CDP connection can reuse that task browser:

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.connect_over_cdp("http://127.0.0.1:9234")
    page = browser.contexts[0].pages[0]
```

Port 9234 is only an example: use the actual task browser endpoint. Bind CDP
locally. Do not close the user's browser when disconnecting a short automation
command. Stop the task browser after handoff if it is no longer needed.

Do not commit profiles, cookies, credentials, raw account-page HTML, shipping
addresses, personal names, raw chats, phone numbers, email addresses or
file-download URLs containing access keys. Capture a cropped project/order row,
not the account dashboard. Store only non-sensitive quote specifications,
prices, filenames, hashes, dates, status, quantities, project numbers and
supplier SKU/reference identifiers. Summarize supplier messages as technical
requirements or unresolved decisions instead of preserving the conversation.

## Prepare fabrication evidence

1. Use the actual validated saved board, not a render or stale fabrication ZIP.
   Run ERC and DRC with all severities and schematic parity where applicable.
   Inspect every report and keep baseline issues distinct; never call a failed
   check clean. Preserve user `.kicad_pro` and editor-local settings.
2. Derive finished dimensions from the Edge.Cuts polygon centreline. KiCad
   plot/job extents can include the outline stroke, overstating each dimension.
   Read actual thickness/layers and keep the original board unchanged.
3. Package one flat upload ZIP per unique design with top/bottom copper, mask,
   silk, sole outline and metric absolute Excellon drills. Use separate PTH and
   NPTH files with clear names; omit genuinely empty drill sets. Internal
   windows/slots must remain in the profile or proper drill/route output.
   Do not duplicate routing in conflicting files. Elecrow accepts ZIP/RAR up
   to 30 MB. RS-274-X Gerbers were successfully uploaded.
4. Include fabrication notes for finish, holes, contacts, tolerances, no artwork
   substitutions and no order marks on functional surfaces. Record raw ZIP
   SHA256 and per-member hashes. Validate archive CRC and drill content.
5. Use per-assembly multiplicities: two identical probe/guide PCBs means twice
   the probe quantity, not two different designs. A bundle containing several
   upload ZIPs is a handoff, not one uploadable PCB design.

Bare PCB only means no component procurement, assembly, BOM/CPL or stencil.
Choose requested solder-mask colour per order; **purple is not a global default**.
Choose finish for the function: flat exposed electrical contacts may require
ENIG rather than uneven HASL. Ordinary ENIG is not hard-gold wear qualification.

## Prepare PCBA evidence

Before submitting an assembly RFQ:

1. Pass the board's ERC/DRC and unconnected-item release gates.
2. Export Gerbers/drills from the validated board revision.
3. Generate BOM and CPL/position data from schematic/PCB sources, not a
   hand-copied interactive BOM.
4. Verify every fitted reference, MPN, quantity, package, side, rotation, and
   coordinate unit. Legacy position files without units must be reconciled
   against the PCB before upload.
5. Exclude tooling holes, logos, test-only no-fit parts, net ties, and other
   project-defined DNP items.
6. Put substitutions, exact-MPN requirements, first-article photos, board
   thickness, mask/silkscreen colours, finish, and functional-contact rules in
   a committed manufacturing note and in the RFQ instructions.
7. Record hashes for the exact Gerber, BOM, CPL, and instruction files.

When several variants share a PCB, use one variant manifest that maps each
supplier reference to its exact BOM, colour, thickness, and quantity. Do not
infer the variant from a generic project name.

## Audit quotations and service orders

Set the service-order table to its largest practical page size before
concluding an item is missing. Audit all statuses, including pending quotation,
quoted, pending payment, production, completed, and canceled.

- Identify jobs by supplier reference and exact uploaded BOM filename. Quoted
  rows may insert a quote-download link before the BOM name, so visible text
  parsing alone is unreliable.
- Submission redirects can lag. Search the service-order table before retrying
  an apparently failed submission; a retry can create a duplicate.
- Treat same-board jobs with different quantities or revisions as distinct
  until their uploaded artifacts are compared.
- Never cancel a suspected duplicate without matching its files, quantity,
  revision, status, and intended retained reference.
- Record cancellations by non-sensitive supplier reference and reason only.

For each final quote, download the quote workbook in the authenticated browser
context and verify:

- board, component, stencil, assembly, freight, tax, and total lines;
- quantity and effective landed cost per board;
- each BOM reference/MPN/quantity group against the submitted BOM;
- substitutions and unavailable parts;
- first-article or photo-approval conditions;
- discrepancies between line sums, BOM footer totals, and quote summaries.

Hash the downloaded workbook and store a PII-free summary. A price mismatch or
unexplained substitution is a release hold, not a rounding detail to ignore.

## Track production and lifecycle

Map supplier state to the product lifecycle without overstating progress:

| Elecrow evidence | Lifecycle state |
| --- | --- |
| RFQ submitted, component cost incomplete | Quote |
| Final quote verified, decision unresolved | Quote |
| Human approved quote, awaiting payment | Payment |
| Paid shop order / supplier production started | Production |
| Shipped or delivered, not yet inspected | Receive and QA |
| Incoming inspection and functional test passed | Listed or Sustaining |
| Canceled duplicate or obsolete build | Done with cancellation evidence |

The lifecycle record may contain supplier reference, quantity, non-sensitive
costs, status, check date, package hashes, and release holds. Do not put
customer, account, delivery, payment, or supplier-person details on a public
issue or project board.

Use `references/audit-record.schema.json` for a portable PII-free audit record.
Set `live_verified` only after checking the authenticated account; a Project
board or prior export is not a substitute for current supplier state.

After QA:

1. Update local and partner stock separately.
2. Reconcile the Tindie/Elecrow listing and price against landed cost.
3. Enable advertising only when the public listing is purchasable and stock is
   positive.
4. Retain calibration, test, yield, and revision evidence for the next batch.

## Prioritize the pipeline

Use pull-based ordering rather than processing quotes in submission order.
Prioritize:

1. paid/in-production jobs needing a technical answer or incoming QA;
2. payment-ready replenishment of proven, out-of-stock sellers;
3. low-capital quotes for products with recent demand;
4. named research pilots with explicit users and release evidence;
5. speculative variants only after earlier batches validate demand.

Hold or close duplicate, obsolete, unexplained-price, unvalidated, and
single-unit quotes whose economics do not represent the intended batch.

## Proven browser workflow

Inspect the live DOM before reusing selectors; the examples below were observed
on the Elecrow FR-4 form. Its scripts initialize on `window.onload`.
`domcontentloaded` alone is too early: selecting a file before the change handler
exists silently leaves the UI unchanged.

```python
page.goto(
    "https://www.elecrow.com/pcb-manufacturing.html",
    wait_until="load",
)
page.wait_for_function(
    '()=>window.jQuery && '
    'jQuery._data(document.querySelector("#pcbFile"),"events")?.change?.length>0'
)
with page.expect_file_chooser() as chooser:
    page.locator("#addGerber").click()
chooser.value.set_files(str(upload_zip.resolve()))
```

The upload requires login: selecting a ZIP while signed out redirects to
Customer Login. Resume after user authentication; do not loop or bypass it.
Confirm `.file_accept` contains the exact selected filename.

The Gerber preview service may report it cannot preview images. This does not
establish a malformed manufacturing ZIP, but it does not prove DFM either.
Keep the warning, manually enter the actual outline dimensions and verify the
uploaded archive later. Do not fix a preview by changing correct PCB geometry.

### Observed FR-4 controls

| Setting | Selector / action |
| --- | --- |
| FR-4 | `label[for=material1]` |
| Two layers | `label[for=model2]` |
| Width / height | Fill `#long`, `#wide`, then press Tab to emit change |
| Quantity dropdown | Click `#form1 .qty_de`, then its quantity label |
| Five / ten boards | `label[for=selCheck1]` / `label[for=selCheck11]` |
| One unique design per ZIP | `label[for=design1]` |
| 1.6 mm finished thickness | `label[for=thickness5]` |
| Green / purple mask | `label[for=color1]` / `label[for=color7]` |
| White silkscreen | `label[for=screen1]` |
| ENIG / immersion gold | `label[for=finish3]` |
| Lead-free HASL (not leaded HASL) | `label[for=finish2]` |
| No castellations | `label[for=hole1]` |
| 1 oz outer copper | `label[for=weight1]` |
| Ordinary 3-5 working days | Select value `35` in `#formTime` |
| Destination country | Select actual destination code in `#formCountry` |
| Add quoted job | Exact button name `Add To Cart` |

Quantity choices are hidden until `.qty_de` opens the dropdown; `.sel_text`
does not exist. Use real labels rather than forcing hidden radio elements.
No-stencil state was shown by hidden `.check` and no stencil item in the cart;
inspect current UI rather than assuming a checkbox API.
Wait for asynchronous pricing and production-time options to settle. Verify
checked values and displayed dimensions/filename before adding each job.

If existing quotes have no edit action, replace only their identified cart rows
through each row's `a[title=Remove]`; never clear the entire cart. Regenerate
and re-upload ZIPs when fabrication notes change, so the file does not contradict
the finish selected in the form. Verify replacement rows and uploaded hashes
again. Respect an explicit lower-cost lead-free HASL choice while recording
its contact-flatness limitations; do not silently keep ENIG.

## Verify server-side outcome

After adding, wait for `/checkout/cart`. For every intended job, locate its
`tbody.cart.item` row by exact ZIP filename. Require:

- Correct PCB quantity, dimensions, layers, finished thickness, mask/silk,
  surface finish and copper weight. Cart line quantity is normally one batch;
  PCB quantity is inside that batch's options. Do not multiply both.
- Correct file and unique design count; no assembly/stencil or unrelated additions.
- Cart SKU and actual discounted line price, currency and subtotal recorded.
  Shipping/tax remain provisional unless separately confirmed.
- Reloaded cart retains the jobs; inspect before retrying an ambiguous add so
  a timeout does not create duplicates.

When the cart offers a download link for the uploaded fabrication archive,
use that authenticated browser context to download it and compare its raw
SHA256 to the submitted ZIP. Do not print or commit the signed/access-key URL:

```python
row = page.locator("tbody.cart.item").filter(has_text=upload_zip.name)
link = row.get_by_role("link", name=upload_zip.name, exact=True)
response = page.request.get(link.get_attribute("href"))
assert response.ok
assert hashlib.sha256(response.body()).hexdigest() == expected_sha256
```

This proves the uploaded artifact matches; it is not supplier DFM approval.
Include critical fabrication requirements inside the ZIP even when the cart
has no notes field. Record whether a manual engineering review was actually
requested; an automatic cart price alone is not that request.

For quote-only tasks, leave the verified jobs in the authenticated cart and
report prices plus unpaid status. Never click checkout/order/payment merely
to obtain an order reference. Further actions require the corresponding user
authorization and a final check of addresses, quantities, costs and terms.
