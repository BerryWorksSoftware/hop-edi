# BerryWorks EDI Input for Apache Hop

A free EDI input plugin for Apache Hop, developed by BerryWorks Software.
It reads X12 and EDIFACT messages and emits rows that Hop pipelines can process.
Model-based loop detection uses BerryWorks YAML EdiModels.

Version **0.1.0** is a preview release. This repository contains public release
information and downloads; the development source is maintained separately.

## Requirements

- Apache Hop 2.19.0
- Java 21 or later

## Download and install

Download the obfuscated plugin ZIP and its SHA-256 checksum from
[GitHub Releases](https://github.com/BerryWorksSoftware/hop-edi/releases) when the
preview release is published. Release asset: `hop-edi-plugin-0.1.0.zip`.

1. Stop the Hop GUI and any other Hop processes using the installation.
2. If upgrading, remove `plugins/transforms/hop-edi-plugin/` from the Hop
   installation. Also remove the legacy `plugins/transforms/hop-transform-sample/`
   folder if present. Do not retain old plugin or framework JARs.
3. Extract the ZIP into the **Hop installation root**, the folder containing
   `hop-gui.sh` or `hop-gui.bat` and the `plugins` directory.
4. Restart Hop. In a pipeline, find **EDI Input** under **BerryWorks EDI**.

The installed plugin JAR is
`plugins/transforms/hop-edi-plugin/hop-edi-plugin.jar`.

Instructions for installation through Hop's Marketplace GUI will be added with
the repository definition. The preview is not yet listed in the default
Marketplace catalog.

## Configure EDI Input

Set **EDI filename** to your input file and **Models directory** to the absolute
path of the bundled models folder in your Hop installation:

```text
plugins/transforms/hop-edi-plugin/models/
```

The preview includes these models:

| Standard | Message | Version | Model file |
|---|---|---|---|
| X12 | 850 Purchase Order | 004010 | `850-004010.yaml` |
| EDIFACT | ORDERS Purchase Order | D96A | `ORDERS-D96A.yaml` |

Alternatively, copy the YAML files into your project's `edi-models` directory;
the transform's default Models directory is `${PROJECT_HOME}/edi-models`.
Use only one model for each message type and version in the selected folder.
A matching model is required for model-based loop detection. Example pipelines
and EDI input files are not bundled.

## Preview behavior and limitations

The transform emits `ENVELOPE`, `SEGMENT`, `LOOP_START`, `LOOP_END`, and `COMPOSITE`
rows. Composite rows identify their owning row using `parent_sequence_number`
and `element_position`.

Recoverable parser errors are logged and parsing continues; fatal errors fail
the transform. This preview performs parsing, not full EDI compliance validation.
Trailer values are reconstructed.

Explicit EDIFACT UNG/UNE groups, full UNB06 and UNH04 values, repeated elements,
and nested composites are not supported. TRADACOMS and HL7 are outside the
supported preview scope.

## Components and licenses

The distribution contains the Hop integration plugin, the unchanged public
[BerryWorks EDIReader parser](https://github.com/BerryWorksSoftware/edireader),
a reduced and obfuscated proprietary BerryWorks EDI Framework runtime, SnakeYAML,
ESD resources, and the two YAML models.

The Hop-derived plugin code is covered by Apache License 2.0. EDIReader and
third-party components retain their own licenses. The proprietary framework is
licensed for use with this Hop plugin; its bundled terms prohibit redistribution.
See `NOTICE.txt` and the `licenses` directory inside the installed plugin for
component notices and license terms.

## Feedback

Report preview issues through this repository's
[issue tracker](https://github.com/BerryWorksSoftware/hop-edi/issues).
Include your Hop and Java versions, the error message, and a small reproducible
example with sensitive information removed.
