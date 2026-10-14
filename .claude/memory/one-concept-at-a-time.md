---
name: one-concept-at-a-time
description: "When explaining, cover one concept per reply and wait for the user to confirm before moving on"
metadata:
  node_type: memory
  type: feedback
  originSessionId: fdbb1792-ae17-4a5e-b39c-7651530f1dae
  modified: 2026-10-14T17:15:30.463Z
---

When answering a conceptual question, explain only one concept per reply, keep it short, and wait until the user confirms they understand before introducing the next one.

**Why:** the user asked about residual stream and received residual connection, residual stream, per-token vectors, cross-attention, and SAE placement all at once. They said: "từ từ, đi từng cái thôi, tôi rõ rồi mới đi tiếp, k đi nhiều khái niệm cùng lúc".

**How to apply:** when a question involves several concepts, choose the first one the others depend on. Explain only that concept, then end with one line naming the next concept and asking whether to continue. Do not stack extra sections. Related: [[keep-technical-terms-english]].
