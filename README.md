🇧🇷 [Português](README.pt-br.md) | 🇺🇸 **English**

# n8n PDF AI Data Pipeline

An automated **PDF data extraction and ETL pipeline** built with **n8n**, **Anthropic Claude**, **Google Drive** and **Google Sheets**. It reads a PDF report, extracts the records of a target table with AI, validates the result, checks for duplicates, and loads clean data into a spreadsheet, with an email notification for every outcome.

![Workflow overview](main/Workflow - N8N.jpeg)

> **Note:** every document and value in this repository is **fictitious**, created only to demonstrate the pipeline.

## The problem

Copying the data of PDF reports into a spreadsheet by hand is slow and error-prone. Two things make it worse: the same file can be processed twice, and bad data can reach the spreadsheet unnoticed. This pipeline automates the extraction and adds guardrails so that what lands in the sheet can be trusted.

## Result

In the original real-world use case, filling the spreadsheet by hand from a batch of 14 PDFs took about **3 hours**. With this pipeline, the same batch takes about **15 minutes**.

| | Manual process | With the pipeline |
|---|---|---|
| Batch of 14 PDFs | ~3 hours | ~15 minutes |
| Per document | ~13 minutes | ~1 minute |
| Time reduction (automated processing) | n/a | **~92%** |
| Time reduction (including human review) | n/a | **~70%** |

The pipeline is designed to work with a human in the loop: the ~92% figure covers the automated processing, and the ~70% figure is the end-to-end gain once the human review of the results is counted.

*These figures come from the original use case. This repository uses fictitious data only.*

## How it works

| Stage | Node | What it does |
|---|---|---|
| **Ingest** | `INGEST — PDF Document` | Downloads the PDF from Google Drive |
| **Dedup** | `DEDUP — Check Sheet` → `DEDUP — Is Duplicate` → `IF — Duplicate?` | Looks for the file's Drive ID in the destination sheet. If it is already there, notifies by email and stops |
| **Extract** | `AI — Document Extraction` | Sends the PDF to Claude, which returns the table rows as structured JSON |
| **Validate** | `VALIDATE — Data Quality` → `IF — Validation Passed` | A JavaScript step checks the AI output before anything is written |
| **Transform** | `TRANSFORM — Normalize Records` | Creates one normalized record per table row and attaches technical metadata |
| **Load** | `LOAD — Google Sheets` | Appends the records to the destination sheet |
| **Notify** | `NOTIFY — ...` | Sends an email for each outcome: already processed, completed, or validation failed |

### Three possible outcomes

1. **Duplicate:** the document was already processed → "Already Processed" email, and no AI call is made.
2. **Validation failed:** the extraction did not meet the quality checks → error report + "Validation Failed" email listing the problems. Nothing is written to the sheet.
3. **Success:** records are normalized, appended to Google Sheets, and a single "Processing Completed" email is sent.

## Data contract

The AI step targets the table of the section **"2. Registros de Atendimento"** and must return exactly this JSON (field names are in Portuguese because the source documents are in Portuguese):

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

Each table row becomes exactly one object. The prompt tells the model to extract every row, never group, sum or infer values, use `null` when a field cannot be identified with confidence, and return only JSON.

## Validation

`VALIDATE — Data Quality` checks the AI response before it continues through the pipeline:

- the response is valid JSON (Markdown fences are stripped if the model adds them)
- the root is an object with an `atendimentos` array containing at least one record
- every required field is present, and none is an empty string
- `quantidade` is a non-negative integer
- there are no unexpected fields
- no record is entirely empty

If any check fails, the execution is routed to the error path instead of the load step.

## Deduplication

The Drive file ID is stored in the `id_relatorio` column of every loaded row. Before calling the AI, the workflow reads the sheet and checks whether that ID is already there. This cuts duplicates early, so no AI call is spent on a document that was already handled.

## Design decisions

- **Dedup runs before the AI step.** It avoids repeated processing and unnecessary AI cost.
- **AI output is never trusted blindly.** A plausible-looking but wrong extraction is worse than a failed one, so validation sits between extraction and loading.
- **Failures are explicit.** Every branch ends in a notification, so nothing fails silently.
- **Stages are separated** (extract → validate → transform → load), which makes each one testable and easy to replace.
- **Descriptive node names** (`STAGE — action`) keep the flow readable at a glance.

## Tech stack

- [n8n](https://n8n.io/): workflow orchestration
- Anthropic Claude: AI-assisted PDF extraction (the workflow is set to `claude-sonnet-5-5`; other Claude models that accept PDF input should also work)
- Google Drive, Google Sheets, Gmail
- JavaScript: Code nodes for dedup, validation and transformation

## Repository structure

```
n8n-pdf-ai-data-pipeline/
├── README.md
├── README.pt-br.md
├── docs/
│   └── workflow.jpeg
├── workflow/
│   └── pdf-ai-extraction.json
├── samples/
│   └── (fictitious sample PDF)
└── .gitignore
```

## Setup

1. Import [`workflow/pdf-ai-extraction.json`](workflow/pdf-ai-extraction.json) into n8n.
2. Create and select your own credentials for Google Drive, Google Sheets, Gmail and Anthropic.
3. Create a Google Sheet with a tab named `Data` and this header row:
   `cnes | placa | tipologia | compet | uf | municipio | oci | qtd | atendimento | id_relatorio | file_name | data_processamento | valid`
4. Replace the placeholders `YOUR_GOOGLE_DRIVE_FILE_ID` and `YOUR_GOOGLE_SHEETS_ID` (in the Drive node and in both Sheets nodes) and `your-email@example.com` (in the three Gmail nodes).
5. Upload the sample PDF from [`samples/`](samples/) to your Drive.
6. Test the three scenarios:
   - **New document:** run it once and check the rows and the success email.
   - **Duplicate:** run the same file again; it must stop with the "Already Processed" email.
   - **Validation failure:** run it with a PDF that does not contain the expected table, and check the failure email.

> This is a sanitized portfolio version. Credentials, real documents and personal identifiers are intentionally excluded.

## Limitations and roadmap

- The trigger is manual and processes one Drive file at a time. A Drive trigger or a folder loop would automate the whole batch.
- [ ] Equivalent implementation in Activepieces
- [ ] Python version of the pipeline
- [ ] Comparison of the three approaches

## Skills demonstrated

ETL · AI-assisted document processing · data quality validation · deduplication · workflow automation · JSON data contracts

## License

Available for educational and portfolio purposes.

## Author

**Ricardo**: BI & Automation · [GitHub](https://github.com/RicardoCraveiro05)
