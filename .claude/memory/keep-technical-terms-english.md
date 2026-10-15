---
name: keep-technical-terms-english
description: Vietnamese explanations keep domain-specific ML/interp terms in English but translate common academic words
metadata:
  node_type: memory
  type: feedback
  originSessionId: fdbb1792-ae17-4a5e-b39c-7651530f1dae
  modified: 2026-10-14T12:58:28.910Z
---

When writing Vietnamese explanations:
- **Keep in English** the domain-specific terms that have no common Vietnamese equivalent, or whose translation reads strangely. Examples: interpretability, interpretable, unit, feature, activation, circuit, SAE, dictionary learning, reconstruction, causal effect, intervention, patching, ablation, residual stream, superposition, polysemantic, spurious correlation, subject-verb agreement, relative clause, cluster.
- **Translate to Vietnamese** common academic or general words whose translation is natural and well known. Examples: hypothesis → giả thuyết, behavior → hành vi, threshold → ngưỡng, mean → trung bình, approximate → xấp xỉ, probability → xác suất, label → nhãn, gender → giới tính, subject/verb/noun → chủ ngữ/động từ/danh từ, direction → hướng, dimension → chiều, parameter → tham số.

**Why:** the user said "hypothesis tôi hiểu là giả thuyết thì có thể dịch", while terms like "interpretability", "unit interpretable" should stay in English. Translating everything makes the text awkward. Keeping everything in English is the opposite extreme and also wrong.

**How to apply:** this applies to chat answers and to generated files. Translate the word when a Vietnamese reader knows it without context; otherwise keep it in English.

**Use the standard ML vocabulary, never a home-made Vietnamese description of it.** Say "linear layer", "weight matrix", "dot product", "projection", "hidden layer", "embedding matrix", "forward pass", "loss", "gradient descent", "activation function", not "lớp nhân", "phép nhân từng tọa độ rồi cộng", "đọc khái niệm ra". The user said: "cứ dùng thuật ngữ hay các tưởng tượng của người trong ngành AI, vì nếu bạn nói lớp nhân hay gì đó thì nó k chuyên ngành và k hợp lí... dùng ngôn ngữ phổ thông của người trong ngành AI thì nói chuyện người khác mới hiểu được". Goal: the user must be able to talk with other ML people using these words. A short Vietnamese gloss after the term is fine the first time.

**Translate by meaning, not word for word.** The user rejected "không cần người viết giả thuyết trước" and wanted "người nghiên cứu không phải đưa ra giả thuyết từ trước". Write each sentence the way a Vietnamese person would say it: add the subject and connecting words, and rearrange the clause order when the result reads more naturally. Never map English → Vietnamese one word at a time.
