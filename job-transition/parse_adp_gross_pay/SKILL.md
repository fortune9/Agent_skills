---
name: parse_adp_gross_pay
description: >-
  Extract pay dates, gross pay amounts, and source line numbers from ADP paystub PDFs into a structured CSV/table using pdftotext.
  Use this skill whenever the user wants to process ADP paystub PDFs or extract pay dates, gross pay, or period ranges from ADP paystubs.
---

# Parse ADP Gross Pay Skill

This skill provides a self-contained procedure and script asset for extracting pay dates (`Period Beginning`, `Period Ending`, `Pay Date`), `Gross Pay` amounts, and corresponding source line numbers from ADP paystub PDF files using Python and `pdftotext`.

## Skill Assets

- **Script Asset**: [extract_paystubs.py](./scripts/extract_paystubs.py)

---

## Workflow Steps

### Step 1: Environment & Prerequisites Check (Mandatory Before Writing/Running Scripts)

Before writing any script or executing extraction, check that both **Python** and **pdftotext** are installed.

1. **Check Python**:
   - Verify installation:
     ```bash
     python3 --version || python --version
     ```
   - **If not installed**, attempt to install it:
     - Ubuntu/Debian: `sudo apt update && sudo apt install -y python3 python3-pip`
     - macOS: `brew install python`
   - **If automated installation fails**, direct the user to download and install Python manually from:
     https://www.python.org/downloads/

2. **Check `pdftotext`**:
   - Verify installation:
     ```bash
     pdftotext -v
     ```
   - **If not installed**, install according to the host operating system:
     - **Ubuntu / Debian**:
       ```bash
       sudo apt install poppler-utils
       ```
     - **macOS**:
       ```bash
       brew install poppler
       ```
     - **Windows**:
       Direct the user to download and install from:
       https://www.xpdfreader.com/download.html

---

### Step 2: Confirm Target Fields with User

1. **Ask User Confirmation**:
   - Prompt the user to confirm the target fields to extract.
   - **Default fields**:
     - `Period Beginning`
     - `Period Ending`
     - `Pay Date`
     - `Gross Pay` (this period amount)
   - **User overrides**: If the user provides or requests other fields (e.g., `Net Pay`, `Federal Income Tax`, `Hours Worked`), configure the extraction to use those requested fields.
2. **Missing Fields Validation**:
   - The extraction script **must throw an error** (e.g. `ValueError` halting execution with an informative message) if any of the confirmed required fields cannot be found in a processed PDF.

---

### Step 3: Check for Existing Extraction Script Asset

1. Check if the script asset exists at `scripts/extract_paystubs.py` within this skill folder.
2. **If `scripts/extract_paystubs.py` already exists** and matches the confirmed target fields:
   - Skip script creation and proceed directly to **Step 4**.
3. **If writing or updating the script for the first time**:
   - **Ask User for Layout Examples**: Ask the user to provide example PDF files representing all possible presentation layouts (e.g., regular salary, bonus-only stubs, stock/RSU vesting, expense adjustments).
   - **Write the Script into `scripts/extract_paystubs.py`**:
     - Include `#!/usr/bin/env python3` shebang so it can be executed directly.
     - Convert PDFs using `pdftotext -layout <pdf> -` to preserve visual column alignments.
     - Save and retain the converted `.txt` files alongside the PDFs for manual verification.
     - Extract the confirmed fields and record their 1-indexed line numbers (`Beg Line`, `End Line`, `Pay Date Line`, `Gross Line`, etc.) in the converted `.txt` file.
     - **Throw an explicit error** if any required field is missing from a PDF.
     - Provide CLI argument parsing (`-o`/`--output` with default `extracted_paysub_gross_pay.csv`, `--no-txt`, and usage display on `-h`/`--help` or when executed without arguments).
     - Make the script executable (`chmod +x scripts/extract_paystubs.py`).

---

### Step 4: Execute the Extraction Script

Run the bundled script asset against the user's PDF file(s) or directory:

```bash
.agents/skills/parse_adp_gross_pay/scripts/extract_paystubs.py <pdf_files_or_directory>
```

Or with a custom output file:
```bash
.agents/skills/parse_adp_gross_pay/scripts/extract_paystubs.py -o <output_path.csv> <pdf_files_or_directory>
```

---

### Step 5: Ask User to Confirm Extracted Numbers

1. Render the extracted data table in markdown format showing:
   - PDF file name
   - Converted `.txt` file name
   - Value and line number for each extracted field (e.g., `Period Beginning` & `Beg Line`, `Gross Pay` & `Gross Line`)
2. Ask the user to verify and confirm the extracted values directly against the specified line numbers in the retained `.txt` files.

---

### Step 6: Strict Guidelines (No Assumptions)

- **Always Ask Questions**: If there is any doubt, unexpected formatting, unmatched pattern, or ambiguity in the paystubs, always stop and ask the user for clarification.
- **Never Assume**: Never assume field locations, column offsets, or calculation logic without confirming against actual converted `.txt` layouts and user requirements.
