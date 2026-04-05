# README #

# SwaggerToCFP

Automated COSMIC functional size measurement tool for OpenAPI REST API specifications.

**SwaggerToCFP** reads one or more OpenAPI 3.x specifications (YAML) and produces a structured CSV
report containing the COSMIC functional size (CFP – COSMIC Function Points) of each measured API,
as described in ISO/IEC 19761.

> Developed as part of a doctoral research project at UQAM (Université du Québec à Montréal).
> Full methodology and evaluation results are available in the associated thesis:
> Lapointe-Boisvert, A. (2026). *Automatisation de la mesure de taille fonctionnelle COSMIC
> ISO 19761 à partir de spécifications OpenAPI dans un contexte DevOps*. Doctorat en
> informatique, UQAM.

---

## How it works

The tool applies a three-step pipeline:

1. **Parsing** – Validates and deserializes the OpenAPI YAML file using the `openapi3` library
   (Orth, 2024), ensuring structural conformance before any measurement.
2. **Mapping** – Maps OpenAPI elements to COSMIC concepts (ISO 19761, v5.0):

   | OpenAPI element | COSMIC concept |
   |---|---|
   | HTTP operation (`GET`, `PUT`, `POST`, `DELETE`…) | Functional Process (FP) |
   | `components/schemas` object with named properties | Data Group (DG) |
   | Property of a Schema Object | Data Attribute |
   | `requestBody` referencing a schema | Entry data movement (E) |
   | `responses` (2xx) referencing a schema | Exit data movement (X) |

3. **Sizing** – Aggregates data movements per functional process and outputs the CFP count.

Reads, Writes (internal persistence) and standard HTTP error responses (4xx/5xx gateway codes
without application schema) are intentionally excluded from the measurement scope, as they are
not observable from the OpenAPI interface contract alone (see thesis §3.3.2 for the full
measurement strategy).

---

## Requirements

- Python 3.12.1+
- Dependencies declared in `requirements.txt`:
  - [`openapi3`](https://github.com/Dorthu/openapi3) – OpenAPI 3.x parser and validator
  - [`PyYAML`](https://pyyaml.org/) – YAML deserializer

---

## Installation

```bash
git clone https://github.com/alapointe/SwaggerToCFP.git
cd SwaggerToCFP
pip install -r requirements.txt
```

> **Note on Swagger 2.0 files:** The tool processes OpenAPI 3.x only. Swagger 2.0 specs must
> be converted beforehand using the
> [Swagger Converter](https://converter.swagger.io/) (SmartBear, 2024).

---

## Usage

1. Copy one or more `.yaml` OpenAPI specification files into the `specs/` directory.
2. Run the tool:

```bash
python main.py
```

3. The output file `.result.csv` is generated in the project root. It contains, for each API:

| Column | Description |
|---|---|
| `api_name` | Name of the OpenAPI specification file |
| `fp_id` | Functional Process identifier (`summary.METHOD.last-path-segment`) |
| `data_groups` | List of identified Data Groups (DG) |
| `data_movements` | Detail of Entry (E) and Exit (X) data movements |
| `cfp` | COSMIC Function Points for this functional process |
| `total_cfp` | Total CFP for the full specification |

### Example

Using the [Swagger Pet Store](https://petstore3.swagger.io/) specification:

```bash
# Place petstore.yaml in specs/
python main.py
# → .result.csv generated
```

---

## Running tests

```bash
python -m unittest discover -s tests
```

Code quality is continuously monitored with [SonarQube](https://www.sonarsource.com/).
Static analysis results are available in Annex A of the thesis.

---

## Validation

The tool was evaluated on **72 functional processes** across **16 industrial REST APIs** from a
Canadian financial institution, measured manually by two certified COSMIC experts.

| Metric | Result |
|---|---|
| Functional accuracy | 100% |
| Repeatability | Deterministic – identical output across runs |
| Reproducibility | Confirmed on independent environments |
| Throughput | 10.16 CFP/ms (vs. 30.6 CFP/h manually) |
| Efficiency gain | 99.99% |

The tool was also successfully applied to the Microsoft Graph API
(1,012,642 SLOC → **45,729 CFP** in a few seconds).

---

## Known limitations

- **Interface boundary only** – Reads, Writes and cross-service exchanges are excluded (not
  observable from OpenAPI alone).
- **OpenAPI 3.x only** – Swagger 2.0 requires prior conversion.
- **No graphical interface** – CLI only; output is a CSV file compatible with Excel, Google
  Sheets, or any BI tool.
- Measurement scope and known methodological limits are discussed in thesis §6.3.

## Contact

Alexandra Lapointe-Boisvert – lapointe.alexandra@gmail.com
