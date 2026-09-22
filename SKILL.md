---
name: file-conversion-toolkit
description: Convert documents and files locally on this Windows machine, document-first. Covers DOCX, XLSX, PPTX, PDF, Markdown, HTML, EPUB, ODT, RST and LaTeX via Pandoc, LibreOffice, MarkItDown, Typst and Python (PyMuPDF / pdfplumber / pypdf); also images via ImageMagick and audio/video via FFmpeg. Use this skill whenever the user wants to convert, transform, export or reformat a document or file, asks which tool to use for a conversion, or says 转格式 / 转换 / 转成 / 导出 / 批量转换 / 把 PDF 转成 / 把 Word 转成. SKIP for pure cloud/API conversions.
---

# Document & File Conversion (this machine)

Pick the right tool first, then run the shortest correct command.

Reply in the same language the user uses. Commands stay in English.

## 0. Router

| The task | Tool |
| --- | --- |
| A document in format A → format B | **Pandoc** |
| DOCX/XLSX/PPTX → PDF, or batch Office work | **LibreOffice** |
| Anything → Markdown (to read / feed an LLM) | **MarkItDown** |
| Markdown → a polished PDF | **Pandoc + Typst** |
| PDF → text / tables / images | **Python (PyMuPDF, pdfplumber)** |
| An image → another image format | **ImageMagick** |
| Audio or video → another format | **FFmpeg** |

Documents are the main event. Images and media are §3 and §4.

## 1. Tools

| Tool | Command | Version | Notes |
| --- | --- | --- | --- |
| Pandoc | `pandoc` | 3.11 | 51 input / 76 output formats |
| LibreOffice | `soffice` | 26.8.0 | Office fidelity, batch → PDF |
| MarkItDown | `markitdown` | 0.1.5 | anything → Markdown |
| Typst | `typst` | 0.14.2 | Pandoc's PDF engine; **no LaTeX here** |
| Python | `D:\Python\python.exe` | 3.13.7 | PyMuPDF 1.28.2, pdfplumber, pypdf, python-docx |
| ImageMagick | `magick` | 7.1.2-31 Q16 | |
| FFmpeg | `ffmpeg` / `ffprobe` | N-126755 (2026-09-22) | |

All of the above resolve from a **fresh terminal**. After any `PATH` change, restart the terminal first.

## 2. Documents

### 2.1 Pandoc — format ↔ format

```powershell
pandoc in.md    -o out.docx                              # md → docx
pandoc in.md    -o out.docx --reference-doc=template.docx # apply your own styling
pandoc in.md    -o out.pptx                              # md → slides
pandoc in.md    -o out.epub
pandoc in.md    -o out.pdf  --pdf-engine=typst           # PDF: engine is mandatory here
pandoc in.docx  -o out.md   --extract-media=./media       # docx → md, pull images out
pandoc in.docx  -o out.html -s                            # -s = standalone
pandoc in.html  -o out.md
pandoc in.tex   -o out.docx
pandoc in.csv   -o out.md
pandoc in.odt   -o out.docx
pandoc in.<X>   -o out.<Y>                                # any supported pair
```

Useful flags: `-s` (standalone), `--toc`, `--number-sections`, `--metadata title="…"`, `-f gfm` / `-t gfm`.

Discover what it supports:

```powershell
pandoc --list-input-formats
pandoc --list-output-formats
```

**PDF output always needs `--pdf-engine=typst`.** There is no LaTeX on this machine.

### 2.2 LibreOffice — Office fidelity and batch

```powershell
soffice --headless --convert-to pdf --outdir . in.docx
soffice --headless --convert-to pdf --outdir . *.docx      # batch: one call, all files
soffice --headless --convert-to csv --outdir . in.xlsx
soffice --headless --convert-to docx --outdir . in.odt
soffice --headless --convert-to "pdf:writer_pdf_Export" --outdir . in.docx
```

- Prefer **one call with many files** over a `ForEach-Object` loop — each start-up costs ~6 s and the user profile is single-instance, so a loop is dramatically slower.
- Reach for LibreOffice when **layout fidelity matters** (forms, styles, footnotes, tracked changes) or when converting **many Office files**. Reach for Pandoc when you want Markdown or structured text.

### 2.3 MarkItDown — anything → Markdown

```powershell
markitdown in.docx -o out.md     # DOCX / PDF / PPTX / XLSX / HTML → Markdown
markitdown in.pdf                # to stdout
cmd /c "markitdown < in.docx"    # from stdin
```

- Best for clean PDF/DOCX/PPTX/XLSX/HTML → Markdown to hand to an LLM.
- **Weak on tables** (pdfminer-based). For table-heavy or scanned PDFs use pdfplumber, or extract with PyMuPDF first.
- **No direct PDF → DOCX anywhere on this machine.** The workaround is two steps: `markitdown in.pdf -o tmp.md` then `pandoc tmp.md -o out.docx`.

### 2.4 Typst — the PDF engine

```powershell
pandoc in.md -o out.pdf --pdf-engine=typst
typst compile in.typ            # Typst's own markup → PDF
```

Typst is only installed to give Pandoc a PDF backend. There is **no LaTeX** (`pdflatex`, `xelatex`, `tectonic` all absent), so never emit a Pandoc PDF command without the engine flag.

### 2.5 PDF extraction — Python, not ImageMagick

ImageMagick **cannot read a PDF** here (it shells out to `gswin64c.exe`, which is absent). Use Python:

```powershell
# PDF → text
D:\Python\python.exe -c "import pymupdf; d=pymupdf.open('in.pdf'); print(chr(10).join(p.get_text() for p in d))"

# PDF → one image per page
D:\Python\python.exe -c "import pymupdf; d=pymupdf.open('in.pdf'); [p.get_pixmap(dpi=200).save(f'p{i+1}.png') for i,p in enumerate(d)]"

# PDF → tables
D:\Python\python.exe -c "import pdfplumber; pdf=pdfplumber.open('in.pdf'); print(pdf.pages[0].extract_table())"

# merge PDFs
D:\Python\python.exe -c "from pypdf import PdfWriter; w=PdfWriter(); w.append('a.pdf'); w.append('b.pdf'); w.write('merged.pdf')"
```

Always call `D:\Python\python.exe` explicitly — the bare `python` command resolves to a Windows Store stub.

## 3. Images — ImageMagick

```powershell
magick in.heic out.jpg                            # format convert
magick in.png -resize 50% out.webp
magick in.png -resize "1920x1080>" out.jpg        # quote: `>` is a geometry modifier
magick in.png -crop 800x600+100+50 +repage out.png
magick in.png -quality 85 -strip out.jpg
magick in.png watermark.png -gravity southeast -composite out.png
magick montage *.jpg -tile 4x -geometry +5+5 sheet.jpg
magick identify -verbose in.png
```

Format support in this build:

- Read **and** write: `AVIF`, `WEBP`, `JXL`, `PSD`, `TIFF`, `SVG`, `EPS`, `PNG`, `JPEG`
- **Write-only**: `PDF` — writing works, reading fails (no Ghostscript)
- **Read-only**: `HEIC`, `HEIF`, `DNG`, `CR2`, `NEF`, `ARW` — so `HEIC → JPG` works, **`JPG → HEIC` does not**

Relevant to documents: this is how you convert a scanned page image, a screenshot, or an iPhone HEIC photo before feeding it to a document tool.

## 4. Audio / video — FFmpeg (secondary)

```powershell
ffmpeg -i in.mkv -c copy out.mp4                                  # remux, no re-encode
ffmpeg -i in.mkv -c:v libx264 -crf 23 -c:a aac -b:a 192k out.mp4 # re-encode
ffmpeg -i in.mp4 -vn -c:a libmp3lame -q:a 2 out.mp3               # extract audio
ffmpeg -i in.mp4 -vf scale=1280:-2 out.mp4
ffprobe -hide_banner -show_format -show_streams in.mkv
```

Add **`-y`** in scripts, or FFmpeg blocks on an overwrite prompt.

The one document-adjacent use: **audio → text**, via `markitdown` (§2.3), which needs FFmpeg on `PATH` *and* network.

## 5. Traps

1. **`convert` is not ImageMagick** — `C:\WINDOWS\system32\convert.exe` is a Windows disk utility. Always use `magick`.
2. **No Ghostscript** → ImageMagick cannot rasterize a PDF. Use PyMuPDF.
3. **No LaTeX** → every Pandoc PDF command needs `--pdf-engine=typst`.
4. **No PDF → DOCX tool.** Two-step through Markdown, or install one.
5. **`python` is a Store stub.** Use `D:\Python\python.exe`.
6. **PATH changes need a new terminal.**
7. **MarkItDown needs FFmpeg + network** for audio only.
8. **LibreOffice was unpacked with `msiexec /a`** — no registry entries, no file associations, no uninstaller ("uninstall" = delete `D:\Tools\LibreOffice`). A harmless `<prefix>` warning prints on each run.

## 6. Gaps

Not installed, document-relevant:

`Ghostscript` · `Poppler` (`pdftotext`, `pdftoppm`) · `qpdf` · `Tesseract` (local OCR) · `pandoc`'s LaTeX engines

- **Scanned-PDF OCR**: no local option. The PDF's text layer must already exist; otherwise use a cloud route.
- **PDF → DOCX with layout**: not possible locally. Only the Markdown round-trip, which loses layout.
