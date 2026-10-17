---
name: file-conversion-toolkit
description: Convert documents and files locally, document-first, on Windows or Fedora Linux. Covers DOCX, XLSX, PPTX, PDF, Markdown, HTML, EPUB, ODT, RST and LaTeX via Pandoc, LibreOffice, MarkItDown, Typst, poppler, Ghostscript and Python (PyMuPDF / pdfplumber / pypdf); also images via ImageMagick and audio/video via FFmpeg. Use this skill whenever the user wants to convert, transform, export or reformat a document or file, asks which tool to use for a conversion, or says 转格式 / 转换 / 转成 / 导出 / 批量转换 / 把 PDF 转成 / 把 Word 转成. SKIP for pure cloud/API conversions.
---

# Document & File Conversion

Pick the right tool first, then run the shortest correct command.

Reply in the same language the user uses. Commands stay in English.

**Which tools exist is host-specific.** §0 probes the host you are on, §1 records
what the current fleet had when last probed, §2 routes on what is *installed
here* — never on what this document wishes were installed.

## 0. Probe the host first

```bash
for t in pandoc soffice markitdown typst magick convert ffmpeg pdftotext pdftoppm gs tesseract qpdf python3 python; do
  command -v "$t" >/dev/null 2>&1 && echo "OK   $t" || echo "MISS $t"
done
```

```powershell
'pandoc','soffice','markitdown','typst','magick','ffmpeg','python' | ForEach-Object {
  $p = Get-Command $_ -ErrorAction SilentlyContinue
  if ($p) { "OK   $_ -> $($p.Source)" } else { "MISS $_" }
}
```

Python's third-party modules are a **separate** check — they are installed only
on the Windows interpreter (§7):

```bash
python3 -c "import importlib.util as u; print([m for m in ('fitz','pdfplumber','pypdf','docx') if u.find_spec(m)])"
```

If the route you want needs a tool that is missing here, say so and offer the
alternative that *is* installed. Never emit a command for an absent tool.

## 1. What the fleet had (probed 2026-10-17)

| Tool | Windows desktop | Fedora (native) | Fedora (WSL) |
| --- | --- | --- | --- |
| Pandoc | 3.11 (`D:\Tools\pandoc`) | — | — |
| LibreOffice | 26.8.0 (`soffice`) | `soffice` | — |
| MarkItDown | 0.1.5 | — | — |
| Typst | 0.14.2 | — | — |
| ImageMagick | 7.1.2 (`magick`) | `magick` + `convert` | — |
| poppler | — | `pdftotext`, `pdftoppm` | — |
| Ghostscript | — | `gs` | — |
| FFmpeg | ✓ | — | — |
| Tesseract | — | — | — |
| Python | 3.13.7 (`D:\Python\python.exe`) **+ PyMuPDF, pdfplumber, pypdf, python-docx** | `python3`, no extra modules | `python3` 3.14, stdlib only |

What that means in practice:

- **Windows** carries the load with Pandoc + Python; the Windows box is the only
  host with MarkItDown, Typst and FFmpeg.
- **Native Fedora** does PDF work with poppler + Ghostscript and Office work with
  LibreOffice — but has **no Pandoc**, so "format A → B" falls to LibreOffice or
  to Python.
- **The WSL distro** has Python's standard library and nothing else from this
  list. A conversion request there usually means "say what is missing".
- Tools installed on Windows under `D:\Tools` and reached through WSL's inherited
  `PATH` are **`.exe` files**: `command -v pandoc` fails inside the distro, so do
  not treat the Windows toolchain as available from WSL.

Only the Windows commands in this file were benchmarked on the fleet; the Fedora
entries are presence-probed and use the tools' standard CLI. Re-probe after any
upgrade (§0) and update this table.

## 2. Router — capability → tool, preferring what is installed

| The task | First choice | If it is absent here |
| --- | --- | --- |
| A document in format A → format B | **Pandoc** | LibreOffice (Office ↔ Office, Office → PDF, → csv/html) |
| DOCX/XLSX/PPTX → PDF, or batch Office work | **LibreOffice** (`soffice`) | — |
| Anything → Markdown (to read / feed an LLM) | **MarkItDown** | `pandoc -t gfm`; for Office, `soffice --convert-to html` first |
| Markdown → a polished PDF | **Pandoc + Typst** | `pandoc -o out.docx` then `soffice --convert-to pdf` |
| PDF → text | `pdftotext` (poppler) | Python PyMuPDF; `markitdown`; `gs -sDEVICE=txtwrite` |
| PDF → tables | Python `pdfplumber` | — |
| PDF → one image per page | `pdftoppm` (poppler) | Python PyMuPDF; `gs -sDEVICE=png16m` |
| PDF merge / split | Python `pypdf` | `qpdf`; `gs` |
| An image → another image format | **ImageMagick** (`magick`) | — |
| Audio or video → another format | **FFmpeg** | — |

Documents are the main event. Images and media are §8 and §9.

## 3. Pandoc — format ↔ format

```powershell
pandoc in.md    -o out.docx                              # md → docx
pandoc in.md    -o out.docx --reference-doc=template.docx # apply your own styling
pandoc in.md    -o out.pptx                              # md → slides
pandoc in.md    -o out.epub
pandoc in.md    -o out.pdf  --pdf-engine=typst           # PDF: an engine is mandatory
pandoc in.docx  -o out.md   --extract-media=./media       # docx → md, pull images out
pandoc in.docx  -o out.html -s                            # -s = standalone
pandoc in.html  -o out.md
pandoc in.html  -o out.pdf --pdf-engine=typst            # web page → PDF
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

**PDF output always needs an engine.** On the Windows box the only engine is Typst
(`--pdf-engine=typst`) — there is no LaTeX there (§10.3). Where no engine and no
Typst exist, do not fight it: emit `out.docx` and let LibreOffice make the PDF.

**Markdown dialects.** Pandoc is the only tool here that reads Markdown, so name the dialect when it matters:

```powershell
pandoc in.docx -t gfm -o out.md          # GitHub-flavoured: tables, task lists, strikethrough
pandoc in.md   -f commonmark -o out.html # strict CommonMark
pandoc in.md   -f markdown_strict -o out.rst
pandoc in.md   -f markdown_mmd  -o out.md
```

`markdown_strict` **drops Pandoc's extensions** (footnotes, definition lists, grid tables). The default `markdown` output is not what GitHub renders either — to get Markdown that GitHub accepts, write `-t gfm` explicitly.

There is **no dedicated Markdown tool** on these machines: no formatter, linter, previewer, `cmark` or `mdbook`. Pandoc covers both directions; where Pandoc is absent, LibreOffice's `html` export is the nearest substitute.

## 4. LibreOffice — Office fidelity and batch

```powershell
soffice --headless --convert-to pdf --outdir . in.docx
soffice --headless --convert-to pdf --outdir . *.docx      # batch: one call, all files
soffice --headless --convert-to csv --outdir . in.xlsx
soffice --headless --convert-to docx --outdir . in.odt
soffice --headless --convert-to "pdf:writer_pdf_Export" --outdir . in.docx
```

- The command is `soffice` on Windows and on Fedora alike. This is the one
  document tool available on every host in the fleet.
- **Batch in one call, never in a loop.** Measured on Windows, 6 DOCX → PDF: one call **8.6 s**, a `ForEach-Object` loop **40–70 s**. Each start-up costs ~6 s and the user profile is single-instance.
- **`--convert-to csv` on a spreadsheet exports only the FIRST sheet.** The other sheets are dropped without warning. For every sheet, use `markitdown` (§5) where it exists, or convert to `pdf` / `html`, which both carry all sheets.
- soffice **overwrites** an existing output file without asking — a batch run can clobber files (§10.9).
- Reach for LibreOffice when **layout fidelity matters** (forms, styles, footnotes, tracked changes). Reach for Pandoc when you want Markdown or structured text.

## 5. MarkItDown — anything → Markdown

```powershell
markitdown in.docx -o out.md     # DOCX / PDF / PPTX / XLSX / HTML → Markdown
markitdown in.xlsx               # every sheet → a `## SheetName` section + GFM table
markitdown in.pdf                # to stdout
cmd /c "markitdown < in.docx"    # from stdin
```

- Best for clean PDF/DOCX/PPTX/XLSX/HTML → Markdown to hand to an LLM.
- **XLSX is the strongest case**: no sheet is lost, each becomes a `## SheetName` heading with a Markdown table. This is the best multi-sheet spreadsheet export on these machines.
- On **text-layer** PDFs, pdfplumber gives better tables than MarkItDown. On **scanned** PDFs neither works — there is no text layer to read (§11).
- **No direct PDF → DOCX anywhere here.** The workaround is two steps: `markitdown in.pdf -o tmp.md` then `pandoc tmp.md -o out.docx`.
- Installed on the Windows box only (§1). Elsewhere, fall back per §2.

## 6. Typst — the PDF engine

```powershell
pandoc in.md -o out.pdf --pdf-engine=typst
typst compile in.typ            # Typst's own markup → PDF
```

Typst exists on the Windows box to give Pandoc a PDF backend; there is **no LaTeX** there (`pdflatex`, `xelatex`, `tectonic` all absent), so never emit a Pandoc PDF command without an engine flag. On Linux, install neither and route through LibreOffice instead.

## 7. PDF extraction — pick the host's tool

Both routes are installed, on different hosts: **poppler/Ghostscript on Fedora**, **Python on Windows**.

Fedora (poppler + Ghostscript):

```bash
pdftotext -layout in.pdf -                  # text, layout preserved  → stdout
pdftoppm -r 200 -png in.pdf page            # one PNG per page: page-1.png, …
gs -sDEVICE=txtwrite -o out.txt in.pdf      # text via Ghostscript
gs -sDEVICE=png16m -r200 -o page-%d.png in.pdf
```

Windows (Python — third-party modules are installed **only** there; check with §0):

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

On Windows always call `D:\Python\python.exe` explicitly — the bare `python` command resolves to a Windows Store stub. On Fedora it is `python3` and the PDF modules are **not** installed: either use poppler/`gs` above, or install them (`python3 -m pip install --user pymupdf pdfplumber pypdf`) with the user's agreement.

Do not reach for ImageMagick to read a PDF: that needs Ghostscript *and* a permissive ImageMagick policy, so it is unreliable even where `gs` exists (§10.2).

## 8. Images — ImageMagick

```powershell
magick in.heic out.jpg                            # format convert
magick in.png -resize 50% out.webp
magick in.png -resize "1920x1080>" out.jpg        # quote: `>` is a geometry modifier
magick in.png -crop 800x600+100+50 +repage out.png
magick in.png -quality 85 -strip out.jpg
magick in.png watermark.png -gravity southeast -composite out.png
magick montage *.jpg -tile 4x -geometry +5+5 sheet.jpg    # output name goes last
magick identify -verbose in.png

# scans → one multi-page PDF (PDF *write* works, even though reading does not)
magick scan1.png scan2.png -page A4 scans.pdf
```

`magick` is ImageMagick 7 on both Windows and Fedora. On Fedora the legacy `convert`
alias also exists — but on Windows `convert` is a disk utility (§10.1), so always
spell it `magick` and the command is then correct on both.

Format support in the **Windows** build (re-check elsewhere with `magick -list format`):

- Read **and** write: `AVIF`, `WEBP`, `JXL`, `PSD`, `TIFF`, `SVG`, `EPS`, `PNG`, `JPEG`
- **Write-only**: `PDF` — writing works, reading fails (no Ghostscript on Windows)
- **Read-only**: `HEIC`, `HEIF`, `DNG`, `CR2`, `NEF`, `ARW` — so `HEIC → JPG` works, **`JPG → HEIC` does not**

Relevant to documents: this is how you convert a scanned page image, a screenshot, or an iPhone HEIC photo before feeding it to a document tool.

## 9. Audio / video — FFmpeg (secondary)

```powershell
ffmpeg -i in.mkv -c copy out.mp4                                  # remux, no re-encode
ffmpeg -i in.mkv -c:v libx264 -crf 23 -c:a aac -b:a 192k out.mp4 # re-encode
ffmpeg -i in.mp4 -vn -c:a libmp3lame -q:a 2 out.mp3               # extract audio
ffmpeg -i in.mp4 -vf scale=1280:-2 out.mp4
ffprobe -hide_banner -show_format -show_streams in.mkv
```

Add **`-y`** in scripts, or FFmpeg blocks on an overwrite prompt. FFmpeg is
installed on the Windows box only (§1) — it is also a prerequisite for
MarkItDown's audio transcription and for the HyperFrames renderer.

The one document-adjacent use: **audio → text**, via `markitdown` (§5), which needs FFmpeg on `PATH` *and* network.

## 10. Traps

1. **`convert` is not ImageMagick on Windows** — `C:\WINDOWS\system32\convert.exe` is a disk utility. Always use `magick` (which is also correct on Fedora, where `convert` would have worked).
2. **ImageMagick cannot be trusted with PDFs** — on Windows there is no Ghostscript at all; on Fedora `gs` exists but ImageMagick's `policy.xml` may still reject the PDF coder. Use poppler or PyMuPDF for anything PDF.
3. **No LaTeX anywhere in the fleet** — every Pandoc PDF command needs an engine flag; on Windows that means `--pdf-engine=typst`.
4. **No PDF → DOCX tool in the fleet.** Two-step through Markdown, or install one.
5. **`python` is a Store stub on Windows.** Use `D:\Python\python.exe`; on Fedora use `python3`.
6. **PATH changes need a new terminal.**
7. **MarkItDown needs FFmpeg + network** for audio only.
8. **`soffice --convert-to csv` exports only the first sheet** of a workbook, silently dropping the rest.
9. **`soffice` overwrites existing output** without asking — a batch run can clobber files.
10. **LibreOffice on Windows was unpacked with `msiexec /a`** — no registry entries, no file associations, no uninstaller ("uninstall" = delete `D:\Tools\LibreOffice`). A harmless `<prefix>` warning prints on each run.
11. **Python's PDF modules exist on Windows only.** `python3 -c "import fitz"` fails on Fedora until they are installed — probe before promising that route (§0, §7).
12. **Windows tools are not WSL tools.** `D:\Tools\*` are `.exe`s; inside the WSL distro they are not on `PATH` under their bare names.

## 11. Gaps

Not installed, document-relevant:

`Ghostscript` and `poppler` (on Windows) · `Pandoc`, `MarkItDown`, `Typst`, `FFmpeg` (on Fedora, both native and WSL) · `qpdf` · `Tesseract` (local OCR) · Pandoc's LaTeX engines (everywhere)

- **Scanned-PDF OCR**: no local option anywhere in the fleet. The PDF's text layer must already exist; otherwise use a cloud route.
- **PDF → DOCX with layout**: not possible locally. Only the Markdown round-trip, which loses layout.
- **`qpdf`** is worth installing where PDF repair/linearisation shows up: neither host has it.
