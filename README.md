# file-conversion-toolkit

A local file-conversion **routing skill**: a document-first decision guide that
routes a conversion task to the correct tool on the host it is running on, with
capability limits and known traps.

`SKILL.md` is the skill itself. It is host-aware: §0 probes the machine, §1
records what the fleet had when last probed, and §2 routes on what is actually
installed rather than on an assumed toolchain.

## Status

Published from this repository and installed as a global `npx skills` skill:

```bash
npx skills add wurenrumian/file-conversion-toolkit --global --skill file-conversion-toolkit --yes
```

It is also declared in the dotfiles manifest (`dot_config/agent-skills/skills.json`),
whose install script performs that `npx skills add` on each host. Installed copies
under the agent skill directories are **derived** — change `SKILL.md` here, push,
then re-run the install script (or `npx skills update -g -y`) so the copies follow.

## What it covers

Documents are the primary focus.

| Tool | Role |
| --- | --- |
| Pandoc | document format ↔ format (md, docx, html, epub, odt, rst, latex, pptx) |
| LibreOffice | Office fidelity and batch → PDF |
| MarkItDown | anything → Markdown; audio → transcript |
| Typst | Pandoc's PDF engine on Windows (no LaTeX there) |
| poppler (`pdftotext`, `pdftoppm`) | PDF → text, PDF → page images on Linux |
| Ghostscript | PDF text/raster fallback on Linux |
| Python (PyMuPDF / pdfplumber / pypdf) | PDF → text, tables, images; merge/split (Windows) |
| ImageMagick | images (secondary) |
| FFmpeg | audio/video (secondary), MarkItDown audio, HyperFrames rendering |

Platform profiles as probed on 2026-10-17:

| | Windows desktop | Fedora (native) | Fedora (WSL) |
| --- | --- | --- | --- |
| Documents | Pandoc, LibreOffice, MarkItDown, Typst | LibreOffice | — |
| PDF | Python (PyMuPDF, pdfplumber, pypdf) | poppler, Ghostscript | — |
| Media | ImageMagick, FFmpeg | ImageMagick | — |
| Python | 3.13 + PDF/document modules | `python3`, stdlib | `python3`, stdlib |

Deliberately **out of scope**: general archives, build toolchains, and anything
that is not a conversion.

## Maintenance

Re-probe after upgrading any tool, and update §1 of `SKILL.md` with the date:

```bash
for t in pandoc soffice markitdown typst magick convert ffmpeg pdftotext pdftoppm gs tesseract qpdf python3; do command -v "$t" >/dev/null && echo "OK   $t" || echo "MISS $t"; done
```

```powershell
pandoc --version; markitdown -v; typst --version; magick -version; ffmpeg -version
magick -list format | Select-String -Pattern 'HEIC|AVIF|WEBP|PSD|SVG|DNG|PDF'
pandoc --list-output-formats
```

## Related notes

- On Windows the tools are installed portably under `D:\Tools\` and exposed via the **user** PATH; those are `.exe` files, so they are not callable by bare name from a WSL distro.
- Windows has **no Ghostscript** and **no LaTeX**; the Windows routing depends on both absences. Fedora is the mirror image: it has Ghostscript and poppler but no Pandoc, MarkItDown, Typst or FFmpeg.
- Reinstalling FFmpeg on Windows is what restores MarkItDown's audio transcription.
