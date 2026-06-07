---
name: xlsx
description: "Use this skill any time a spreadsheet file is the primary input or output. This means any task where the user wants to: open, read, edit, or fix an existing .xlsx, .xlsm, .csv, or .tsv file (e.g., adding columns, computing formulas, formatting, charting, cleaning messy data); create a new spreadsheet from scratch or from other data sources; or convert between tabular file formats. Trigger especially when the user references a spreadsheet file by name or path — even casually (like 'the xlsx in my downloads') — and wants something done to it or produced from it. Also trigger for cleaning or restructuring messy tabular data files (malformed rows, misplaced headers, junk data) into proper spreadsheets. The deliverable must be a spreadsheet file. Do NOT trigger when the primary deliverable is a Word document, HTML report, standalone Python script, database pipeline, or Google Sheets API integration, even if tabular data is involved."
license: Proprietary. LICENSE.txt has complete terms
---

# XLSX creation, editing, and analysis

## Requirements for Outputs

- **Professional font**: Arial or Times New Roman unless user specifies otherwise
- **Zero formula errors**: Deliver with ZERO #REF!, #DIV/0!, #VALUE!, #N/A, #NAME?
- **Preserve existing templates**: EXACTLY match format/style/conventions when modifying files

## Financial Models — Color Coding

- **Blue text (0,0,255)**: Hardcoded inputs
- **Black text (0,0,0)**: ALL formulas and calculations
- **Green text (0,128,0)**: Links from other worksheets
- **Red text (255,0,0)**: External links to other files
- **Yellow background (255,255,0)**: Key assumptions needing attention

## Number Formatting

- **Years**: Text strings ("2024" not "2,024")
- **Currency**: $#,##0 with units in headers ("Revenue ($mm)")
- **Zeros**: Format as "-" including percentages
- **Percentages**: 0.0% default
- **Negatives**: Parentheses (123) not minus -123

## CRITICAL: Use Formulas, Not Hardcoded Values

```python
# ❌ WRONG
sheet['B10'] = df['Sales'].sum()  # Hardcodes 5000

# ✅ CORRECT
sheet['B10'] = '=SUM(B2:B9)'
sheet['C5'] = '=(C4-C2)/C2'
sheet['D20'] = '=AVERAGE(D2:D19)'
```

## Common Workflow

1. **Choose tool**: pandas for data analysis, openpyxl for formulas/formatting
2. **Create/Load**: New workbook or load existing
3. **Modify**: Add/edit data, formulas, formatting
4. **Save**: Write to file
5. **Recalculate (MANDATORY if using formulas)**:
   ```bash
   python scripts/recalc.py output.xlsx
   ```
6. **Verify and fix errors**: Script returns JSON with error locations

## Creating New Files

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment

wb = Workbook()
sheet = wb.active
sheet['A1'] = 'Hello'
sheet['B2'] = '=SUM(A1:A10)'
sheet['A1'].font = Font(bold=True)
wb.save('output.xlsx')
```

## Reading Files

```python
import pandas as pd
df = pd.read_excel('file.xlsx')
all_sheets = pd.read_excel('file.xlsx', sheet_name=None)
```

## Formula Error Prevention Checklist

- [ ] Test 2-3 sample references before building full model
- [ ] Column mapping correct (Excel col 64 = BL, not BK)
- [ ] Row offset: Excel rows are 1-indexed (DataFrame row 5 = Excel row 6)
- [ ] NaN handling with `pd.notna()`
- [ ] Division by zero checks
- [ ] Cross-sheet references use format `Sheet1!A1`

## Library Selection

- **pandas**: Data analysis, bulk operations, simple export
- **openpyxl**: Complex formatting, formulas, Excel-specific features

**Warning**: `data_only=True` + save replaces formulas with values permanently.
