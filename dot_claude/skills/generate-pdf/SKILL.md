---
name: pdf
description: Generate professional PDF documents with fillable form fields using Python and reportlab. Use when asked to create PDFs, forms, letters, or documents with interactive fields.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

This skill generates PDF documents with fillable form fields using Python's `reportlab` library.

## Workflow

1. Gather requirements from the user: document text, layout, and which fields should be fillable
2. Generate a Python script that uses `reportlab` to produce the PDF
3. Run the script to create the PDF

## Output Location

- Save the generated Python script to `/tmp/generate_pdf.py`
- Save the PDF output to `/tmp/output.pdf` (or a descriptive name based on the document type, e.g. `/tmp/Letter_of_Authorization.pdf`)

## Python Script Structure

Every generated script MUST follow this pattern:

```python
from reportlab.lib.pagesizes import A4
from reportlab.pdfgen import canvas
from reportlab.lib.units import cm, mm
from reportlab.lib.colors import HexColor

OUTPUT = "/tmp/<document_name>.pdf"

width, height = A4
c = canvas.Canvas(OUTPUT, pagesize=A4)

# ... layout and content ...

c.save()
print(f"PDF created: {OUTPUT}")
```

## Available Field Types

Use `c.acroForm` to create fillable fields. Common types:

### Text Field
```python
c.acroForm.textfield(
    name="field_name",
    tooltip="Hint text",
    x=x_pos, y=y_pos - 3,
    width=5 * cm, height=0.55 * cm,
    borderWidth=0,
    fillColor=HexColor("#f5f5f5"),
    textColor=HexColor("#1a1a1a"),
    fontSize=11,
    fontName="Helvetica",
)
```

### Checkbox
```python
c.acroForm.checkbox(
    name="checkbox_name",
    tooltip="Check this box",
    x=x_pos, y=y_pos - 3,
    size=14,
    borderWidth=1,
    fillColor=HexColor("#f5f5f5"),
    borderColor=HexColor("#cccccc"),
    textColor=HexColor("#1a1a1a"),
)
```

### Radio Button
```python
c.acroForm.radio(
    name="radio_group",
    tooltip="Select one",
    value="option1",
    x=x_pos, y=y_pos - 3,
    size=14,
    fillColor=HexColor("#f5f5f5"),
    borderColor=HexColor("#cccccc"),
    textColor=HexColor("#1a1a1a"),
)
```

### Choice (Dropdown)
```python
c.acroForm.choice(
    name="choice_name",
    tooltip="Select an option",
    options=[("val1", "Label 1"), ("val2", "Label 2")],
    x=x_pos, y=y_pos - 3,
    width=5 * cm, height=0.55 * cm,
    borderWidth=1,
    fillColor=HexColor("#f5f5f5"),
    textColor=HexColor("#1a1a1a"),
    fontSize=11,
    fontName="Helvetica",
)
```

## Design Guidelines

- Use A4 page size
- Margins: 3 cm left/right
- Use a clean color palette:
  - Dark text: `#1a1a1a`
  - Accent/headers: `#2c3e50`
  - Light grey lines: `#cccccc`
  - Field background: `#f5f5f5`
- Line height for body text: `0.55 * cm`
- Font sizes: title 18-22pt bold, body 11pt, hints 8pt italic
- Available fonts: `Helvetica`, `Helvetica-Bold`, `Helvetica-Oblique`, `Courier`, `Times-Roman`
- Separate sections with subtle lines using `c.line()` or `c.setStrokeColor()`
- For signature areas, use `c.roundRect()` with a hint label inside

## Inline Fields

To place a fillable field inline with text:
```python
c.drawString(left, y, "Name: ")
name_x = left + c.stringWidth("Name: ", "Helvetica", 11)
c.acroForm.textfield(
    name="full_name",
    x=name_x, y=y - 3,
    width=5 * cm, height=0.55 * cm,
    # ... styling ...
)
after_x = name_x + 5 * cm + 2 * mm
c.drawString(after_x, y, " (please print clearly)")
```

## Running the Script

After writing the script, install the dependency (if needed) and run:
```bash
pip install reportlab && python /tmp/generate_pdf.py
```

## Reference Example

Below is a condensed example showing the key patterns — header, body text with an inline field, labeled form fields, a signature box, and a footer:

```python
from reportlab.lib.pagesizes import A4
from reportlab.pdfgen import canvas
from reportlab.lib.units import cm, mm
from reportlab.lib.colors import HexColor

OUTPUT = "/tmp/output.pdf"

width, height = A4
c = canvas.Canvas(OUTPUT, pagesize=A4)

# Colors
dark = HexColor("#1a1a1a")
accent = HexColor("#2c3e50")
light_grey = HexColor("#cccccc")
field_bg = HexColor("#f5f5f5")

left = 3 * cm
line_height = 0.55 * cm

# --- Header ---
y = height - 2.5 * cm
c.setFont("Helvetica-Bold", 22)
c.setFillColor(accent)
c.drawCentredString(width / 2, y, "Document Title")

y -= 0.6 * cm
c.setStrokeColor(accent)
c.setLineWidth(1.5)
c.line(left, y, width - 3 * cm, y)

# --- Body with inline field ---
y -= 1.4 * cm
c.setFont("Helvetica", 11)
c.setFillColor(dark)
c.drawString(left, y, "I, ")
name_x = left + c.stringWidth("I, ", "Helvetica", 11)

c.acroForm.textfield(
    name="inline_name",
    tooltip="Enter your full name",
    x=name_x, y=y - 3,
    width=5 * cm, height=line_height,
    borderWidth=0, fillColor=field_bg,
    textColor=dark, fontSize=11, fontName="Helvetica",
)

after_x = name_x + 5 * cm + 2 * mm
c.drawString(after_x, y, ", confirm the details below.")

# --- Labeled form fields ---
y -= 2 * cm
c.setFont("Helvetica-Bold", 11)
c.setFillColor(accent)
c.drawString(left, y, "Details")
c.setStrokeColor(light_grey)
c.setLineWidth(0.5)
c.line(left, y - 0.3 * cm, width - 3 * cm, y - 0.3 * cm)

y -= 1.4 * cm
c.setFont("Helvetica", 11)
c.setFillColor(dark)

c.drawString(left, y, "Email:")
c.acroForm.textfield(
    name="email", tooltip="Enter your email",
    x=left + 3.5 * cm, y=y - 3,
    width=10 * cm, height=line_height,
    borderWidth=0, fillColor=field_bg,
    textColor=dark, fontSize=11, fontName="Helvetica",
)

y -= 1.2 * cm
c.drawString(left, y, "Date:")
c.acroForm.textfield(
    name="date", tooltip="DD/MM/YYYY",
    x=left + 3.5 * cm, y=y - 3,
    width=5 * cm, height=line_height,
    borderWidth=0, fillColor=field_bg,
    textColor=dark, fontSize=11, fontName="Helvetica",
)

# --- Signature box ---
y -= 1.5 * cm
c.drawString(left, y, "Signature:")
sig_x = left + 3.5 * cm
sig_y = y - 1.2 * cm
c.setStrokeColor(light_grey)
c.setLineWidth(0.8)
c.setFillColor(field_bg)
c.roundRect(sig_x, sig_y, 10 * cm, 1.8 * cm, 3, fill=1, stroke=1)

c.setFont("Helvetica-Oblique", 8)
c.setFillColor(HexColor("#999999"))
c.drawString(sig_x + 3 * mm, sig_y + 3 * mm, "Sign here")

# --- Footer ---
c.setFont("Helvetica-Oblique", 8)
c.setFillColor(HexColor("#888888"))
c.drawCentredString(width / 2, 1.5 * cm, "Generated document — please review before signing.")

c.save()
print(f"PDF created: {OUTPUT}")
```

## Important Notes

- Each field MUST have a unique `name` parameter
- The y-coordinate in reportlab starts from the bottom of the page
- When positioning fields next to text, offset y by -3 to visually align with the text baseline
- For multi-page documents, call `c.showPage()` between pages
- Always call `c.save()` at the end to write the PDF
