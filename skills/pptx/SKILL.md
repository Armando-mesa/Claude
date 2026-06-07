---
name: pptx
description: "Use this skill any time a .pptx file is involved in any way — as input, output, or both. This includes: creating slide decks, pitch decks, or presentations; reading, parsing, or extracting text from any .pptx file (even if the extracted content will be used elsewhere, like in an email or summary); editing, modifying, or updating existing presentations; combining or splitting slide files; working with templates, layouts, speaker notes, or comments. Trigger whenever the user mentions 'deck,' 'slides,' 'presentation,' or references a .pptx filename, regardless of what they plan to do with the content afterward. If a .pptx file needs to be opened, created, or touched, use this skill."
license: Proprietary. LICENSE.txt has complete terms
---

# PPTX Skill

## Quick Reference

| Task | Guide |
|------|-------|
| Read/analyze content | `python -m markitdown presentation.pptx` |
| Edit or create from template | Read [editing.md](editing.md) |
| Create from scratch | Read [pptxgenjs.md](pptxgenjs.md) |

## Reading Content

```bash
python -m markitdown presentation.pptx   # Text extraction
python scripts/thumbnail.py presentation.pptx  # Visual overview
python scripts/office/unpack.py presentation.pptx unpacked/  # Raw XML
```

## Design Ideas

**Don't create boring slides.** Plain bullets on white won't impress.

- **Pick a bold color palette** specific to THIS topic
- **Dominance over equality**: One color 60-70%, 1-2 supporting, one sharp accent
- **Dark/light contrast**: Dark for title+conclusion, light for content ("sandwich")
- **Commit to a visual motif**: ONE distinctive element repeated across all slides
- **Every slide needs a visual element** — image, chart, icon, or shape

**NEVER use accent lines under titles** — hallmark of AI-generated slides.

### Typography

| Element | Size |
|---------|------|
| Slide title | 36-44pt bold |
| Section header | 20-24pt bold |
| Body text | 14-16pt |

## QA (Required)

```bash
python -m markitdown output.pptx
python -m markitdown output.pptx | grep -iE "xxxx|lorem|ipsum"
```

**Visual QA with subagents:**
```bash
python scripts/office/soffice.py --headless --convert-to pdf output.pptx
pdftoppm -jpeg -r 150 output.pdf slide
```

Inspect for: overlapping elements, text overflow, low contrast, leftover placeholders, uneven gaps.

**Verification loop**: Generate → Convert → Inspect → Fix → Re-verify. Do not declare success without at least one fix-and-verify cycle.

## Dependencies

- `pip install "markitdown[pptx]"` — text extraction
- `npm install -g pptxgenjs` — creating from scratch
- LibreOffice + Poppler — conversion to images
