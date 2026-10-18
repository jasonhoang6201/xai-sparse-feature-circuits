---
name: top-down-few-terms
description: explain a new idea top-down (what problem it solves first), with few technical terms per reply
metadata:
  type: feedback
---

Start each new concept with the problem it solves, in plain words, before any formula or symbol. Go down one level per reply. Do not pack several terms (IE, Eq. 5, ∇, decoder, encoder…) into one answer.

**Why:** the user said: "từ từ, đừng đưa quá nhiều thuật ngữ, hãy nói tôi biết nó giải quyết điều gì trước từ highlevel xuống… nếu nói 1 lúc quá nhiều thuật ngữ tôi sẽ không hiểu". Triggered by an edge explanation that mixed a comparison table, Eq. 5, gradients and decoder/encoder in one reply.

**How to apply:**
- Level 1: what problem this concept solves, one concrete sentence example, no formulas.
- Only go to mechanism and formulas after the user confirms level 1.
- Reuse terms the user already confirmed; introduce at most one new term per reply.
- Related: [[one-concept-at-a-time]], [[annotate-formula-symbols]].
