# Folder Structure — Indian Income Tax Calculator

> ⚠️ **DISCLAIMER**: This tool generates AI-assisted tax calculations and is prone to errors.
> All calculations, values, and details MUST be independently verified by the user before filing.

---

## 1. Project Source Code Structure

```
income-tax-calculator/
├── .env                          # API keys (GEMINI_API_KEY) — NEVER committed
├── .env.example                  # Template for .env file
├── .gitignore                    # Excludes .env, output/, documents/, __pycache__
├── pyproject.toml                # Project metadata, dependencies, entry points
├── README.md                     # Project documentation
├── LICENSE                       # MIT License
│
├── documentation/
│   ├── design/
│   │   └── high_level/           # Architecture & design documents (this folder)
│   └── prompts/                  # Original requirement prompts
│
├── src/
│   └── itr_calculator/
│       ├── __init__.py
│       ├── __main__.py           # Entry point: python -m itr_calculator
│       │
│       ├── cli/                  # CLI Interface Layer
│       │   ├── __init__.py
│       │   ├── app.py            # Click application & command group
│       │   ├── commands/
│       │   │   ├── __init__.py
│       │   │   ├── run.py        # `itrcalc run` — full pipeline execution
│       │   │   ├── validate.py   # `itrcalc validate` — document validation only
│       │   │   ├── init.py       # `itrcalc init` — create input folder structure
│       │   │   ├── search.py     # `itrcalc search` — search ITR regulatory changes
│       │   │   └── status.py     # `itrcalc status` — check pipeline state
│       │   └── display.py        # Rich console formatting utilities
│       │
│       ├── config/               # Configuration & Environment
│       │   ├── __init__.py
│       │   ├── settings.py       # Pydantic settings model (loads .env)
│       │   ├── tax_rates.yaml    # Tax slab rates, special rates by FY
│       │   └── schedules.yaml    # Schedule metadata & field definitions
│       │
│       ├── pipeline/             # Pipeline Engine
│       │   ├── __init__.py
│       │   ├── engine.py         # Pipeline orchestrator
│       │   ├── state.py          # ITRState dataclass (pipeline state)
│       │   └── stages/
│       │       ├── __init__.py
│       │       ├── stage_01_discovery.py
│       │       ├── stage_02_bootstrap.py
│       │       ├── stage_03_extraction.py
│       │       ├── stage_04_reconciliation.py
│       │       ├── stage_05_external.py
│       │       ├── stage_06_computation.py
│       │       ├── stage_07_verification.py
│       │       ├── stage_08_generation.py
│       │       └── stage_09_review.py
│       │
│       ├── models/               # Pydantic Data Models
│       │   ├── __init__.py
│       │   ├── base.py           # Base schedule model, common types
│       │   ├── taxpayer.py       # Taxpayer profile (PAN, name, etc.)
│       │   ├── salary.py         # Schedule Salary model
│       │   ├── house_property.py # Schedule HP model
│       │   ├── capital_gains.py  # Schedule CG model (LTCG, STCG)
│       │   ├── other_sources.py  # Schedule OS model (dividends, interest)
│       │   ├── loss_setoff.py    # Schedule CYLA, BFLA, CFL models
│       │   ├── deductions.py     # Schedule VI-A, 80G, 80GGA models
│       │   ├── amt.py            # Schedule AMT, AMTC models
│       │   ├── special_income.py # Schedule SI model
│       │   ├── exempt_income.py  # Schedule EI model
│       │   ├── foreign.py        # Schedule FA, FSI, TR, Form 67 models
│       │   ├── pass_through.py   # Schedule PTI model
│       │   ├── assets_liab.py    # Schedule AL model
│       │   ├── tds.py            # Schedule TDS model
│       │   ├── total_income.py   # Part B-TI model
│       │   ├── tax_liability.py  # Part B-TTI model
│       │   └── verification.py   # AIS, TIS, 26AS cross-check models
│       │
│       ├── parsers/              # Document Parsers
│       │   ├── __init__.py
│       │   ├── base.py           # Abstract base parser
│       │   ├── pdf_parser.py     # Generic PDF text/table extraction
│       │   ├── excel_parser.py   # Generic Excel sheet parser
│       │   ├── form16_parser.py  # Form 16 specialized parser
│       │   ├── itr_preview_parser.py  # Previous year ITR-2 preview parser
│       │   ├── form_1042s_parser.py   # 1042-S PDF parser
│       │   ├── ais_parser.py     # Annual Information Statement parser
│       │   ├── tis_parser.py     # Taxpayer Information Summary parser
│       │   ├── form26as_parser.py # Form 26AS parser
│       │   ├── salary_slip_parser.py  # Monthly salary slip parser
│       │   ├── bank_cert_parser.py    # Bank interest certificate parser
│       │   ├── bank_stmt_parser.py    # Bank statement parser (SGB interest)
│       │   ├── fidelity_parser.py     # Fidelity account statements & trades
│       │   └── foreign_excel_parser.py # Pre-calculated foreign income Excel
│       │
│       ├── ai/                   # Gemini AI Integration
│       │   ├── __init__.py
│       │   ├── client.py         # Gemini API client with rate limiting
│       │   ├── extractor.py      # Document data extraction via AI
│       │   ├── searcher.py       # Web search for ITR changes via Gemini
│       │   └── prompts/          # AI prompt templates
│       │       ├── extract_form16.txt
│       │       ├── extract_itr_preview.txt
│       │       ├── extract_ais.txt
│       │       ├── extract_1042s.txt
│       │       ├── search_itr_changes.txt
│       │       └── validate_data.txt
│       │
│       ├── schedules/            # Schedule Computation Engine
│       │   ├── __init__.py
│       │   ├── base.py           # Abstract base schedule calculator
│       │   ├── salary.py         # Schedule Salary computation
│       │   ├── house_property.py # Schedule HP computation
│       │   ├── capital_gains.py  # Schedule CG computation (LTCG, STCG, F&O)
│       │   ├── other_sources.py  # Schedule OS computation
│       │   ├── loss_setoff.py    # CYLA, BFLA, CFL computation
│       │   ├── deductions.py     # VI-A, 80G, 80GGA computation
│       │   ├── amt.py            # AMT, AMTC computation
│       │   ├── special_income.py # SI computation
│       │   ├── exempt_income.py  # EI computation
│       │   ├── foreign.py        # FSI, TR, Form 67 computation
│       │   ├── pass_through.py   # PTI computation
│       │   ├── assets_liab.py    # AL computation
│       │   ├── tds.py            # TDS reconciliation
│       │   ├── total_income.py   # Part B-TI computation
│       │   └── tax_liability.py  # Part B-TTI computation
│       │
│       ├── external/             # External Data Fetching
│       │   ├── __init__.py
│       │   ├── ttbr.py           # SBI TTBR rate fetcher
│       │   └── cache.py          # Local file cache for external data
│       │
│       ├── reports/              # Report Generation
│       │   ├── __init__.py
│       │   ├── excel_generator.py    # Excel workbook generation
│       │   ├── markdown_detailed.py  # Detailed markdown report
│       │   ├── markdown_copypaste.py # Copy-paste values markdown
│       │   └── templates/            # Jinja2 templates
│       │       ├── detailed_report.md.j2
│       │       ├── copypaste_report.md.j2
│       │       ├── schedule_section.md.j2
│       │       └── disclaimer.md.j2
│       │
│       └── utils/                # Shared Utilities
│           ├── __init__.py
│           ├── currency.py       # Currency conversion helpers
│           ├── dates.py          # Financial year date utilities
│           ├── masking.py        # PII masking for logs/display
│           ├── decimal_utils.py  # Decimal arithmetic helpers (rounding rules)
│           └── validators.py     # Common validation functions
│
├── tests/                        # Test Suite
│   ├── conftest.py               # Shared fixtures
│   ├── test_parsers/
│   ├── test_schedules/
│   ├── test_pipeline/
│   ├── test_reports/
│   ├── test_external/
│   └── fixtures/                 # Sample documents for testing
│       ├── sample_form16.pdf
│       ├── sample_itr_preview.pdf
│       └── sample_ais.pdf
│
└── scripts/                      # Development scripts
    ├── lint.sh
    └── run_tests.sh
```

---

## 2. User Input Folder Structure

The `itrcalc init` command creates this folder structure for the user to populate:

```
documents/                              # Root input directory (configurable)
├── previous_year/
│   └── itr2_preview_AY2025-26.pdf      # Previous year ITR-2 Preview (REQUIRED)
│
├── salary/
│   ├── form16_FY2025-26.pdf            # Form 16 from employer (REQUIRED)
│   ├── salary_slip_march_2026.pdf      # Last month salary slip
│   └── it_working_sheet_march_2026.pdf # Last month IT working sheet
│
├── foreign_income/
│   ├── account_1/                      # One folder per trading account
│   │   ├── 1042-S.pdf                  # Tax withholding statement
│   │   │
│   │   ├── option_1_self_calc/         # Option 1: Raw documents for self-calculation
│   │   │   ├── statements/             # Monthly account statements (PDF)
│   │   │   │   ├── jan_2025.pdf
│   │   │   │   ├── feb_2025.pdf
│   │   │   │   └── ...
│   │   │   ├── trades/                 # Trade confirmation reports
│   │   │   │   └── trades_2025.pdf
│   │   │   └── transactions/           # Transaction summaries
│   │   │       └── transaction_summary_2025.pdf
│   │   │
│   │   └── option_2_precalculated/     # Option 2: Pre-calculated Excel
│   │       └── foreign_income_calculated.xlsx
│   │
│   └── account_2/                      # Additional accounts follow same structure
│       └── ...
│
├── income_tax_forms/
│   ├── ais_FY2025-26.pdf               # Annual Information Statement (REQUIRED)
│   ├── tis_FY2025-26.pdf               # Taxpayer Information Summary
│   └── form26as_FY2025-26.pdf          # Form 26AS (REQUIRED)
│
├── bank_interest/
│   ├── sbi_savings_interest_cert.pdf   # Bank interest certificates
│   ├── hdfc_fd_interest_cert.pdf       # Fixed deposit certificates
│   └── icici_rd_interest_cert.pdf      # Recurring deposit certificates
│
├── sgb/                                # Sovereign Gold Bonds
│   └── sgb_interest_statement.xlsx     # Bank statement with SGB interest
│
└── donations/                          # For Schedule 80G / 80GGA
    ├── donation_receipt_1.pdf
    └── donation_receipt_2.pdf
```

### Folder Structure Rules

1. **Minimum Required Documents**:
   - `previous_year/itr2_preview_*.pdf` — Bootstrap data
   - `salary/form16_*.pdf` — Salary income source
   - `income_tax_forms/ais_*.pdf` — Cross-verification
   - `income_tax_forms/form26as_*.pdf` — TDS verification

2. **Foreign Income Options**:
   - Each trading account gets its own sub-folder under `foreign_income/`
   - User chooses **Option 1** (self-calculation from raw PDFs) or **Option 2** (pre-calculated Excel)
   - Both options may coexist; Option 2 takes precedence if present

3. **File Naming**:
   - No strict naming convention enforced; documents are classified by folder location and content analysis via Gemini AI
   - Financial year suffixes (e.g., `_FY2025-26`) are recommended but not required

---

## 3. Output Folder Structure

```
output/                                 # Root output directory (configurable)
├── FY2025-26/
│   ├── itr2_calculation_FY2025-26.xlsx           # Excel workbook with all calculations
│   ├── itr2_detailed_report_FY2025-26.md         # Detailed markdown report
│   ├── itr2_copypaste_values_FY2025-26.md        # Copy-paste ready values
│   │
│   ├── form67_FY2025-26.md                       # Form 67 details (if FSI applicable)
│   │
│   ├── audit/                                    # Audit trail
│   │   ├── delta_report.md                       # Changes from previous year
│   │   ├── itr_changes_summary.md                # Regulatory changes summary
│   │   ├── ttbr_rates_used.csv                   # SBI TTBR rates with dates
│   │   ├── ais_verification.md                   # AIS cross-verification report
│   │   ├── 26as_verification.md                  # 26AS cross-verification report
│   │   ├── unresolved_fields.md                  # Fields needing manual review
│   │   └── suggested_sections.md                 # Recommended additional sections
│   │
│   └── logs/
│       ├── pipeline.log                          # Full pipeline execution log
│       └── ai_interactions.log                   # All Gemini AI request/response log
```

### Output File Details

| File | Format | Purpose |
|------|--------|---------|
| `itr2_calculation_*.xlsx` | Excel | Multi-sheet workbook: Summary, per-schedule sheets, tax computation, TDS reconciliation |
| `itr2_detailed_report_*.md` | Markdown | Full narrative with section explanations, calculations shown step-by-step, references to IT Act sections |
| `itr2_copypaste_values_*.md` | Markdown | Flat list of field→value mappings organized by ITR-2 schedule, for direct portal entry |
| `form67_*.md` | Markdown | Form 67 details for foreign tax credit (only if Schedule FSI is applicable) |
| `delta_report.md` | Markdown | What changed from previous year, what was auto-filled vs manually needed |
| `itr_changes_summary.md` | Markdown | Regulatory changes for the current FY discovered via web search |
| `ttbr_rates_used.csv` | CSV | Every TTBR rate used: date, rate, currency pair, transaction reference |
| `unresolved_fields.md` | Markdown | Fields that could not be populated from documents and need user input |

---

## 4. Configuration File Structure

### `.env` File
```env
GEMINI_API_KEY=your-api-key-here
FINANCIAL_YEAR=2025-26
LOG_LEVEL=INFO
OUTPUT_DIR=./output
INPUT_DIR=./documents
```

### `config.yaml` (Optional user config)
```yaml
# Taxpayer information (can also be extracted from Form 16)
taxpayer:
  pan: "XXXXX1234X"      # Will be masked in logs
  name: "Taxpayer Name"
  
# Schedule selection override
# By default, schedules are auto-detected from previous year ITR
# Add schedules not present in previous year here
additional_schedules:
  - SCHEDULE_FA
  - SCHEDULE_AL

# Foreign income configuration
foreign_accounts:
  - name: "Fidelity"
    country: "US"
    calculation_mode: "self"    # "self" or "precalculated"
    currency: "USD"
```

---

*Next: See [04_schedule_engine.md](./04_schedule_engine.md) for schedule computation details.*
