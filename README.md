# file-conversion-toolkit

A **staging repository** for a local file-conversion routing skill.

`SKILL.md` is the skill itself: a decision guide that routes any media/document
conversion task to the correct tool on this machine, with verified capability
limits and known traps.

## Status

**Not installed.** This directory is a staging area so the skill can be reviewed
and version-controlled before it is wired into an agent. Nothing reads it
automatically.

## What it covers

| Tool | Role |
| --- | --- |
| FFmpeg | video & audio |
| ImageMagick | bitmap images |
| Pandoc | document format interconversion |
| LibreOffice | Office fidelity / batch → PDF |
| MarkItDown | anything → Markdown, audio → transcript |
| Typst | Pandoc's PDF engine (no LaTeX on this machine) |
| Python (Pillow / PyMuPDF / pdfplumber / pypdf) | PDF rasterizing, text, tables, scripting |
| 7-Zip | archives |

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
ffmpeg -version; pandoc --version; magick -version; markitdown -v; typst --version
magick -list format | Select-String -Pattern 'HEIC|AVIF|WEBP|PSD|SVG|DNG'
```

## Related notes

- Tools are installed portably under `D:\Tools\` and exposed via the **user** PATH.
- Reinstalling FFmpeg is what restores MarkItDown's audio transcription.
- This machine has **no Ghostscript** and **no LaTeX**; the skill's routing
  depends on those absences.
