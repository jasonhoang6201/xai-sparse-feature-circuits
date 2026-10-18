---
name: annotate-formula-symbols
description: after every formula symbol or abbreviation (W_U, x_L, W_enc, f, vᵢ…) add its meaning in parentheses
metadata:
  type: feedback
---

Every time a formula symbol or abbreviation appears (W_U, x_L, W_enc, W_dec, b, f, fᵢ, vᵢ, x̂, ε, d_model, logits…), write its meaning in parentheses right after it, e.g. `W_U` (weight unembedding), `x_L` (residual stream ở layer cuối, L = last layer).

**Why:** the user said: "tôi chưa quen lắm với việc đọc x_U là gì, W_enc thì tôi hiểu là weight encoder, nên hãy (...) sau mỗi từ viết tắt đó để tôi đọc quen dần với công thức". They were confused reading `W_U` and `x_L` in the unembedding explanation.

**How to apply:** annotate at every occurrence in a reply (at least the first occurrence in each section), in chat and in generated HTML. Also explain subscript letters (U = unembedding, L = last layer, i = index). Keep doing it until the user says they no longer need it. Related: [[one-concept-at-a-time]].
