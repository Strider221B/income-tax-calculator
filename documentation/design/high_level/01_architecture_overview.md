# Architecture Overview — Indian Income Tax Calculator (ITR-2, New Regime)

> ⚠️ **DISCLAIMER**: This tool generates AI-assisted tax calculations and is prone to errors.
> All calculations, values, and details MUST be independently verified by the user before filing.
> The developers and AI assume no liability for incorrect calculations or missed compliance requirements.

---

## 1. System Goals

| Goal | Description |
|------|-------------|
| **Automated ITR-2 Report Generation** | Parse financial documents and generate a comprehensive ITR-2 (New Tax Regime) calculation report |
| **Multi-Schedule Coverage** | Support all 20+ ITR-2 schedules including Salary, HP, CG, OS, CYLA, BFLA, CFL, VI-A, 80G, 80GGA, AMT, AMTC, SI, EI, PTI, FSI, TR, FA, AL, TDS, and Form 67 |
| **Document Intelligence** | Use Gemini AI for PDF/Excel parsing, OCR, and data extraction from financial documents |
| **Prior Year Bootstrapping** | Pre-populate from previous year's ITR-2 preview, then overlay current year data |
| **Regulatory Awareness** | Search for and summarize ITR changes for the current financial year |
| **Triple Output** | Excel workbook (calculations), detailed Markdown report, and a quick-reference copy-paste Markdown |
| **Compliance Verification** | Cross-verify against AIS, TIS, and Form 26AS |

---

## 2. High-Level Architecture

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│                              USER (CLI Interface)                                │
│  $ itrcalc run --input ./documents --output ./reports --fy 2025-26              │
└──────────────────┬───────────────────────────────────────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────────────────────────────────────┐
│                           CLI ORCHESTRATOR (main.py)                              │
│  ┌─────────────┐  ┌─────────────────┐  ┌──────────────┐  ┌──────────────────┐   │
│  │  Arg Parser  │  │ Config Manager  │  │  Env Loader  │  │ Progress Display │   │
│  │  (click)     │  │ (.env, YAML)    │  │  (dotenv)    │  │ (rich)           │   │
│  └─────────────┘  └─────────────────┘  └──────────────┘  └──────────────────┘   │
└──────────────────┬───────────────────────────────────────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────────────────────────────────────┐
│                          PIPELINE ENGINE (pipeline.py)                            │
│                                                                                  │
│  Stage 1: Document Discovery & Validation                                        │
│  Stage 2: Previous Year ITR-2 Parsing (Bootstrap)                                │
│  Stage 3: Current Year Document Extraction                                       │
│  Stage 4: Data Reconciliation & Delta Report                                     │
│  Stage 5: External Data Fetch (TTBR rates, ITR changes)                          │
│  Stage 6: Schedule Computation Engine                                            │
│  Stage 7: Cross-Verification (AIS, TIS, 26AS)                                   │
│  Stage 8: Report Generation (Excel + Markdown × 2)                               │
│  Stage 9: User Review & Interactive Amendments                                   │
└──────────────────┬───────────────────────────────────────────────────────────────┘
                   │
        ┌──────────┼──────────┬──────────────┬──────────────┐
        ▼          ▼          ▼              ▼              ▼
┌────────────┐ ┌────────┐ ┌──────────┐ ┌──────────┐ ┌───────────┐
│  Document  │ │ Gemini │ │ Schedule │ │ External │ │  Report   │
│  Parsers   │ │ AI     │ │ Calc     │ │ Data     │ │ Generator │
│  Layer     │ │ Layer  │ │ Engine   │ │ Fetcher  │ │ Layer     │
└────────────┘ └────────┘ └──────────┘ └──────────┘ └───────────┘
```

---

## 3. Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Language** | Python 3.11+ | Core application |
| **CLI Framework** | `click` | Command-line interface, argument parsing, interactive prompts |
| **AI / LLM** | Google Gemini API (`google-genai`) | PDF text extraction, document understanding, web search |
| **PDF Parsing** | `PyMuPDF` (fitz), Gemini Vision | Extract text and tables from PDF documents |
| **Excel I/O** | `openpyxl` (read/write), `pandas` | Parse input Excel files, generate output workbook |
| **Web Scraping** | `requests`, `beautifulsoup4` | SBI TTBR rates, regulatory updates |
| **Configuration** | `python-dotenv`, `pyyaml` | Environment variables (.env), user config (YAML) |
| **Data Models** | `pydantic` | Strict schema validation for all schedule data |
| **Progress/UI** | `rich` | Terminal progress bars, tables, formatted output |
| **Markdown** | `jinja2` | Template-based markdown report generation |
| **Testing** | `pytest` | Unit and integration tests |
| **Logging** | `logging` (stdlib) | Structured logging with configurable verbosity |

---

## 4. Environment Configuration

The application requires a `.env` file at the project root:

```env
# ─── Required ───────────────────────────────────────────────────────
GEMINI_API_KEY=your-gemini-api-key-here

# ─── Optional ───────────────────────────────────────────────────────
# Financial year being computed (default: auto-detect based on current date)
FINANCIAL_YEAR=2025-26

# Assessment year (derived from FY if not set)
ASSESSMENT_YEAR=2026-27

# Log level: DEBUG, INFO, WARNING, ERROR
LOG_LEVEL=INFO

# Output directory (default: ./output)
OUTPUT_DIR=./output
```

> **Security Note**: The `.env` file MUST be listed in `.gitignore`. The application MUST
> error out if `GEMINI_API_KEY` is not set. No hardcoded fallback secrets are permitted.

---

## 5. Core Component Descriptions

### 5.1 CLI Orchestrator (`cli/`)
- Entry point for the application
- Subcommands: `run` (full pipeline), `validate` (check documents only), `search` (ITR changes), `init` (create folder structure)
- Interactive prompts for missing information (schedule selection, new accounts, etc.)
- Rich terminal output with progress tracking

### 5.2 Pipeline Engine (`pipeline/`)
- Defines and executes the 9-stage processing pipeline
- Each stage is independently testable and restartable
- Maintains a pipeline state object that flows through all stages
- Emits structured logs and progress events

### 5.3 Document Parsers (`parsers/`)
- **PDF Parser**: Extracts text, tables, and structured data from PDFs (Form 16, 1042-S, AIS, etc.)
- **Excel Parser**: Reads bank statements, pre-calculated foreign income sheets, interest certificates
- **ITR Preview Parser**: Specialized parser for previous year's ITR-2 preview document
- Uses Gemini AI for complex/scanned documents where direct text extraction fails

### 5.4 Gemini AI Layer (`ai/`)
- Centralized Gemini API client with rate limiting and retry logic
- Functions: document text extraction, table parsing, web search (ITR changes), data validation
- Prompt templates stored as separate files for maintainability
- All AI outputs are logged for auditability

### 5.5 Schedule Computation Engine (`schedules/`)
- One module per ITR-2 schedule (e.g., `schedule_salary.py`, `schedule_cg.py`)
- Each schedule module implements a standard interface: `compute(data) -> ScheduleResult`
- Handles inter-schedule dependencies (e.g., CYLA depends on CG + OS + HP)
- Tax rate tables and slab configurations are externalized in YAML

### 5.6 External Data Fetcher (`external/`)
- **SBI TTBR Rates**: Scrapes/fetches telegraphic transfer buying rates for USD→INR conversion
- **ITR Regulatory Changes**: Uses Gemini search to find and summarize changes for the current FY
- All fetched data is cached locally to avoid repeated network calls
- Each conversion includes the rate and date used in the final report

### 5.7 Report Generator (`reports/`)
- **Excel Generator**: Multi-sheet workbook with one sheet per schedule, summary sheet, and calculation audit trail
- **Detailed Markdown Report**: Full narrative report with all calculations, section references, and explanations
- **Copy-Paste Markdown**: Condensed per-section values formatted for direct entry into the IT portal
- Uses Jinja2 templates for consistent formatting
- All reports include the AI disclaimer warning

---

## 6. Security Considerations

| Concern | Mitigation |
|---------|-----------|
| **API Key Storage** | `.env` file only; never committed to VCS; application errors on missing key |
| **PII in Documents** | All processing is local; no data sent externally except to Gemini API for extraction |
| **PII Masking in Logs** | PAN, Aadhaar, bank account numbers are masked in log output (e.g., `XXXXX1234`) |
| **File Path Security** | All file paths are resolved and validated against the input directory boundary |
| **Network Requests** | HTTPS only for all external API calls; SBI TTBR scraping uses verified URLs |
| **Gemini API** | Requests include only document content needed for extraction; no credentials sent |
| **Output Files** | Reports contain sensitive financial data; user warned to secure output directory |

---

## 7. Diagram Key

```
┌──────────┐
│ Component │  = Application module / layer
└──────────┘

    ──▶       = Data flow direction

    ───       = Association / dependency
```

---

*Next: See [02_data_flow.md](./02_data_flow.md) for detailed data flow diagrams.*
*See [03_folder_structure.md](./03_folder_structure.md) for input/output folder conventions.*
*See [04_schedule_engine.md](./04_schedule_engine.md) for schedule computation details.*
*See [05_document_parsers.md](./05_document_parsers.md) for document parsing architecture.*
*See [06_report_generation.md](./06_report_generation.md) for output format specifications.*
*See [07_security_design.md](./07_security_design.md) for security architecture details.*
