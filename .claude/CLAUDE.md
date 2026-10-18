# Project: studying "Sparse Feature Circuits"

The paper is "Sparse Feature Circuits: Discovering and Editing Interpretable Causal Graphs in Language Models" (Marks, Rager, Michaud, Belinkov, Bau, Mueller; ICLR 2025; arXiv 2403.19647).

- The PDF is `xai.pdf`, and a plain-text copy is `.claude/paper.txt`. Look up numbers there before quoting them.
- The user learns from the keywords up to the overall method. They read `study/keywords-slides.html` one slide at a time and ask about anything unclear.
- Answer in Vietnamese.

## Files
- `study/keywords-slides.html` is the main study deck: 36 slides, light theme, Be Vietnam Pro font. ←/→/Space navigate, M opens the menu, `#n` jumps to a slide.
  - Groups: Mở đầu; 1 Bối cảnh; 2 SAE; 3 Causal effect; 4 Tìm circuit; 5 SHIFT; 6 Unsupervised; Tổng hợp.
- Do not overwrite `study/giai-thich-paper.html`, `research/`, the `.pptx` files in `slide/`, or `video/`.
- Folder layout: `study/` holds the HTML files for learning (deck + diagrams). `slide/` holds the seminar material (`.pptx` files, `seminar-flow.md`).
- `.claude/learning-progress.md` lists the concepts the user has already confirmed and the next concept to explain. Read it at the start of a session and update it after each concept the user confirms.

## Output
- Never publish an Artifact unless the user explicitly says "artifact". Study HTML goes into `study/`; seminar/slide material goes into `slide/`.

## Memory
- Memory for this project lives only in `.claude/memory/` in this folder. The user syncs it across machines via git.
- This overrides the default auto-memory location. Do not write memories to `~/.claude/projects/.../memory/` or anywhere else outside this folder.
- Memory format: one fact per file, with frontmatter (`name`, `description`, `metadata.type`: user | feedback | project | reference). For feedback and project memories, add **Why:** and **How to apply:** lines.
- After creating a file, add a one-line pointer to `.claude/memory/MEMORY.md`: `- [Title](file.md) — hook`.
- Before creating a memory, check whether an existing file already covers it. Update that file instead of creating a duplicate.

@memory/MEMORY.md

## Rules
@rules/one-concept-at-a-time.md
@rules/term-language.md
@rules/writing-style.md
@learning-progress.md
