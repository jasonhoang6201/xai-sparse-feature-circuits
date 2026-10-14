# Bảng đối chiếu — dùng trực tiếp làm slide

Mỗi bảng ở đây có thể copy thành một slide. Số liệu đều lấy từ paper hoặc từ nguồn ghi trong file 05.

---

## Bảng 1 — SFC đứng ở đâu giữa các phương pháp tìm circuit

| | Đơn vị phân tích | Cần giả thuyết trước? | Cách tính effect | Chi phí | Đơn vị có diễn giải được? |
|---|---|---|---|---|---|
| Activation patching thủ công (IOI, greater-than) | attention head, MLP | có — người chọn component để thử | patching chính xác | 1 forward pass / node | không (polysemantic) |
| **ACDC** (Conmy 2023) | head, MLP | không cho circuit, nhưng có cho task | patching chính xác trên từng edge | tỉ lệ với số edge | không |
| **EAP** (Syed 2023) | head, MLP | như trên | attribution patching | 2 fwd + 1 bwd | không |
| **EAP-IG** (Hanna 2024) | head, MLP | như trên | integrated gradients | ×N so với EAP | không |
| **SFC** (Marks 2024) | **SAE feature + SAE error, theo vị trí token** | **không** | attribution patching hoặc IG | 2 fwd + 1 bwd (atp), ×10 (ig) | **có** |
| **Attribution graphs** (Anthropic 2025) | cross-layer transcoder feature | không | hiệu ứng tuyến tính trên một prompt | — | có |

Câu chốt cho slide: SFC là ô duy nhất trong bảng có **cả** "không cần giả thuyết trước" **và** "đơn vị diễn giải được", với chi phí không tăng theo số node.

---

## Bảng 2 — Feature circuit vs Neuron circuit (kết quả §3)

| Tiêu chí | Feature circuit | Neuron circuit | Tỉ lệ |
|---|---|---|---|
| Pythia-70M: số node giải thích phần lớn hành vi | ~100 | ~1 500 | 15× |
| Gemma-2-2B: số node giải thích phần lớn hành vi | ~500 | ~50 000 | 100× |
| Điểm interpretability, đơn vị ngẫu nhiên | 52.8 | 32.6 | +20.2 |
| Điểm interpretability, top-30 trên agreement | 62.3 | 30.2 | +32.1 |
| Điểm interpretability, top-30 trên Bias in Bios | 81.5 | 36.0 | +45.5 |
| Completeness: số node cần ablate để xoá hiệu năng | vài node | hàng trăm (Pythia) → hàng nghìn (Gemma) | |

Điểm đáng chú ý ẩn trong bảng: với neuron, đơn vị **quan trọng nhất về mặt nhân quả** (30.2) lại **khó hiểu hơn** đơn vị ngẫu nhiên (32.6). Với feature thì ngược lại (62.3 và 81.5 so với 52.8). Đây là bằng chứng mạnh nhất trong paper cho luận điểm "neuron không phải đơn vị đúng".

---

## Bảng 3 — SHIFT so với literature spurious correlation

| Phương pháp | Cần nhãn nhóm? | Cần dữ liệu phân định? | Cần biết trước tín hiệu spurious là gì? | Nguồn tín hiệu phân định |
|---|---|---|---|---|
| Group DRO (Sagawa 2020) | có | có | có | nhãn nhóm |
| JTT (Liu 2021) | không | **có** | không | mẫu bị phân loại sai ở vòng 1 |
| DFR (Kirichenko 2023) | có | có | có | dữ liệu cân bằng để train lại layer cuối |
| INLP (Ravfogel 2020) / LEACE (Belrose 2023) | có | — | **có** | nhãn của concept cần xoá |
| Data balancing / reweighting | thường có | có | thường có | dữ liệu |
| CBP (Yan 2023, baseline của SFC) | không | không | **có** — phải soạn 20 keyword của task | kiến thức người dùng về task đích |
| **SHIFT** | **không** | **không** | **không** | **phán đoán của người khi đọc feature** |

Đây là bảng quan trọng nhất của phần §4. Nó cho thấy SHIFT không phải "tốt hơn một chút" mà **hoạt động trong một chế độ mà các phương pháp kia không định nghĩa được bài toán**: khi giới tính dự báo nhãn 100%, không có mẫu sai để upweight, không có nhóm để cân bằng, không có nhãn concept để chiếu bỏ.

---

## Bảng 4 — Kết quả SHIFT trên Bias in Bios (Bảng 2 của paper, trang 8)

Accuracy trên balanced set. Gender 50 = không còn dựa vào giới tính.

| Phương pháp | Pythia Prof ↑ | Pythia Gender ↓ | Pythia Worst ↑ | Gemma Prof ↑ | Gemma Gender ↓ | Gemma Worst ↑ |
|---|---|---|---|---|---|---|
| Original | 61.9 | 87.4 | 24.4 | 67.7 | 81.9 | 18.2 |
| CBP | 83.3 | 60.1 | 67.7 | 90.2 | 50.1 | 86.7 |
| Random ablation | 61.8 | 87.5 | 24.4 | 67.3 | 82.3 | 18.0 |
| SHIFT | 88.5 | 54.0 | 76.0 | 76.0 | 51.5 | 50.0 |
| **SHIFT + retrain** | **93.1** | 52.0 | **89.0** | **95.0** | 52.4 | **92.9** |
| Neuron skyline | 75.5 | 73.2 | 41.5 | 65.1 | 84.3 | 5.6 |
| Feature skyline | 88.5 | 54.3 | 62.9 | 80.8 | 53.7 | 56.7 |
| Oracle | 93.0 | 49.4 | 91.9 | 95.0 | 50.6 | 93.1 |

Bốn phép so sánh nên chỉ ra khi trình bày bảng này:
1. **SHIFT+retrain vs Oracle** (93.1 vs 93.0; 95.0 vs 95.0): gần bằng nhau, dù Oracle được xem nhãn thật của balanced set.
2. **Random vs Original** (61.8 vs 61.9): ablate bừa không có tác dụng gì → hiệu quả đến từ việc chọn đúng feature.
3. **SHIFT vs Feature skyline** ở worst-group Pythia (76.0 vs 62.9): con người chọn tốt hơn thuật toán được xem đáp án.
4. **Neuron skyline Gemma worst-group 5.6 < Original 18.2**: ablate neuron làm mọi thứ **tệ hơn** trạng thái ban đầu.

---

## Bảng 5 — Hai model, hai bộ SAE

| | Pythia-70M | Gemma-2-2B |
|---|---|---|
| Layer | 6 | 26 |
| SAE | nhóm tự train | Gemma Scope |
| Encoder | ReLU tuyến tính + L1 (λ=0.1) | JumpReLU |
| Bề rộng | `64 × d` | width 16384 (paper §2 ghi `8 × d`) |
| Submodule có SAE | embedding, attn, MLP, residual | attn, MLP, residual |
| Vị trí SAE attn | output của attention | input của out-projection |
| Vị trí SAE MLP | output của MLP | output của LayerNorm sau MLP |
| Dữ liệu train SAE | The Pile, ~2B token, 120k step | — (dùng bộ có sẵn) |
| Circuit case study across-RC | 86 node, faithfulness 0.21 | 223 node, faithfulness 0.21 |
| Circuit BiB | 67 feature, 55 bị loại | 46 feature, 43 bị loại |
| Layer cho classifier | ℓ = 4 (penultimate) | ℓ = 22 (chọn layer generalize tệ nhất) |
| Vướng mắc riêng | tỉ lệ feature chết cao (12–44% alive) | **BOS features** — kích hoạt trên token BOS, IE lớn, không diễn giải được → bị loại khỏi phân tích |

---

## Bảng 6 — Ba cách ablate trong paper

| | Công thức | Dùng khi | Dùng ở đâu | Rủi ro |
|---|---|---|---|---|
| Patching | `a ← a_patch` | có cặp tương phản | §3 | cần thiết kế cặp input khớp nhau |
| Zero-ablation | `a ← 0` | chỉ có một input | §4, §5 | đưa model ra ngoài phân phối dữ liệu |
| Mean-ablation | `a ←` mean theo vị trí token | đánh giá faithfulness | §3.2 | cần dữ liệu templatic để mean có nghĩa |

---

## Bảng 7 — Attribution patching vs Integrated gradients

| | Attribution patching (Eq. 3) | Integrated gradients (Eq. 4) |
|---|---|---|
| Công thức | `∇_a m|_{a_clean} · (a_patch − a_clean)` | `(1/N) Σ_α ∇_a m|_{α a_clean+(1−α) a_patch} · (a_patch − a_clean)` |
| Số bước | 1 điểm | N = 10 điểm cách đều |
| Chi phí | 2 forward + 1 backward, song song toàn bộ node | ×N, và không song song được giữa node phụ thuộc nhau |
| Chính xác ở đâu | phần lớn layer | mọi nơi |
| Sai ở đâu | **MLP layer 0 và residual stream layer đầu** — ước lượng thấp | — |
| SFC dùng | để so sánh | **cho mọi thí nghiệm chính** |

Nguồn bằng chứng: Figure 25, trang 36, dữ liệu từ 30 input của tập across-RC.

---

## Bảng 8 — SFC vs Attribution Graphs (Anthropic 03/2025)

| | Sparse Feature Circuits | Attribution Graphs |
|---|---|---|
| Thời điểm | 03/2024 (ICLR 2025 Oral) | 27/03/2025 |
| Model | Pythia-70M, Gemma-2-2B | Claude 3.5 Haiku |
| Đơn vị | SAE feature (một layer) | cross-layer transcoder feature (đọc 1 layer, ghi mọi layer sau) |
| Phần dư | error node có cạnh vào **và** cạnh ra | error node chỉ là node **đầu vào**, không có cạnh vào |
| Phạm vi | circuit trung bình trên một dataset | circuit cho **một prompt cụ thể** |
| Kiểm chứng | faithfulness/completeness + SHIFT | thí nghiệm nhiễu loạn theo sau mỗi giả thuyết |
| Attention | có SAE cho attention output | **không giải thích được QK-circuit**, attention pattern bị cố định |
| Ứng dụng downstream | SHIFT — gỡ spurious cue, đo bằng số | đọc hiểu cơ chế (thơ, tính nhẩm, đa ngôn ngữ, từ chối, ảo giác) |

---

## Bảng 9 — Ba đóng góp của paper, ứng với ba phần và ba loại dữ liệu đầu vào (Figure 1)

| | §3 | §4 | §5 |
|---|---|---|---|
| Dữ liệu vào | contrastive pairs | classification data | hành vi tự phát hiện bằng clustering |
| Ví dụ | "The boy near the teacher has" / "The boys near the teacher have" | "His research is in… → Professor" / "She worked in the OR… → Nurse" | Part 1, Part 2, Part 3, Part 4 |
| Ablation | patching | zero | zero |
| Gộp vị trí token | giữ nguyên vị trí (templatic) | cộng qua vị trí | cộng qua vị trí |
| Metric `m` | log-prob difference | `−log C(y|x)` | `−log P(y_i|x_i)` |
| Đầu ra | circuit + đánh giá faithfulness/completeness | classifier đã gỡ bias | hàng nghìn circuit, duyệt tại feature-circuits.xyz |
| Mức tự động | dữ liệu do người soạn | người chú thích feature | gần như hoàn toàn tự động |
