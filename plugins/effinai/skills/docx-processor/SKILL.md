---
name: docx-processor
description: "Create, edit, and analyze Word documents with tracked changes, comments, formatting, and redlining support"
version: "1.0.0"
tags: [documents, docx, word, office, editing]
createdBy: "built-in"
status: "active"
---

# DOCX Processor

## Activation
When the user asks to create, edit, analyze, or review .docx files. Also triggers for tracked changes, redlining, or document comparison tasks.

## Workflow

### 1. Determine Operation Type

**Reading/Analyzing Content:**
- Convert to markdown for text extraction: `pandoc --track-changes=all file.docx -o output.md`
- For comments, formatting, embedded media: unpack the ZIP archive and read raw XML

**Creating New Documents:**
- Use docx-js (JavaScript/TypeScript) with Document, Paragraph, TextRun components
- Export via Packer.toBuffer()

**Editing Existing Documents:**
- Unpack: extract the .docx ZIP to access `word/document.xml`, `word/comments.xml`, `word/media/`
- Apply edits to XML using Python with defusedxml for secure parsing
- Repack to .docx

### 2. Text Extraction

```bash
# Convert with tracked changes preserved
pandoc --track-changes=all document.docx -o output.md

# Options: --track-changes=accept/reject/all
```

### 3. Key XML Structure
- `word/document.xml` — Main document body
- `word/comments.xml` — Comments referenced in document.xml
- `word/media/` — Embedded images and media
- Tracked changes: `<w:ins>` (insertions), `<w:del>` (deletions)

### 4. Redlining Workflow (Document Review)

For professional document review with tracked changes:

1. **Get markdown representation**: Convert to markdown preserving tracked changes
2. **Identify and group changes**: Organize into logical batches of 3-10 changes
3. **Map text to XML**: Grep `word/document.xml` to verify how text splits across `<w:r>` elements
4. **Implement changes in batches**: Use precise edits — only mark text that actually changes

**Minimal, Precise Edits Principle:**
```python
# BAD - Replaces entire sentence
'<w:del><w:delText>The term is 30 days.</w:delText></w:del><w:ins><w:t>The term is 60 days.</w:t></w:ins>'

# GOOD - Only marks what changed
'<w:r><w:t>The term is </w:t></w:r><w:del><w:delText>30</w:delText></w:del><w:ins><w:t>60</w:t></w:ins><w:r><w:t> days.</w:t></w:r>'
```

5. **Pack the document**: Repack XML back into .docx
6. **Verify**: Convert final document back to markdown and validate all changes applied

### 5. Document to Image Conversion

```bash
# Step 1: DOCX to PDF
soffice --headless --convert-to pdf document.docx

# Step 2: PDF pages to JPEG
pdftoppm -jpeg -r 150 document.pdf page
# Creates page-1.jpg, page-2.jpg, etc.
```

### 6. New Document Creation (docx-js)

```javascript
const { Document, Paragraph, TextRun, Packer } = require('docx');

const doc = new Document({
  sections: [{
    properties: {},
    children: [
      new Paragraph({
        children: [
          new TextRun({ text: "Title", bold: true, size: 28 }),
        ],
      }),
      new Paragraph({
        children: [
          new TextRun("Body text content here."),
        ],
      }),
    ],
  }],
});

const buffer = await Packer.toBuffer(doc);
fs.writeFileSync("output.docx", buffer);
```

## Dependencies
- pandoc (text extraction/conversion)
- docx npm package (creating new documents)
- LibreOffice (PDF conversion)
- poppler-utils (pdftoppm for image conversion)
- defusedxml (secure XML parsing)

## Output Format
Deliver the processed document as a .docx file. Provide a summary of changes made and any issues encountered.
