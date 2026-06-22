# Data Flow Architecture — Indian Income Tax Calculator

> ⚠️ **DISCLAIMER**: This tool generates AI-assisted tax calculations and is prone to errors.
> All calculations, values, and details MUST be independently verified by the user before filing.

---

## 1. End-to-End Pipeline Flow

```mermaid
flowchart TB
    subgraph Stage1["Stage 1: Document Discovery"]
        A1[Scan Input Directory] --> A2[Validate Folder Structure]
        A2 --> A3[Classify Documents by Type]
        A3 --> A4[Generate Missing Document Report]
    end

    subgraph Stage2["Stage 2: Previous Year Bootstrap"]
        B1[Parse Previous Year ITR-2 Preview PDF] --> B2[Extract All Schedule Values]
        B2 --> B3[Build Baseline Data Model]
        B3 --> B4[Identify Non-Zero Schedules]
    end

    subgraph Stage3["Stage 3: Current Year Extraction"]
        C1[Parse Form 16] --> C5[Merge Into Current Year Model]
        C2[Parse Bank Certificates] --> C5
        C3[Parse Foreign Income Docs] --> C5
        C4[Parse AIS / TIS / 26AS] --> C5
        C6[Parse Salary Slip & IT Sheet] --> C5
        C7[Parse SGB Bank Statements] --> C5
    end

    subgraph Stage4["Stage 4: Data Reconciliation"]
        D1[Compare Previous Year vs Current Year] --> D2[Flag Unresolved Fields]
        D2 --> D3[Generate Delta Report]
        D3 --> D4[Prompt User for Missing Data]
    end

    subgraph Stage5["Stage 5: External Data Fetch"]
        E1[Fetch SBI TTBR Rates] --> E3[Cache Results Locally]
        E2[Search ITR Regulatory Changes via Gemini] --> E3
        E3 --> E4[Generate Change Summary]
    end

    subgraph Stage6["Stage 6: Schedule Computation"]
        F1[Schedule Salary] --> F14[Part B-TI: Total Income]
        F2[Schedule HP] --> F14
        F3[Schedule CG] --> F14
        F4[Schedule OS] --> F14
        F5[Schedule CYLA] --> F14
        F6[Schedule BFLA] --> F14
        F7[Schedule CFL] --> F14
        F8[Schedule VI-A] --> F14
        F9[Schedule SI] --> F14
        F10[Schedule EI] --> F14
        F11[Schedule FSI + Form 67] --> F14
        F12[Schedule TR] --> F14
        F13[Schedule FA + AL] --> F14
        F14 --> F15[Part B-TTI: Tax Liability]
        F15 --> F16[Schedule AMT / AMTC]
        F16 --> F17[Schedule TDS Reconciliation]
    end

    subgraph Stage7["Stage 7: Cross-Verification"]
        G1[Compare Computed vs AIS] --> G3[Generate Discrepancy Report]
        G2[Compare Computed vs 26AS] --> G3
        G3 --> G4[Suggest Additional Sections]
    end

    subgraph Stage8["Stage 8: Report Generation"]
        H1[Excel Workbook Generator] --> H4[Output Directory]
        H2[Detailed Markdown Report] --> H4
        H3[Copy-Paste Values Markdown] --> H4
    end

    subgraph Stage9["Stage 9: User Review"]
        I1[Display Summary in Terminal] --> I2[Interactive Q&A]
        I2 --> I3[Accept / Amend Values]
        I3 --> I4{Changes Made?}
        I4 -->|Yes| F1
        I4 -->|No| H4
    end

    Stage1 --> Stage2
    Stage2 --> Stage3
    Stage3 --> Stage4
    Stage4 --> Stage5
    Stage5 --> Stage6
    Stage6 --> Stage7
    Stage7 --> Stage8
    Stage8 --> Stage9
```

---

## 2. Data Model Flow

```mermaid
flowchart LR
    subgraph RawDocs["Raw Documents"]
        PDF[PDF Files]
        XLS[Excel Files]
    end

    subgraph Extraction["Extraction Layer"]
        PE[PDF Extractor]
        XE[Excel Extractor]
        GE[Gemini AI Extractor]
    end

    subgraph DataModel["Pydantic Data Models"]
        BM[Baseline Model\nPrevious Year ITR-2]
        CM[Current Year Model\nMerged Data]
        SM[Schedule Models\nPer-Schedule Typed Data]
    end

    subgraph Computation["Computation"]
        SE[Schedule Engine]
        TE[Tax Engine]
    end

    subgraph Output["Output Layer"]
        EX[Excel Workbook]
        MD1[Detailed Report .md]
        MD2[Copy-Paste Values .md]
    end

    PDF --> PE --> BM
    PDF --> GE --> CM
    XLS --> XE --> CM
    BM --> CM
    CM --> SM
    SM --> SE --> TE
    TE --> EX
    TE --> MD1
    TE --> MD2
```

---

## 3. Foreign Income Processing Flow (Option 1: Self-Calculation)

This is one of the most complex flows as it involves currency conversion, multiple income types, and multiple schedules.

```mermaid
flowchart TB
    subgraph ForeignDocs["Input Documents per Account"]
        F1[1042-S PDF]
        F2[Monthly Account Statements]
        F3[Trade Reports]
        F4[Transaction Statements & Summary]
    end

    subgraph Extraction["Data Extraction"]
        E1[Extract Dividends & Dates]
        E2[Extract Interest Income]
        E3[Extract LTCG / STCG Transactions]
        E4[Extract F&O / Intraday Trades]
    end

    subgraph TTBR["Currency Conversion"]
        T1[Fetch SBI TTBR Rates]
        T2[Match Rate to Transaction Date]
        T3[Convert USD → INR per Transaction]
        T4[Log Rate + Date per Conversion]
    end

    subgraph Schedules["Populate Schedules"]
        S1[Schedule CG\nLTCG / STCG in INR]
        S2[Schedule OS\nDividends & Interest in INR\nQuarter-wise Split]
        S3[Schedule FA\nForeign Asset Details]
        S4[Schedule FSI\nForeign Source Income Details]
        S5[Form 67\nForeign Tax Credit Claim]
        S6[Schedule TR\nTax Relief Summary]
    end

    ForeignDocs --> Extraction
    Extraction --> TTBR
    TTBR --> Schedules
    E1 --> S2
    E2 --> S2
    E3 --> S1
    E4 --> S1
    F1 --> S5
    F1 --> S6
    F1 --> S3
```

---

## 4. Foreign Income Processing Flow (Option 2: Pre-Calculated Excel)

```mermaid
flowchart TB
    subgraph Input["Input"]
        X1[Pre-calculated Excel Sheet]
    end

    subgraph Parse["Parse Sheets"]
        P1[Read LTCG/STCG Sheet]
        P2[Read Dividend Sheet]
        P3[Read Interest Sheet]
        P4[Read F&O Sheet]
        P5[Read Schedule FA Sheet]
        P6[Read Form 67 Sheet]
    end

    subgraph Map["Map to Schedules"]
        M1[Schedule CG]
        M2[Schedule OS]
        M3[Schedule FA]
        M4[Schedule FSI]
        M5[Form 67]
        M6[Schedule TR]
    end

    X1 --> Parse
    P1 --> M1
    P2 --> M2
    P3 --> M2
    P4 --> M1
    P5 --> M3
    P6 --> M4
    P6 --> M5
    P6 --> M6
```

---

## 5. Schedule Dependency Graph

Schedules must be computed in a specific order due to interdependencies:

```mermaid
graph TB
    SAL[Schedule Salary] --> TI[Part B-TI\nTotal Income]
    HP[Schedule HP] --> TI
    CG[Schedule CG] --> CYLA
    OS[Schedule OS] --> CYLA
    
    CYLA[Schedule CYLA\nCurrent Year Loss Set-Off] --> BFLA
    BFLA[Schedule BFLA\nBrought Forward Loss Set-Off] --> CFL
    CFL[Schedule CFL\nCarry Forward Losses] --> TI
    
    VIA[Schedule VI-A\nDeductions] --> TI
    G80[Schedule 80G\nDonations] --> VIA
    GGA[Schedule 80GGA\nScientific Research] --> VIA
    
    FSI[Schedule FSI] --> TR[Schedule TR]
    FORM67[Form 67] --> FSI
    FA[Schedule FA] --> FSI
    
    TI --> TTI[Part B-TTI\nTax Liability]
    SI[Schedule SI\nSpecial Rate Income] --> TTI
    AMT[Schedule AMT] --> AMTC[Schedule AMTC]
    AMTC --> TTI
    TR --> TTI
    TDS[Schedule TDS] --> TTI
    
    PTI[Schedule PTI] --> TI
    EI[Schedule EI] -.-> TI
    AL[Schedule AL] -.-> TI

    style TI fill:#1a73e8,color:#fff
    style TTI fill:#e8710a,color:#fff
    style CYLA fill:#34a853,color:#fff
    style BFLA fill:#34a853,color:#fff
```

**Legend**:
- **Solid arrows** (`→`): Direct computational dependency
- **Dashed arrows** (`⇢`): Informational dependency (referenced but doesn't feed into computation)

---

## 6. Cross-Verification Data Flow

```mermaid
flowchart LR
    subgraph Computed["Computed Values"]
        CV1[Salary Income]
        CV2[Interest Income]
        CV3[Dividend Income]
        CV4[Capital Gains]
        CV5[TDS Deducted]
    end

    subgraph AISData["AIS / TIS / 26AS"]
        AD1[Reported Salary]
        AD2[Reported Interest]
        AD3[Reported Dividends]
        AD4[Reported CG]
        AD5[Reported TDS]
    end

    subgraph Verification["Cross-Verification"]
        V1{Match?}
        V2[Discrepancy Report]
        V3[Suggested Corrections]
    end

    CV1 & AD1 --> V1
    CV2 & AD2 --> V1
    CV3 & AD3 --> V1
    CV4 & AD4 --> V1
    CV5 & AD5 --> V1
    V1 -->|Mismatch| V2 --> V3
    V1 -->|Match| OK[✓ Verified]
```

---

## 7. State Management

The pipeline maintains a central `ITRState` object that flows through all stages:

```python
@dataclass
class ITRState:
    # Configuration
    financial_year: str
    assessment_year: str
    
    # Stage 1 outputs
    discovered_documents: dict[DocumentType, list[Path]]
    missing_documents: list[str]
    
    # Stage 2 outputs
    previous_year_data: dict[ScheduleType, ScheduleData]
    non_zero_schedules: list[ScheduleType]
    
    # Stage 3 outputs
    current_year_data: dict[ScheduleType, ScheduleData]
    
    # Stage 4 outputs
    delta_report: DeltaReport
    unresolved_fields: list[UnresolvedField]
    
    # Stage 5 outputs
    ttbr_rates: dict[date, Decimal]
    itr_changes_summary: str
    
    # Stage 6 outputs
    computed_schedules: dict[ScheduleType, ComputedSchedule]
    total_income: Decimal
    tax_liability: TaxLiability
    
    # Stage 7 outputs
    verification_results: VerificationReport
    
    # Stage 8 outputs
    output_paths: OutputPaths
```

---

*Next: See [03_folder_structure.md](./03_folder_structure.md) for input/output folder conventions.*
