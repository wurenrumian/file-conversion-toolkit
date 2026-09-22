# file-conversion-toolkit

A **staging repository** for a local file-conversion routing skill.

`SKILL.md` is the skill itself: a document-first decision guide that routes a
conversion task to the correct tool on this machine, with verified capability
limits and known traps.

## Status

**Not installed.** This directory is a staging area so the skill can be reviewed
and version-controlled before it is wired into an agent. Nothing reads it
automatically.

## What it covers

Documents are the primary focus.

| Tool | Role |
| --- | --- |
| Pandoc | document format ↔ format (md, docx, html, epub, odt, rst, latex, pptx) |
| LibreOffice | Office fidelity and batch → PDF |
| MarkItDown | anything → Markdown; audio → transcript |
| Typst | Pandoc's PDF engine (there is no LaTeX on this machine) |
| Python (PyMuPDF / pdfplumber / pypdf) | PDF → text, tables, images; merge/split |
| ImageMagick | images (secondary) |
| FFmpeg | audio/video (secondary) |

Deliberately **out of scope**: general archives, build toolchains, and anything
that is not a conversion.

## Install later

Copy or symlink this directory into the agent's skills folder, then restart the
agent so it re-scans skills:

```powershell
# OpenCode
Copy-Item -Recurse . "$env:USERPROFILE\.config\opencode\skills\file-conversion-toolkit"

# Claude
Copy-Item -Recurse . "$env:USERPROFILE\.claude\skills\file-conversion-toolkit"
```

The directory name must match the `name:` field in the frontmatter
(`file-conversion-toolkit`).

## Maintenance

The tool paths, versions, and capability claims in `SKILL.md` were verified on
2026-09-22. Re-verify after upgrading any tool:

```powershell
pandoc --version; markitdown -v; typst --version; magick -version; ffmpeg -version
magick -list format | Select-String -Pattern 'HEIC|AVIF|WEBP|PSD|SVG|DNG|PDF'
pandoc --list-output-formats
```

## Related notes

- Tools are installed portably under `D:\Tools\` and exposed via the **user** PATH.
- Reinstalling FFmpeg is what restores MarkItDown's audio transcription.
- This machine has **no Ghostscript** and **no LaTeX**; the routing in the skill
  depends on both absences.
