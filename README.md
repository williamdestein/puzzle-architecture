# doc/

Design documentation for Puzzlefin.

## Layout

```
doc/
  design.md      Main design document (text + Mermaid diagrams, all in Git as text)
  diagrams/      Editable diagram source (draw.io XML, PlantUML) — text, diffable
  images/        Exported/binary assets: SVG (diagrams), PNG/JPEG (screenshots)
```

## Conventions

- Prefer **Mermaid** (fenced code in `design.md`) for architecture, sequence,
  and ER diagrams — the diagram *is* text in the doc, so it never drifts out of
  sync and diffs cleanly.
- Use **draw.io** (`diagrams/*.drawio`) for freeform pictures; export SVG into
  `images/` and reference that from the doc.
- Use **SVG** for diagrams (crisp at any zoom, diffable) and **PNG/JPEG** only
  for screenshots/photos.
- If binary assets get large or numerous, enable **Git LFS**.

## Rendering

- GitHub/GitLab render `design.md` (including Mermaid) natively.
- Local preview: VS Code with a Markdown + Mermaid preview extension.
- PDF/DOCX: `pandoc design.md -o design.pdf --toc` (add a Mermaid filter such
  as `mermaid-filter` to render diagrams inline).
