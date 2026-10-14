# Quick Start: turn an HL7 admission into procedure and insurance rows

In about 5–10 minutes, read a fictional hospital admission and produce four
business rows: two procedures and two insurance plans. A second CSV shows the
underlying HL7 segments, loop boundaries, and composite fields. No database,
clinical system, or coding is needed to run the demo.

## Before you start

Use Apache Hop 2.19.0, Java 21 or later, and the **HL7-enabled BerryWorks EDI Input
preview build**. The older 0.1.0 preview does not support this example. Use the
new preview ZIP when available; the existing installation procedure is described
in the [repository README](../README.md#manual-zip-installation).

Download this repository using GitHub **Code → Download ZIP**, extract it, and
keep the entire `quick-start-hl7` folder together in a writable location. This
demo includes its own ADT_A01/2.5.1 model, so it does not depend on the models
folder in your Hop installation. It contains no ORU model.

## Run it in Hop

1. Open Hop GUI and open `pipelines/admission-summary.hpl` inside this folder.
2. Click **Run** and choose your **local** pipeline run configuration.
3. On the **Parameters** tab, set `HL7_QUICK_START_HOME` to the **absolute path
   of this quick-start-hl7 folder**, without quotes or a `file:` prefix. For
   example, `/Users/you/Downloads/hop-edi-main/quick-start-hl7` or
   `C:\Users\you\Downloads\hop-edi-main\quick-start-hl7`.
4. Run the pipeline and confirm there are no errors.
5. Open `output/admission-items.csv` and `output/parser-rows.csv` in a text
   editor or spreadsheet. Each run overwrites both files.

The `expected` folder contains reference results. If the models directory cannot
be found, check that the parameter points to this folder, not `pipelines`.

## What you should see

`admission-items.csv` has four rows for fictional patient **DEMO-001**, **Alex
Example**:

| Group | Loop instance | Code | Description | Organization |
|---|---:|---|---|---|
| Procedure | 1 | DEMO-P1 | Synthetic procedure one | |
| Procedure | 2 | DEMO-P2 | Synthetic procedure two | |
| Insurance | 3 | DEMO-PLAN-1 | Primary demo plan | EXAMPLE PRIMARY INSURER |
| Insurance | 4 | DEMO-PLAN-2 | Secondary demo plan | EXAMPLE SECONDARY INSURER |

`parser-rows.csv` contains 30 data rows for the supplied sample. Find the four `LOOP_START` rows and their matching
`LOOP_END` rows:

- Instances 1 and 2 use `PR1-ADT_A01-PROCEDURE`.
- Instances 3 and 4 use `IN1-ADT_A01-INSURANCE`.
- The first insurance instance contains IN1, IN2, and IN3; the second contains
  IN1 and IN2. These supporting segments retain their owning loop instance.

The groups are siblings, so `parent_loop_instance` is empty. A `COMPOSITE` row
points to its owning row through `parent_sequence_number`; `element_position`
identifies the field being expanded. For example, PID field 5 holds the patient
name, and IN1 field 2 holds the plan code and description.

## Try a change

Open `input/admission.hl7` in an editor that preserves its carriage-return
segment endings. Change `Secondary demo plan` to `Supplemental demo plan`, save,
and run again. Only the second insurance description should change in the
business output. Restore the original text afterward.

Alternatively, preview **Read HL7 admission** in Hop to explore all parser rows,
or preview **Extract procedures and insurance** to see the four business rows.

## How the pipeline works

1. **Read HL7 admission** parses the message using the supplied YAML model and
   copies every row to both branches.
2. **Write parser rows** saves the structural view, including loop and composite
   relationships and the first fifteen element columns.
3. **Extract procedures and insurance** carries patient context forward, reads
   procedure and plan composites, and emits one business row at each loop end.
4. **Write admission items** saves those four rows as CSV.

For someone new to HL7: `ADT^A01` identifies an admission/visit notification.
`MSH` is the message header, `EVN` describes the event, `PID` identifies the
patient, and `PV1` describes the visit. `PR1` begins each procedure group; `IN1`
begins each insurance group. `IN2` and `IN3` supply additional insurance details.
A pipe separates fields, and a caret separates components within a field.

A group can be optional while its first segment is required whenever the group
appears. ADT_A01's PROCEDURE and INSURANCE groups both have required first
segments, making their repeated instances recognizable by the current detector.

For someone new to Hop: a transform is one processing step, and an arrow carries
rows between steps. Double-click a transform to inspect it. The JavaScript mapping
is already configured; editing it is optional.

## Scope of this example

All names, identifiers, procedures, and insurance details are synthetic. MSH-11
is `T` for test. This is a parsing demonstration, not a clinically validated
admission record, an HL7 compliance validator, or a production patient mapping.

The mapping assumes one admission, one patient identifier, one patient name,
and the two sibling group types shown here. It does not handle arbitrary patient
identifier/name repetitions or perform clinical interpretation. It writes files
only; it does not send messages to a hospital or other service.

The model was generated from HAPI HL7v2 2.6.0's HL7 2.5.1 structure using the
framework's `hl7-model-poc` tool. HAPI is not a plugin runtime dependency. The
model retains structural and field metadata, but does not provide complete
nested-datatype validation, conditional rules, or vocabulary validation.

This preview demonstrates model-based HL7 loops with ADT_A01. ORU structures
with optional leading items or groups beginning with nested groups require
further loop-detection work; ORU is not included in this demo.

## Optional: make it a Hop project

Register a project with this `quick-start-hl7` folder as its home and the supplied
`project-config.json` as its project configuration. It includes a `local` run
configuration under `metadata`. When the project is active,
`HL7_QUICK_START_HOME` defaults to `${PROJECT_HOME}`.
