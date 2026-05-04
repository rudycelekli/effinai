---
name: xlsx-processor
description: "Create, edit, and analyze spreadsheets with formulas, formatting, financial models, and data visualization"
version: "1.0.0"
tags: [documents, excel, spreadsheet, data, finance]
createdBy: "built-in"
status: "active"
---

# XLSX Processor

## Activation
When the user asks to create, edit, or analyze .xlsx, .xlsm, .csv, or .tsv files, or build financial models.

## Critical Rules

### Zero Formula Errors
Every Excel model MUST be delivered with ZERO formula errors (#REF!, #DIV/0!, #VALUE!, #N/A, #NAME?).

### Use Formulas, Not Hardcoded Values
ALWAYS use Excel formulas instead of calculating in Python and hardcoding results:

```python
# WRONG
total = df['Sales'].sum()
sheet['B10'] = total  # Hardcodes 5000

# CORRECT
sheet['B10'] = '=SUM(B2:B9)'
```

### Financial Model Color Coding
- **Blue text (0,0,255)**: Hardcoded inputs and scenario values
- **Black text (0,0,0)**: ALL formulas and calculations
- **Green text (0,128,0)**: Links from other worksheets
- **Red text (255,0,0)**: External links to other files
- **Yellow background (255,255,0)**: Key assumptions needing attention

### Number Formatting
- Years: Text strings ("2024" not "2,024")
- Currency: $#,##0 format with units in headers ("Revenue ($mm)")
- Zeros: Format as "-" including percentages
- Percentages: 0.0% (one decimal)
- Multiples: 0.0x for valuation multiples
- Negatives: Parentheses (123) not minus -123

## Workflow

### 1. Data Analysis with pandas

```python
import pandas as pd

df = pd.read_excel('file.xlsx')
all_sheets = pd.read_excel('file.xlsx', sheet_name=None)

df.head()
df.info()
df.describe()

df.to_excel('output.xlsx', index=False)
```

### 2. Creating New Excel Files

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment

wb = Workbook()
sheet = wb.active

sheet['A1'] = 'Revenue'
sheet['A1'].font = Font(bold=True, color='FF0000')
sheet['A1'].fill = PatternFill('solid', start_color='FFFF00')

sheet['B2'] = '=SUM(B5:B16)'
sheet.column_dimensions['A'].width = 20

wb.save('output.xlsx')
```

### 3. Editing Existing Files

```python
from openpyxl import load_workbook

wb = load_workbook('existing.xlsx')
sheet = wb.active

sheet['A1'] = 'New Value'
sheet.insert_rows(2)

new_sheet = wb.create_sheet('NewSheet')
new_sheet['A1'] = 'Data'

wb.save('modified.xlsx')
```

### 4. Formula Recalculation

After creating or modifying formulas, recalculate using LibreOffice:
```bash
python recalc.py output.xlsx 30
```

The script scans ALL cells for errors and returns JSON with details:
```json
{
  "status": "success",
  "total_errors": 0,
  "total_formulas": 42,
  "error_summary": {}
}
```

### 5. Formula Assumptions
- Place ALL assumptions in separate cells
- Use cell references: `=B5*(1+$B$6)` not `=B5*1.05`
- Document hardcoded sources: "Source: Company 10-K, FY2024, Page 45"

## Verification Checklist
- [ ] Test 2-3 sample references before building full model
- [ ] Confirm column mapping (column 64 = BL, not BK)
- [ ] Row offset: DataFrame row 5 = Excel row 6
- [ ] Handle NaN with `pd.notna()`
- [ ] Check division by zero in formula denominators
- [ ] Verify cross-sheet references use correct format (Sheet1!A1)

## Library Selection
- **pandas**: Data analysis, bulk operations, simple export
- **openpyxl**: Complex formatting, formulas, Excel-specific features
