---
name: only-paper-numbers
description: in slide/seminar files, write only numbers stated in the paper; no self-computed or web-sourced numbers
metadata:
  type: feedback
---

In seminar/slide material, every number must appear in the paper itself (`.claude/paper.txt`). Do not write numbers I computed (e.g. 19 × 32 769 node count), numbers from other sources (d_model of Pythia/Gemma, dates/venues of later work), or derived interpretations stated as numbers ("≈ 50% = random", "one order of magnitude").

**Why:** the user said "chỉ ghi số trong paper" after the first `seminar-flow.md` draft mixed in self-computed and external numbers.

**How to apply:** before writing a number into a slide file, grep `paper.txt` for it. If it is not there, describe the fact without the number, or cite the paper's own wording. Related: [[keep-technical-terms-english]].
