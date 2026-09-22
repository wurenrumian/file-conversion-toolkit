---
name: file-conversion-toolkit
description: Route any local file-conversion task to the right tool on this Windows machine — video, audio, image, or document. Covers FFmpeg, ImageMagick, Pandoc, LibreOffice, MarkItDown, Typst, and Python (Pillow / PyMuPDF / pypdf), with exact install paths, verified capability limits, and known traps. Use this skill whenever the user wants to convert, transcode, compress, resize, extract, merge, split, or reformat media or documents (mp4, mkv, webm, mov, avi, mp3, flac, aac, opus, wav, jpg, png, webp, avif, heic, jxl, tiff, psd, svg, raw, pdf, docx, xlsx, pptx, odt, html, md, epub, rst, latex), even if they do not name a tool. Also use it when the user asks which tool to use for a conversion, when a conversion command fails and needs a different approach, or when the task is "把这个转成那个" / "转格式" / "提取内容" / "压缩一下" / "批量转换". SKIP for pure cloud/API conversions or when a dedicated format skill (pdf, docx, xlsx, pptx) already owns the task.
---

# File Conversion Toolkit (this machine)

A routing guide. The job of this skill is to pick the **right tool first**, then run the shortest correct command.

## Language rule

Reply in the same language the user uses. Commands and code stay in English.

## 0. The one-line router

| If the file is… | Use |
| --- | --- |
| Video or audio (any container/codec) | **FFmpeg** |
| A bitmap image (raster) | **ImageMagick** (`magick`) |
| A PDF that needs rasterizing / text / tables | **Python: PyMuPDF / pdfplumber** |
| A document in another document format | **Pandoc** |
| An Office file needing real Office fidelity, or batch → PDF | **LibreOffice** (`soffice`) |
| Anything → Markdown (for reading/LLM input) | **MarkItDown** |
| Markdown → a polished PDF | **Pandoc + Typst** |
| An archive | **7-Zip** |

When two could work, prefer the one higher in this list only if the format matches; otherwise follow the table. See §2 for the full decision table.

## 1. Tool inventory on this machine

The main conversion tools are installed **portable-style on the D: drive** (`D:\Tools\`) and exposed via the **user** `PATH`. MarkItDown is a `uv` tool and Typst is a WinGet package, so those two live under `%USERPROFILE%`. Versions verified 2026-09-22.

| Tool | Executable | Version | Notes |
| --- | --- | --- | --- |
| FFmpeg | `D:\Tools\ffmpeg\bin\ffmpeg.exe` | `N-126755-g52f05ac780-20260922` (BtbN GPL) | + `ffprobe.exe`, `ffplay.exe` |
| ImageMagick | `D:\Tools\ImageMagick\magick.exe` | 7.1.2-31 Q16 x64 | portable build; `MAGICK_HOME` set |
| Pandoc | `D:\Tools\pandoc\pandoc.exe` | 3.11 (+server +lua) | 51 input / 76 output formats |
| LibreOffice | `D:\Tools\LibreOffice\program\soffice.exe` | 26.8.0 | also `soffice.com` |
| MarkItDown | `%USERPROFILE%\.local\bin\markitdown.exe` | 0.1.5 (uv tool) | needs FFmpeg for audio |
| Typst | `%LOCALAPPDATA%\Microsoft\WinGet\Packages\Typst.Typst_*\typst.exe` | 0.14.2 | Pandoc's PDF engine |
| Python | `D:\Python\python.exe` | 3.13.7 | Pillow 12.2.0, PyMuPDF 1.28.2, pypdf, pdfplumber, python-docx, reportlab, pypdfium2 |
| 7-Zip | `D:\7-Zip\7z.exe` | 25.01 | archives + `7z t` integrity check |
| MSYS2 | `D:\msys64` | — | `zstd`, `xz`, `brotli`, unix toolchain |

User `PATH` entries added for this toolkit:

```
D:\Tools\pandoc
D:\Tools\ImageMagick
D:\Tools\ffmpeg\bin
D:\Tools\LibreOffice\program
MAGICK_HOME = D:\Tools\ImageMagick
```

**After any PATH change the terminal must be restarted.** In a fresh terminal, `ffmpeg`, `ffprobe`, `pandoc`, `magick`, `soffice`, `markitdown`, `typst`, `7z` all resolve.

## 2. Decision table (task → tool → command)

### Video & audio — FFmpeg

| Task | Command |
| --- | --- |
| Remux, change container | `ffmpeg -i in.mkv -c copy out.mp4` |
| Re-encode H.264 + AAC | `ffmpeg -i in.mkv -c:v libx264 -crf 23 -preset medium -c:a aac -b:a 192k out.mp4` |
| H.265 / VP9 / AV1 | `-c:v libx265` / `-c:v libvpx-vp9 -crf 32 -b:v 0` / `-c:v libsvtav1` |
| Extract audio | `ffmpeg -i in.mp4 -vn -c:a copy out.m4a` (or `-c:a libmp3lame` / `-c:a libopus`) |
| Audio transcode | `ffmpeg -i in.flac -c:a libmp3lame -q:a 2 out.mp3` |
| Trim without re-encode | `ffmpeg -ss 00:01:00 -to 00:02:30 -i in.mp4 -c copy out.mp4` |
| Resize / scale | `-vf scale=1280:-2` |
| Extract frames | `ffmpeg -i in.mp4 -vf fps=1 out_%04d.png` |
| Video → GIF | `ffmpeg -i in.mp4 -vf "fps=12,scale=480:-1:flags=lanczos" out.gif` |
| Concat (same codec) | `ffmpeg -f concat -safe 0 -i list.txt -c copy out.mp4` |
| Inspect media | `ffprobe -hide_banner -show_format -show_streams in.mkv` |
| Silence/strip metadata | `-map_metadata -1` |

Add **`-y`** to every command run from a script or batch loop — otherwise FFmpeg stops and waits for an overwrite confirmation and the job hangs. Add `-hide_banner -loglevel error` to keep output clean.

### Images — ImageMagick (`magick`)

| Task | Command |
| --- | --- |
| Format convert | `magick in.heic out.jpg` |
| Resize (50%) | `magick in.png -resize 50% out.webp` |
| Resize to fit box, shrink only | `magick in.png -resize "1920x1080>" out.jpg` |
| Crop | `magick in.png -crop 800x600+100+50 +repage out.png` |
| Rotate / flip | `magick in.png -rotate 90 out.png` |
| Quality / strip metadata | `magick in.png -quality 85 -strip out.jpg` |
| Compose / watermark | `magick in.png watermark.png -gravity southeast -composite out.png` |
| Montage / contact sheet | `magick montage *.jpg -tile 4x -geometry +5+5 sheet.jpg` |
| Batch (PowerShell) | `Get-ChildItem *.heic \| ForEach-Object { magick $_.FullName ($_.BaseName + '.jpg') }` |
| Identify | `magick identify -verbose in.png` |

**Verified format support (this build):**

- Read **and** write: `AVIF`, `WEBP`, `JXL`, `PSD`, `TIFF`, `SVG` (RSVG 2.40.20), `MSVG`, `EPS`, `PNG`, `JPEG`, `MP4`
- **Write-only**: `PDF` — `magick in.png out.pdf` works, but **reading** a PDF fails (it shells out to `gswin64c.exe`, which is absent). Never use ImageMagick to rasterize a PDF; use PyMuPDF.
- **Read-only** (`r--`): `HEIC`, `HEIF` (libheif 1.23.2), `DNG`, `CR2`, `NEF`, `ARW` (LibRaw)
- So `HEIC → JPG` works; **`JPG → HEIC` does not**. Same for camera RAW: decode only.
- **Quote geometry modifiers containing `>` or `<`** in PowerShell: `-resize "1920x1080>"`. A backslash (`\>`) is bash syntax and is passed to ImageMagick verbatim here.

### PDF — Python, not ImageMagick

ImageMagick lists `PDF rw+`, but **Ghostscript is NOT installed**, so it can only *write* a PDF, never *read* one. Rasterizing a PDF with ImageMagick fails. Use Python instead.

| Task | Command |
| --- | --- |
| PDF → images (per page) | `D:\Python\python.exe -c "import pymupdf; d=pymupdf.open('in.pdf'); [p.get_pixmap(dpi=200).save(f'p{i+1}.png') for i,p in enumerate(d)]"` |
| PDF → text | `D:\Python\python.exe -c "import pymupdf; d=pymupdf.open('in.pdf'); print(chr(10).join(p.get_text() for p in d))"` |
| PDF tables | `D:\Python\python.exe -c "import pdfplumber; pdf=pdfplumber.open('in.pdf'); print(pdf.pages[0].extract_table())"` |
| Merge / split / rotate / encrypt | `D:\Python\python.exe -c "from pypdf import PdfWriter; w=PdfWriter(); w.append('a.pdf'); w.append('b.pdf'); w.write('merged.pdf')"` |

### Documents — Pandoc

| Task | Command |
| --- | --- |
| md → docx | `pandoc in.md -o out.docx` |
| docx → md | `pandoc in.docx -o out.md --extract-media=./media` |
| md/html/docx/odt/epub/rst/latex 互转 | `pandoc in.<ext> -o out.<ext>` |
| md → PDF (no LaTeX needed) | `pandoc in.md -o out.pdf --pdf-engine=typst` |
| Standalone HTML with CSS | `pandoc in.md -o out.html --standalone --css=style.css` |
| With TOC / numbering | `--toc --number-sections` |
| GFM input / output | `-f gfm` / `-t gfm` |
| Extract media from docx | `--extract-media=./media` |

**PDF output requires an explicit engine here.** Only **Typst 0.14.2** is installed; there is **no LaTeX** (`pdflatex`/`xelatex`/`tectonic` all absent). Always pass `--pdf-engine=typst`.

### Office documents — LibreOffice

| Task | Command |
| --- | --- |
| DOCX → PDF | `soffice --headless --convert-to pdf --outdir . in.docx` |
| XLSX → CSV | `soffice --headless --convert-to csv --outdir . in.xlsx` |
| PPTX → PDF | `soffice --headless --convert-to pdf --outdir . in.pptx` |
| Batch (whole folder) | `soffice --headless --convert-to pdf --outdir . *.docx` — one process for all files |
| Pick a filter explicitly | `--convert-to "pdf:writer_pdf_Export"` |

**Prefer one `soffice` call with many files over a PowerShell `ForEach-Object` loop.** Each `soffice` start-up costs ~6 s and only one instance can hold the user profile at a time, so a loop is dramatically slower.

Use LibreOffice when **Office fidelity matters** or when converting **many Office files at once**. Use Pandoc instead when the goal is Markdown/structured text.

### Anything → Markdown — MarkItDown

| Task | Command |
| --- | --- |
| Convert to Markdown | `markitdown in.pdf -o out.md` |
| Print to stdout | `markitdown in.docx` |
| Audio → transcript | `markitdown in.mp3` |
| From stdin | `cmd /c "markitdown < in.pdf"` — **not** `Get-Content -Raw \| markitdown`, which corrupts binary through PowerShell's text pipeline |

Best for: clean PDF/DOCX/PPTX/XLSX/HTML → Markdown to feed an LLM. **Weak on tables** (pdfminer-based). For table-heavy or scanned PDFs, prefer PyMuPDF/pdfplumber, or a cloud OCR route.

Audio transcription chain: `pydub → ffmpeg → speech_recognition → Google Speech API`. It therefore needs **FFmpeg on PATH** *and* **network** (it exits through the Clash proxy via the `HTTP_PROXY`/`HTTPS_PROXY` user env vars). It only handles audio — extract the audio track from a video first:

```powershell
ffmpeg -i video.mp4 -vn -c:a mp3 audio.mp3
markitdown audio.mp3
```

### Archives — 7-Zip

| Task | Command |
| --- | --- |
| Extract | `7z x archive.7z -o<dir> -y` |
| Create | `7z a archive.7z <files>` |
| Test integrity | `7z t archive.zip` |

## 3. Known traps (each one has bitten already)

1. **`convert` is NOT ImageMagick.** `C:\WINDOWS\system32\convert.exe` is a Windows disk-conversion utility. The ImageMagick portable dir has **no** `convert.exe` either — always use `magick`.
2. **No Ghostscript** → ImageMagick cannot rasterize PDF. Use PyMuPDF / pypdfium2.
3. **HEIC/HEIF and camera RAW are read-only** in this ImageMagick build. Decoding is fine; encoding will fail.
4. **No LaTeX** → `pandoc x.md -o x.pdf` fails. Always add `--pdf-engine=typst`.
5. **`python` resolves to the WindowsApps stub**, not the real interpreter, even though `D:\Python` is on the user PATH (system PATH wins). Always call `D:\Python\python.exe` explicitly, or fix PATH ordering.
6. **PATH changes need a new terminal.** Never debug a "command not found" before checking this.
7. **`markitdown` audio needs FFmpeg + network.** Removing FFmpeg silently breaks it (it degrades to a `RuntimeWarning` and then fails at transcription).
8. **The LibreOffice here was unpacked with `msiexec /a`** (administrative extract) to avoid admin rights and C: usage. Consequences: no registry entries, **no file associations, no Start Menu entry, no uninstaller** — "uninstall" means deleting `D:\Tools\LibreOffice`. It prints a harmless `Could not find platform independent libraries <prefix>` warning on each run.
9. **BtbN FFmpeg has no `flite` filter** (the gyan.dev build did). Irrelevant for transcoding; only affects text-to-speech generation.
10. **Downloading tools: prefer Chinese mirrors.** Verified speeds — Tsinghua TUNA **5.7 MB/s**, `ghfast.top` 0.13 MB/s, `ghproxy.net` 0.01 MB/s, `gh-proxy.com` 403, direct `gyan.dev` hangs. LibreOffice → `mirrors.tuna.tsinghua.edu.cn/libreoffice/`; GitHub releases → `https://ghfast.top/https://github.com/...`.

## 4. Not installed (know the gap)

If a task needs one of these, say so instead of inventing a command:

`Ghostscript` · `Poppler` (`pdftoppm`, `pdftotext`) · `qpdf` · `pdftk` · `HandBrakeCLI` · `MKVToolNix` (`mkvmerge`) · `Calibre` (`ebook-convert`) · `SoX` · `LaTeX` (`pdflatex`, `xelatex`, `tectonic`) · `GIMP` · `Inkscape` · `Tesseract` (local OCR) · `ExifTool`

For **scanned-PDF OCR** there is currently no local option. Either install Tesseract, or use a cloud route (e.g. the MinerU open API).

## 5. Environment facts

- Proxy: Clash Verge on `127.0.0.1:7890`; `HTTP_PROXY`/`HTTPS_PROXY` are set as user env vars, so Python and `curl` inherit it.
- `D:\Tools\_downloads` holds the original installers (kept as an offline backup).
- Git 2.54.0 is available; the D: drive is where portable software lives on this machine.
