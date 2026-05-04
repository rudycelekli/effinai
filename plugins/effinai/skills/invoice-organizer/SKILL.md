---
name: invoice-organizer
description: "Automatically organize invoices and receipts for tax preparation — extract info, rename, sort, and generate CSV summaries"
version: "1.0.0"
tags: [productivity, finance, invoices, tax, bookkeeping]
createdBy: "built-in"
status: "active"
---

# Invoice Organizer

## Activation
When the user asks to organize invoices, receipts, or financial documents for taxes, bookkeeping, or expense tracking.

## Workflow

### 1. Scan the Folder

```bash
find . -type f \( -name "*.pdf" -o -name "*.jpg" -o -name "*.png" \) -print
```

Report: total files, file types, date range, current organization state.

### 2. Extract Information from Each File

**From PDF invoices** — look for patterns:
- "Invoice Date:", "Date:", "Issued:"
- "Invoice #:", "Invoice Number:"
- Company name (usually at top)
- "Amount Due:", "Total:", "Amount:"
- "Description:", "Service:", "Product:"

**From image receipts:**
- Read visible text, identify vendor name (top)
- Find date and total amount

**Fallback for unclear files:**
- Use filename clues
- Check file modification date
- Flag for manual review

### 3. Determine Organization Strategy

Ask user preference if not specified:
1. **By Vendor** (Adobe/, Amazon/, etc.)
2. **By Category** (Software/, Office Supplies/, Travel/)
3. **By Date** (2024/Q1/, 2024/Q2/)
4. **By Tax Category** (Deductible/, Personal/)
5. **Custom** structure

Default: Year/Category/Vendor

### 4. Create Standardized Filename

Pattern: `YYYY-MM-DD Vendor - Invoice - Description.ext`

Examples:
- `2024-03-15 Adobe - Invoice - Creative Cloud.pdf`
- `2024-01-10 Amazon - Receipt - Office Supplies.pdf`
- `2023-12-01 Stripe - Invoice - Monthly Payment Processing.pdf`

### 5. Execute Organization

Show the plan first, then after approval:
```bash
mkdir -p "Invoices/2024/Software/Adobe"
cp "original.pdf" "Invoices/2024/Software/Adobe/2024-03-15 Adobe - Invoice - Creative Cloud.pdf"
```

### 6. Generate CSV Summary

```csv
Date,Vendor,Invoice Number,Description,Amount,Category,File Path
2024-03-15,Adobe,INV-12345,Creative Cloud,52.99,Software,Invoices/2024/Software/Adobe/...
2024-03-10,Amazon,123-4567890,Office Supplies,127.45,Office,Invoices/2024/Office/Amazon/...
```

Useful for: accounting software import, sharing with accountants, expense tracking, tax preparation.

### 7. Completion Summary

Report:
- Files processed, date range, total amounts
- Unique vendors count
- New folder structure with file counts
- Files needing manual review
- Next steps: review CSV, check flagged files, import to accounting software

## Special Cases

**Missing Information:** Flag for manual review, use file modification date as fallback, create "Needs-Review/" folder.

**Duplicate Invoices:** Compare file hashes, keep highest quality version, note duplicates in summary.

**Multi-Page Invoices:** Merge PDFs if needed, use consistent naming for parts.

## Organization Patterns

```
# Tax-Friendly
Invoices/2024/Software/
Invoices/2024/Hardware/
Invoices/2024/Travel/

# Accountant-Ready
Invoices/Deductible/Software/
Invoices/Partially-Deductible/Meals-Travel/
Invoices/Personal/
```
