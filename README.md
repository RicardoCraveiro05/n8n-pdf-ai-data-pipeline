# n8n PDF AI Data Pipeline

<p align="center">
  🇧🇷 <a href="README.pt-br.md">Português</a> &nbsp;|&nbsp;
  🇺🇸 <strong>English</strong>
</p>

## Overview

An automated **PDF data extraction and ETL pipeline** built with **n8n**, **Anthropic Claude**, **Google Drive**, and **Google Sheets**.

The workflow reads a PDF report, locates the target table, extracts structured records with AI, validates the returned JSON, transforms the records into a standardized structure, checks for duplicate reports, and loads validated data into Google Sheets.

The project demonstrates a practical application of **AI-assisted document processing, ETL, data quality, deduplication, and workflow automation**.

## Architecture

```text
Google Drive
     │
     ▼
PDF Document
     │
     ▼
Deduplication Check
     │
     ├── DUPLICATE ──► Notification
     │
     └── NEW
          │
          ▼
   Anthropic Claude
          │
          ▼
   Data Validation
          │
          ├── FAILED ──► Error Notification
          │
          └── PASSED
                │
                ▼
        Record Transformation
                │
                ▼
        Google Sheets
                │
                ▼
       Processing Notification
```

## Technologies

- **n8n** — workflow orchestration
- **Anthropic Claude** — AI-assisted PDF extraction
- **Google Drive** — document source
- **Google Sheets** — destination
- **JavaScript** — validation and transformation
- **JSON** — structured data contract
- ETL
- Data quality validation
- Deduplication

## Data Extraction

The AI extraction step targets the PDF section **"2. Registros de Atendimento"** and returns:

```json
{
  "atendimentos": [
    {
      "cnes": "...",
      "placa": "...",
      "tipologia": "...",
      "compet": "...",
      "uf": "...",
      "municipio": "...",
      "oci": "...",
      "quantidade": 0
    }
  ]
}
```

Each table row represents exactly one attendance record.

The extraction prompt explicitly instructs the model to:

- Extract all rows.
- Avoid grouping records.
- Avoid calculations.
- Avoid inference.
- Preserve values found in the PDF.
- Return only the expected JSON structure.

## Data Validation

The `VALIDATE — Data Quality` node verifies the AI response before the data continues through the pipeline.

It checks:

- Valid JSON.
- Object structure.
- Presence of the `atendimentos` array.
- Required fields.
- Empty strings.
- Quantity type and value.
- Unexpected fields.
- At least one extracted record.

This creates a validation layer between AI extraction and data loading.

## Data Transformation

After validation, the workflow creates one n8n item per attendance record.

The normalized output contains:

```text
cnes
placa
tipologia
compet
uf
municipio
oci
qtd
atendimento
id_relatorio
```

Technical metadata can also be attached to the processing records.

## Deduplication

Before sending the PDF to the AI extraction stage, the workflow checks whether the report has already been registered.

### New document

```text
NEW
 ↓
Process PDF
 ↓
Extract records
 ↓
Validate
 ↓
Transform
 ↓
Load to Google Sheets
```

### Previously processed document

```text
DUPLICATE
 ↓
Skip processing
 ↓
Send notification
```

This prevents repeated processing of the same report.

## Error Handling

If the AI response does not satisfy the expected data contract, the workflow routes the execution to the validation error path instead of loading invalid records.

The workflow therefore separates:

```text
Extraction
    ↓
Validation
    ↓
Transformation
    ↓
Loading
```

## Repository Structure

```text
n8n-pdf-ai-data-pipeline/
│
├── workflow/
│   └── pdf-ai-extraction.json
│
├── screenshots/
│
├── README.md
├── README.pt-br.md
└── .gitignore
```

## Setup

1. Import `workflow/pdf-ai-extraction.json` into n8n.
2. Configure Google Drive credentials.
3. Configure Google Sheets credentials.
4. Configure the Anthropic credential.
5. Configure Gmail if email notifications are required.
6. Replace the placeholder IDs and email address with your own values.
7. Review the Google Sheets columns.
8. Test the workflow with a sample PDF.
9. Test both `NEW` and `DUPLICATE` scenarios.
10. Activate the workflow after validation.

> The workflow included in this repository is a sanitized portfolio version. Real credentials, private documents, and personal account identifiers are intentionally excluded.

## Portfolio Context

This project demonstrates practical skills in:

- Data Analytics
- Business Intelligence
- ETL
- Data Quality
- AI-assisted data processing
- Workflow Automation
- JSON transformation
- Business-oriented data pipelines

## License

This project is available for educational and portfolio purposes.
