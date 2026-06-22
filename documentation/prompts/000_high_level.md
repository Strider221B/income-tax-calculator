# Requirements

---
## Overview
I want to create a command line utility which will perform and generate a detailed income tax calculation report for Indian Income Tax.
The tool will target ITR-2 Form - new tax regime and will cover the following aspects.
- SCHEDULE SALARY - DETAILS OF INCOME FROM SALARY
  - Salary split up "Salary as per section 17(1)", "Value of perquisites as per section 17(2)", "SECTION 10(13A)
HOUSE RENT ALLOWANCE(HRA)"
- SCHEDULE HP - DETAILS OF INCOME FROM HOUSE PROPERTY (PLEASE REFER INSTRUCTIONS)
  - Property details, Section 24(b) - Interest on borrowed capital
- SCHEDULE CG CAPITAL GAINS
- SCHEDULE OS INCOME FROM OTHER SOURCES
  - Dividends, Gross
  - Interest, Gross
  - Information about accrual/receipt of income from Other Sources - quarter wise split for dividends
- SCHEDULE CYLA DETAILS OF INCOME AFTER SET OFF OF CURRENT YEAR LOSSES
- SCHEDULE BFLA DETAILS OF INCOME AFTER SET OFF OF BROUGHT FORWARD LOSSES OF EARLIER YEARS
- SCHEDULE CFL DETAILS OF LOSSES TO BE CARRIED FORWARD TO FUTURE YEARS
- SCHEDULE VI-A DEDUCTIONS UNDER CHAPTER VI-A
- SCHEDULE 80G DETAILS OF DONATION ENTITLED FOR DEDUCTION UNDER SECTION 80G
- SCHEDULE 80GGA DETAILS OF DONATIONS FOR SCIENTIFIC RESEARCH OR RURAL DEVELOPMENT
- SCHEDULE AMT - COMPUTATION OF ALTERNATE MINIMUM TAX PAYABLE UNDER SECTION 115JD
- SCHEDULE AMTC - COMPUTATION OF TAX CREDIT UNDER SECTION 115JC
- SCHEDULE SI - INCOME CHARGEABLE TO TAX AT SPECIAL RATES (PLEASE SEE INSTRUCTIONS NO. 9 FOR RATE OF TAX)
- SCHEDULE EI - DETAILS OF EXEMPT INCOME (INCOME NOT TO BE INCLUDED IN TOTAL INCOME OR NOT CHARGEABLE TO TAX)
- SCHEDULE PTI - PASS THROUGH INCOME DETAILS FROM BUSINESS TRUST OR INVESTMENT FUND AS PER SECTION 115U, 115UA,
115UB
- SCHEDULE FSI - DETAILS OF INCOME FROM OUTSIDE INDIA AND TAX RELIEF (AVAILABLE IN CASE OF RESIDENT)
- SCHEDULE TR - SUMMARY OF TAX RELIEF CLAIMED FOR TAXES PAID OUTSIDE INDIA (AVAILABLE ONLY IN CASE OF RESIDENT)
- SCHEDULE FA - DETAILS OF FOREIGN ASSETS AND INCOME FROM ANY SOURCE OUTSIDE INDIA
- SCHEDULE AL ASSETS AND LIABILITIES AT THE END OF THE YEAR (OTHER THAN THOSE INCLUDED IN PART A- BS) (APPLICABLE IN A
CASE WHERE TOTAL INCOME EXCEEDS 1 CRORE)
- PART B – TI COMPUTATION OF TOTAL INCOME
- PARTB-TTI - COMPUTATION OF TAX LIABILITY ON TOTAL INCOME
- SCHEDULE TDS1 - 20B DETAILS OF TAX DEDUCTED AT SOURCE FROM SALARY [AS PER FORM 16 ISSUED BY EMPLOYER(S)]
Also, fill out the details for Form 67 if Schedule FSI is required.
- Details of income from a country or specified territory outside India and Foreign Tax Credit claimed
---

---
## Documents Required
The report will ask for the necessary documents in a specific folder struture so that it can be parsed and generate the output.
The documents will be required at a minimum:
- Previous Year ITR 2 Preview document. Pre-populate all fields from there. Then for all non zero entries scan for documents from current year and update the values. Generate a final report of what values could not be  fetched from current documents and had to be used from previous year ITR 2 Preview document.
- List down all te schedules that had non zero values and then ask of the user wants to add any other schedule.
- Perform a google search for changes in ITR for this financial report and generate a summary for the user. A change could be like
tax rate has changed for LTCG or requirements for schedule AL or schedule FA has changed etc. Or it could be like a new scedule has been introduced and what qualifies for that.
- Mention the documents referring the Previous Year ITR 2 Preview document. Also based on the documents provided, suggest if some new 
section needs to be filled in order to remain complaint.
- Some of the documents being
- Provide a list of section
- Salary
  - Form 16 (Verify it's for the correct Financial Year)
  - Last month salary slip - March
  - Last month IT working sheet - March
- Foreign asset
  - Ask if new trading account was created this year in addition to the previous year records.
  - For each trading account
  - 1042-S.pdf
  - Two distinct set of documents for LTCG, STCG, F&O, Intraday, Dividend, Interest, Schedule FA, Schedule FSI, Form 67, Schedule TR 
    - Option 1: Self calculation - gather all info required. Input document type PDF - Fidelity 
      - Query for the appropriate "SBI Telegraphic Transfer Buying Rate (TTBR)" for converting USD to INR using API/web scraping.
        Include the rate and date used per conversion in the report.
      - Monthwise account statements
      - Trades
      - Transaction statements and summary.
    - Option 2: Pre calculated excel sheet having all details.
- Current income tax forms
  - Annual Information Statement (AIS) - verify we are covering all the details mentioned in AIS
  - Taxpayer Information Summary (TIS)
  - Form 26AS - verify we are covering all the details mentioned in AIS
- Bank Interest certificates
  - For each saving, fixed deposit and recurring deposit account.
- Bank Statement for Calculating interest received from Sovereign Gold Bonds - input format excel
      
--- 

Can you come up with the high level architecture diagrams for this application. Backend techstack python.
Ask for Gemini API key to be set in  .env file so that searches, text extractions can be performed.

Report output format: 
- Excel for all calculation. 
- A detailed markdown file with all the calculation and section details
- A mardown file with values per section which can be used to just copy paste values to income tax website.

Add a warning saying that this is AI generated and is prone to errors. Please verify all calculations / details.

Generate all the design documents at this location:
documentation/design/high_level
