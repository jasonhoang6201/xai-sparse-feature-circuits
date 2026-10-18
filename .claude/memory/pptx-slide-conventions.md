---
name: pptx-slide-conventions
description: rules for building the seminar pptx — Google Slides import, real sub/superscript, topic titles, few words, keep the old deck's style
metadata:
  type: feedback
---

When building or editing the seminar `.pptx`:
- The file is imported into Google Slides. Use a font Google has (Arial); text boxes use word wrap with fixed width, no `wrap="none"`, no manual line breaks, no OMML equations.
- Formulas use real subscript/superscript (run `baseline`), never underscores (`x_clean` → x with subscript clean).
- Titles name the topic ("Indirect effect (IE)"), not a lead-in sentence.
- Few words, more diagrams, paper figures and paper numbers. Speech goes into speaker notes.
- Keep the old deck's style: white background, teal blob top-right, small grey section label + bold title, 01/02 badges, page number bottom-right.
- No large empty areas: content must fill the slide body (about y 1.9–6.8 in on 13.33×7.5); enlarge figures/cards or spread rows instead of leaving the lower part blank. Comparisons of numbers go in a table, not loose big-number stacks.
- When removing text leaves a card sparse, restyle instead of leaving empty boxes: e.g. paper-title chips (italic English title + author), a tinted "Hạn chế" strip, symmetric two-panel layouts. Check how the whole slide reads, not just the removed line.
- No grey explanatory sub-lines under items (e.g. "chỉ làm cho Pythia (App F)", step descriptions, citation examples). Keep only what the audience must see; move the rest into the speaker notes / `seminar-flow.md` speech.
- No summary/takeaway line at the bottom of a slide. If the point matters, show it in the slide's main content (a row, a tag, a heading); otherwise leave it for the presenter and put it in the speaker notes / `seminar-flow.md` speech.
- No left-border/stripe accents (the user dislikes them, including the old yellow bar callout). Callout = pale-yellow rounded box; headers = tinted rounded pill with colored bold text; highlighted card = white or tinted rounded box, no colored side stripe.
- No caption under diagrams or figures; put labels needed for reading (model, node count) above the figure as a heading.
- No "§" sign; write "Section 3".
- No "·" separators; use separate lines. Table of contents shows section titles only, no slide ranges or descriptions.
- Plain wording: prefer the English term over a stiff Vietnamese phrase ("interpretable", not "đọc được nghĩa"; "coarse-grained unit", not "đơn vị thô"). Avoid "probing"/"probe" (Vietnamese readers rarely use it); write "linear classifier". A paper's solution sits inside a labeled box ("Giải pháp của paper").
- Keep paper titles in English; keep domain terms in English ([[keep-technical-terms-english]]), numbers only from the paper ([[only-paper-numbers]]).

**Why:** the user listed these requirements on 2026-10-18 when asking to rebuild the pptx from `slide/seminar-flow.md`, and added "giữ style pptx hiện tại, chỉ đổi nội dung và flow".

**How to apply:** the build script lives only in the session scratchpad; regenerate with python-pptx from the old deck as template (theme, blobs) and write a new file in `slide/` rather than overwriting the existing `.pptx`.

The full ruleset and build kit now live in the global skill `~/.claude/skills/do-pptx` (SKILL.md, scripts/pptx_kit.py); use it for any new deck.
