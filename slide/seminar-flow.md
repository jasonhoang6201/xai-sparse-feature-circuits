# Sparse Feature Circuits: flow trình bày seminar (10–15 phút)

Paper: Marks, Rager, Michaud, Belinkov, Bau, Mueller. *Sparse Feature Circuits: Discovering and Editing Interpretable Causal Graphs in Language Models*. ICLR 2025, arXiv 2403.19647.

Mỗi slide có 4 phần:
- **Trên slide**: chữ và công thức đưa lên slide.
- **Hình**: hình hoặc bảng lấy từ paper (số trang theo PDF `xai.pdf`).
- **Lời nói**: nội dung nói khi trình bày.
- **Nguồn**: chỗ trong paper chứa số liệu của slide.

Tổng: 16 slide, khoảng 14:30. Nếu thiếu thời gian thì rút slide 12 (case study) và slide 15 (unsupervised) xuống còn 1 câu mỗi slide.

| # | Slide | Thời lượng | Cộng dồn |
|---|---|---|---|
| 1 | Tiêu đề | 0:20 | 0:20 |
| 2 | Vấn đề và 2 thách thức | 0:50 | 1:10 |
| 3 | Circuit và công trình trước | 0:50 | 2:00 |
| 4 | Neuron polysemantic, superposition | 0:45 | 2:45 |
| 5 | SAE | 1:15 | 4:00 |
| 6 | Setup: SAE gắn vào model, node ε | 1:00 | 5:00 |
| 7 | Đo ảnh hưởng của một node: IE | 1:05 | 6:05 |
| 8 | Chi phí, AtP và IG | 1:15 | 7:20 |
| 9 | 4 bước tìm circuit | 1:00 | 8:20 |
| 10 | 4 task và 3 tiêu chí đánh giá | 1:00 | 9:20 |
| 11 | Kết quả trên agreement | 1:00 | 10:20 |
| 12 | Case study: agreement across RC | 0:50 | 11:10 |
| 13 | SHIFT: phương pháp và dữ liệu | 1:00 | 12:10 |
| 14 | SHIFT: kết quả | 1:00 | 13:10 |
| 15 | Unsupervised circuit discovery | 0:40 | 13:50 |
| 16 | Limitations và hướng sau paper | 0:50 | 14:40 |

---

## Phần I. Vấn đề (slide 1–4)

### Slide 1. Tiêu đề
**Trên slide**
- Sparse Feature Circuits: Discovering and Editing Interpretable Causal Graphs in Language Models
- Marks, Rager, Michaud, Belinkov, Bau, Mueller. ICLR 2025
- Tên người trình bày

**Lời nói**
"Paper này giải thích một hành vi của language model bằng một graph gồm các feature interpretable. Sau đó paper dùng graph đó để sửa model."

---

### Slide 2. Vấn đề và 2 thách thức
**Trên slide**
- Mục tiêu: giải thích **các hành vi không lường trước** của model, bằng các unit **nhỏ** và **interpretable**, mà người nghiên cứu **không phải đưa ra giả thuyết từ trước**.
- Thách thức 1: chọn unit để phân tích. Neuron thường không interpretable. Linear classifier thì cần giả thuyết có sẵn.
- Thách thức 2: tìm circuit trên số lượng đơn vị rất lớn (scalability).
- Cách paper giải quyết: (1) SAE (dictionary learning), (2) xấp xỉ tuyến tính (attribution patching, integrated gradients).

**Lời nói**
"Paper đặt ra một mục tiêu và hai thách thức. Mục tiêu là giải thích hành vi của model bằng các thành phần nhỏ mà con người hiểu được (interpretable), và không cần ai đoán trước cơ chế. Thách thức đầu tiên là chọn đơn vị phân tích: neuron thường không interpretable, còn linear classifier thì cần giả thuyết có sẵn. Thách thức thứ hai là có quá nhiều đơn vị, nên phải tìm được circuit với chi phí chấp nhận được. Phần còn lại của bài đi lần lượt qua hai thách thức này."

**Nguồn**: Intro p.1, đoạn 2–3.

---

### Slide 3. Circuit và công trình trước
**Trên slide**
- **Circuit**: graph con gồm node và edge, gây ra một hành vi cụ thể. Chỉ giữ các node có ảnh hưởng lớn tới output.
- Bảng 2 nhóm công trình trước:

| Nhóm | Ví dụ | Hạn chế |
|---|---|---|
| Coarse-grained unit: attention head, cả MLP | Induction heads (Olsson 2022); MLP trong factual recall (Meng 2022, Geva 2023) | Polysemantic (Elhage 2022): một head hoặc một MLP làm nhiều việc, khó đọc nghĩa, khó dùng cho ứng dụng |
| Fine-grained unit, cần supervision: linear classifier, concept vector | Kim 2018 (TCAV), Belinkov 2022, Geiger 2023, Zou 2023 (RepE) | Người nghiên cứu phải đưa ra giả thuyết và chuẩn bị data trước. Không dùng được cho hành vi không đoán trước được |

**Lời nói**
"Circuit là tập nhỏ các thành phần mà khi thay đổi giá trị của chúng thì hành vi của model thay đổi theo. Các công trình trước tìm circuit theo hai hướng. Hướng thứ nhất lấy đơn vị là cả một attention head hay cả một MLP, ví dụ induction heads của Olsson 2022 hay MLP trong factual recall của Meng 2022. Các đơn vị này polysemantic, tức một đơn vị tham gia nhiều việc khác nhau. Hướng thứ hai dùng đơn vị nhỏ hơn, là hướng tìm bằng linear classifier, ví dụ TCAV của Kim 2018 hay Representation Engineering của Zou 2023. Nhưng muốn train classifier thì phải biết trước mình tìm concept gì, nên hướng này không áp dụng được cho hành vi ta chưa đoán ra."

**Nguồn**: Intro p.1; Related work Section 6 p.10.

---

### Slide 4. Neuron polysemantic và superposition
**Trên slide**
- Neuron trong paper = một chiều (một tọa độ) của activation vector.
- Polysemantic: một neuron kích hoạt mạnh với nhiều loại input không liên quan nhau.
- Nguyên nhân (superposition): số concept model cần biểu diễn nhiều hơn số chiều. Mỗi concept nằm trên một hướng riêng, không trùng với trục của neuron nào.
- Hệ quả: đọc từng neuron không biết concept nào đang có mặt, nhưng đọc **hướng** thì biết được.

**Hình**: hình 2-D có 3 hướng concept trên 2 trục (lấy từ `polysemantic-diagram.html`, bước superposition).

**Lời nói**
"Lấy ví dụ 2 chiều nhưng có 3 concept. Mỗi concept được đặt trên một hướng nằm giữa hai trục, nên một trục nhận giá trị từ nhiều concept. Đó là lý do neuron polysemantic. Vậy muốn đọc concept thì phải tìm các hướng đó, và SAE làm việc này."

**Nguồn**: p.1 footnote 1; Section 2 p.2 (trích Elhage 2022). Paper không tự giải thích superposition, chỉ trích dẫn.

---

## Phần II. Phương pháp (slide 5–9)

### Slide 5. SAE (sparse autoencoder)
**Trên slide**
- Công thức (Eq. 1):
  `x = x̂ + ε(x) = Σᵢ fᵢ(x)·vᵢ + b + ε(x)`
  - `x` (activation vector tại một vị trí đọc của model)
  - `vᵢ` (feature: vector đơn vị chỉ hướng của một concept, là cột i của decoder)
  - `fᵢ(x)` (feature activation: hệ số ≥ 0, sparse, tính lại cho mỗi input)
  - `b` (bias)
  - `ε(x)` (SAE error: phần SAE chưa tái tạo được)
- Encoder: `f = ReLU(W_enc·x + b_enc)` (Pythia). Decoder: `x̂ = W_dec·f + b`.
- Loss = L2 (reconstruction `‖x − x̂‖²`) + L1 trên `f` (ép sparse).
- Đây là dictionary learning: tìm một tập hướng `vᵢ` sao cho mỗi `x` là tổng của **ít** hướng.
- Nghĩa của feature được gán **sau khi train**: đọc các đoạn văn làm `fᵢ` lớn nhất, và các token mà `vᵢ` làm tăng logit (Neuronpedia).

**Hình**: sơ đồ x → encoder → f (đa số = 0) → decoder → x̂, kèm ε = x − x̂.

**Lời nói**
"SAE là một mạng nhỏ gắn ngoài model, được train trên activation của model đã đóng băng. Encoder biến x thành vector f rất dài nhưng gần như toàn số 0. Decoder ghép lại x̂ bằng tổng các cột vᵢ, mỗi cột nhân với fᵢ. Mỗi cột vᵢ là một feature. Ta không đặt tên feature lúc train. Sau khi train xong, mới xem feature đó kích hoạt mạnh trên đoạn văn nào để gán nhãn. Phần SAE không tái tạo được gọi là ε."

**Nguồn**: Eq. 1 p.2; Section 2 p.3 (ReLU-linear encoder, L2 + L1); Section 4 p.8 (Neuronpedia).

---

### Slide 6. Setup: SAE gắn vào model, node ε
**Trên slide**
- 2 model:

| | Pythia-70M | Gemma-2-2B |
|---|---|---|
| Layer | layer 0–5 (App B.1.3) | 26 layer (App B.2) |
| SAE | tác giả tự train: embedding, và attn out, MLP out, residual của mỗi layer | Gemma Scope (Lieberum 2024): attn, MLP, residual của mỗi layer, không dùng SAE cho embedding |
| d_SAE | 64 × d | 8 × d (Section 2); App B.2 ghi width 16 384 |
| Encoder | ReLU | JumpReLU |

- **Node** = một `fᵢ` hoặc một `ε` tại một vị trí token cụ thể.
- Giữ `ε` làm node nên `x̂ + ε = x` đúng tuyệt đối. Do đó forward của LM **không đổi** khi gắn SAE vào.
- Nếu bỏ `ε`, layer sau nhận `x̂ ≠ x`, tức là ta đang giải thích một model khác.
- Ví dụ dùng xuyên suốt: `The boy near the teacher` → `has` / `have`.

**Hình**: Figure 2 p.4, nửa trái (submodule → SAE feature và SAE error).

**Lời nói**
"Paper gắn SAE tại ba vị trí trong mỗi layer: output attention, output MLP và residual stream. Ý quan trọng ở đây là paper không bỏ phần error ε mà giữ nó làm một node trong graph. Vì x̂ cộng ε bằng đúng x, các layer sau nhận đúng giá trị như khi chưa gắn SAE. Nhờ vậy, circuit tìm ra là circuit của chính model gốc. Ví dụ dùng xuyên suốt phần sau là câu The boy near the teacher, model phải chọn has hay have."

**Nguồn**: Section 2 p.2–3 (2 bộ SAE, xử lý ε); Section 3.1 p.4 ("nodes correspond to feature activations or SAE errors at particular token positions").

---

### Slide 7. Đo ảnh hưởng của một node: IE
**Trên slide**
- Metric `m` (một số đại diện cho hành vi): `m = log P(have) − log P(has)`.
- Contrastive pair: `x_clean = "The teacher"` và `x_patch = "The teachers"`. Hai câu chỉ khác số ít/số nhiều.
- Indirect effect (Eq. 2):
  `IE(m; a) = m(x_clean | do(a = a_patch)) − m(x_clean)`
  - `a` (giá trị của node)
  - `a_patch` (giá trị node đó khi chạy `x_patch`)
  - `do(·)` (ghi đè node trong lúc chạy `x_clean`, mọi thứ khác giữ nguyên)
- Khi chỉ có 1 input (SHIFT, unsupervised): zero-ablation, `IE = m(x | do(a = 0)) − m(x)`.
- `|IE|` lớn → node quan trọng với hành vi. Dấu cho biết node làm `m` tăng hay giảm.

**Lời nói**
"Muốn biết một feature có quan trọng không, ta chạy câu clean, lấy giá trị của feature đó từ câu patch rồi ghi đè vào, và xem m đổi bao nhiêu. Phần chênh lệch đó là indirect effect. Nếu không có câu patch thì đặt feature về 0, gọi là zero-ablation. Lưu ý IE chỉ cho biết feature quan trọng tới mức nào. Còn feature mang nghĩa gì thì gán nhãn như ở slide SAE."

**Nguồn**: Eq. 2 p.3; zero-ablation p.3 cuối Section 2.

---

### Slide 8. Chi phí, AtP và IG
**Trên slide**
- Tính Eq. 2 cho từng node: mỗi node cần một lần chạy riêng. Số node rất lớn nên paper viết Eq. 2 "cannot be done efficiently".
- **Attribution patching (AtP)**, Eq. 3: `IE ≈ ∇ₐm · (a_patch − a_clean)`
  - `∇ₐm` (gradient của m theo node a, tại a_clean)
  - 2 forward + 1 backward cho **mọi** node cùng lúc.
  - Nhược điểm: ước lượng IE thấp hơn thực tế ở MLP layer 0 và residual stream các layer đầu (App H, Fig. 25).
- **Integrated gradients (IG)**, Eq. 4: lấy trung bình gradient tại N = 10 điểm nằm giữa a_clean và a_patch.
  - Chính xác hơn rõ ở các vị trí AtP sai.
  - Tốn khoảng N lần. Không chạy song song được cho hai node mà node này nằm sau node kia, nên chi phí tăng theo N và theo độ sâu của model.
- Paper dùng IG cho các thí nghiệm. Code gốc (`circuit.py`) dùng `method="ig"` cho **toàn bộ node**. Edge dùng xấp xỉ tuyến tính một bước.

**Hình**: Figure 25 p.36 (approximate IE so với exact IE: AtP ở trên, IG ở dưới).

**Lời nói**
"Tính đúng IE thì mỗi node cần một lần chạy riêng, mà số node là mọi feature của mọi SAE tại mọi vị trí token. Attribution patching thay phép ghi đè bằng xấp xỉ bậc nhất: lấy gradient nhân với độ chênh lệch giá trị. Chỉ cần 2 forward và 1 backward là có IE xấp xỉ cho mọi node. Nhưng hình 25 cho thấy AtP ước lượng thấp ở MLP layer 0 và residual stream các layer đầu. Integrated gradients lấy trung bình gradient trên 10 điểm, nên sát IE thật hơn và tốn khoảng 10 lần. Tác giả dùng IG khi tính node. Edge thì paper dùng xấp xỉ tuyến tính một bước."

**Nguồn**: Eq. 3–4 p.3; Section 3.1 p.4; App H p.36; repo github.com/saprmarks/feature-circuits, `circuit.py`.

---

### Slide 9. 4 bước tìm circuit
**Trên slide** (theo Figure 2)
1. Cache activations và metric (forward `x_clean`, `x_patch`).
2. Backpropagate, lưu gradient `∇m` cho mỗi node.
3. Tính IE xấp xỉ cho mỗi node, lấy trung bình trên dataset, giữ node có `|IE| > T_N` (node threshold).
4. Tính IE cho edge giữa node upstream `u` và downstream `d` (Eq. 5, App A.1), giữ edge có `|IE| > T_E` (edge threshold).
- Gộp kết quả: data theo template thì giữ riêng từng vị trí token. Data không theo template (SHIFT, unsupervised) thì cộng các vị trí token rồi lấy trung bình theo ví dụ.

**Hình**: Figure 2 p.4 (đủ 4 bước).

**Lời nói**
"Ghép lại, ta có quy trình 4 bước. Chạy forward để lưu activation, chạy backward để lấy gradient, tính IE xấp xỉ rồi lọc node theo ngưỡng T_N. Sau đó làm tương tự cho edge. IE của edge đo ảnh hưởng của u lên m đi qua d, không tính đường đi qua các node khác. Kết quả là một graph gồm node và edge, chính là sparse feature circuit. Với data theo template, IE được tính riêng ở từng vị trí token. Với data không theo template như SHIFT và unsupervised, IE được cộng qua các vị trí token rồi lấy trung bình theo ví dụ."

**Nguồn**: Figure 2 p.4; Section 3.1 p.4; App A.1–A.2 p.18–19.

---

## Phần III. Thí nghiệm subject-verb agreement (slide 10–12)

### Slide 10. 4 task và 3 tiêu chí đánh giá
**Trên slide**
- 4 dataset subject-verb agreement, xây từ Finlayson et al. 2021 (Table 1):

| Task | Input clean | m |
|---|---|---|
| Simple | The parents | p(is) − p(are) |
| Within RC | The athlete that the managers | p(likes) − p(like) |
| Across RC | The athlete that the managers like | p(do) − p(does) |
| Across PP | The secretaries near the cars | p(has) − p(have) |

- 3 tiêu chí, so với neuron circuit (cùng phương pháp nhưng node là neuron, không có ε):
  - **Interpretability**: crowdworker chấm 0–100 (chỉ cho Pythia, App F).
  - **Faithfulness** `= (m(C) − m(∅)) / (m(M) − m(∅))`
    - `C` (circuit), `M` (cả model), `∅` (circuit rỗng)
    - Mọi node ngoài C bị **mean-ablate**: đặt về trung bình theo vị trí trên dataset. Lý tưởng = 1.
  - **Completeness**: faithfulness của phần bù `M \ C`. Lý tưởng = 0.
- Đánh giá trên test split, bỏ 1/3 đầu của circuit (các layer đầu xử lý token cụ thể, không trùng giữa train và test).

**Lời nói**
"Paper thử trên 4 biến thể của subject-verb agreement, khó dần: có relative clause hoặc prepositional phrase chen giữa chủ ngữ và động từ. Đánh giá theo 3 tiêu chí. Interpretability do crowdworker chấm điểm 0 tới 100, và chỉ làm cho Pythia. Faithfulness hỏi: nếu chỉ giữ circuit, còn mọi thứ khác đặt về giá trị trung bình, thì circuit giữ được bao nhiêu phần hành vi; tốt nhất là 1. Completeness hỏi ngược lại: bỏ circuit đi thì phần còn lại của model còn làm được bao nhiêu; tốt nhất là 0. Mọi kết quả được so với neuron circuit, tức là cùng phương pháp nhưng node là neuron và không có ε. Paper đo trên test split và bỏ qua 1/3 đầu của circuit, vì các layer đầu xử lý token cụ thể mà train và test không dùng chung."

**Nguồn**: Table 1 p.5; Section 3.2 p.5–6; footnote 2 p.6 (mean-ablation theo Wang et al. 2023).

---

### Slide 11. Kết quả trên agreement
**Trên slide**
- Faithfulness:
  - Pythia: khoảng **100 node** feature giải thích phần lớn hành vi. Neuron cần khoảng **1 500** để giải thích một nửa.
  - Gemma: khoảng **500 node** feature, so với khoảng **50 000** neuron.
  - Caveat: 100/500 node có tính error node, mà error node là vector nhiều chiều nên không so công bằng với neuron. Bỏ residual error node thì model hỏng nặng. Bỏ attn/MLP error node thì ảnh hưởng ít hơn.
- Completeness: ablate vài node feature là hành vi biến mất, kể cả khi giữ nguyên mọi ε. Neuron cần hàng trăm (Pythia) hoặc hàng nghìn (Gemma).
- Interpretability (Table 7, thang 0–100):

| | Random | Agreement (top 30) | BiB (top 30) |
|---|---|---|---|
| Neuron | 32.6 | 30.2 | 36.0 |
| SAE feature | 52.8 | 62.3 | 81.5 |

- Pythia và Gemma cho cùng một xu hướng.

**Hình**: Figure 3 p.5 (faithfulness và completeness theo số node).

**Lời nói**
"Ở Pythia, khoảng 100 node feature đã giải thích phần lớn hành vi, còn neuron cần khoảng 1 500 mới giải thích được một nửa. Ở Gemma là 500 so với 50 000. Con người cũng chấm feature dễ đọc hơn neuron. Chỗ cần lưu ý là các con số này có tính error node, và riêng residual error thì không bỏ được."

**Nguồn**: Section 3.2 p.6; Figure 3 p.5; Table 7 p.33.

---

### Slide 12. Case study: agreement across RC
**Trên slide**
- Circuit nhỏ cho dễ gán nhãn: Pythia **86 node**, Gemma **223 node**, faithfulness **0.21**.
- Hai đường xử lý trong cả 2 model:
  1. Feature phát hiện số của chủ ngữ → tăng xác suất dạng động từ khớp.
  2. Feature phát hiện ranh giới PP/RC mang thông tin số tới cuối mệnh đề.
- Gemma có thêm NP number tracker: giữ số của danh từ chính, kích hoạt tới hết noun phrase.
- Circuit across RC trùng nhiều với circuit across PP. Ít trùng với simple và within RC (khớp với Finlayson 2021).

**Hình**: Figure 4 p.7.

**Lời nói**
"Đọc circuit thì thấy được cách model làm task. Model phát hiện chủ ngữ là số ít hay số nhiều, phát hiện chỗ bắt đầu mệnh đề phụ, rồi mang thông tin số qua mệnh đề đó tới vị trí động từ. Hai model kích thước khác nhau nhưng dùng cùng một cách. Ngoài ra, circuit across RC trùng nhiều với circuit across PP, nhưng ít trùng với simple và within RC."

**Nguồn**: Section 3.3 p.6; Figure 4 p.7.

---

## Phần IV. Ứng dụng (slide 13–15)

### Slide 13. SHIFT: phương pháp và dữ liệu
**Trên slide**
- Vấn đề: classifier dựa vào spurious correlation. Các phương pháp trước cần **data có label riêng** cho tín hiệu đó.
- SHIFT (Sparse Human-Interpretable Feature Trimming), 4 bước:
  1. Tìm feature circuit cho accuracy của classifier (`m = −log C(y|x)`, zero-ablation).
  2. Con người đọc từng feature và đánh giá có liên quan tới task không.
  3. Zero-ablate các feature không liên quan → classifier C′.
  4. (Tùy chọn) Train lại linear head trên cùng data.
- Dữ liệu: Bias in Bios (De-Arteaga 2019), đoán nghề từ tiểu sử.
  - Ambiguous set (dùng để train): giáo sư nam và y tá nữ. Giới tính dự đoán đúng 100% nhãn.
  - Balanced set (dùng để test): đủ 4 nhóm nam/nữ × giáo sư/y tá.
- Circuit: Pythia 67 feature, người đánh giá chọn bỏ 55. Gemma 46 feature, bỏ 43.

**Lời nói**
"Đây là ứng dụng thứ nhất. Ta có một classifier đoán nghề, nhưng data train chỉ gồm giáo sư nam và y tá nữ, nên classifier có thể đoán nghề dựa vào giới tính. Không có data nào tách được hai tín hiệu này. SHIFT tìm circuit của classifier bằng zero-ablation với metric m = −log C(y|x), cho người đọc từng feature xem có liên quan tới task không, zero-ablate những feature không liên quan để được classifier mới C′, rồi nếu cần thì train lại riêng linear head trên cùng data. Người làm không cần data có nhãn mới."

**Nguồn**: Section 4 p.7–8.

---

### Slide 14. SHIFT: kết quả
**Trên slide**
- Table 2, accuracy trên balanced set:

| Pythia-70M | Profession ↑ | Gender ↓ | Worst group ↑ |
|---|---|---|---|
| Original | 61.9 | 87.4 | 24.4 |
| CBP | 83.3 | 60.1 | 67.7 |
| Random ablation | 61.8 | 87.5 | 24.4 |
| SHIFT | 88.5 | 54.0 | 76.0 |
| SHIFT + retrain | 93.1 | 52.0 | 89.0 |
| Neuron skyline | 75.5 | 73.2 | 41.5 |
| Oracle | 93.0 | 49.4 | 91.9 |

- Gemma: SHIFT + retrain đạt 95.0 / 52.4 / 92.9 (oracle 95.0 / 50.6 / 93.1). SHIFT không retrain làm giảm profession (76.0), retrain khôi phục lại.
- Gender accuracy (Pythia) giảm từ 87.4 xuống 52.0, gần mức oracle 49.4 → classifier gần như không còn dùng giới tính.
- Bỏ phiên bản SHIFT dùng neuron, vì gần như không neuron nào đọc được nghĩa.
- Neuron skyline (chọn neuron bằng balanced set) vẫn thua, vì neuron liên quan giới tính cũng mang thông tin nghề.

**Lời nói**
"Với Pythia, worst-group accuracy tăng từ 24.4 lên 76.0 chỉ nhờ bỏ feature, và lên 89.0 sau khi train lại head, gần bằng oracle train trên data cân bằng. Ablate ngẫu nhiên cùng số feature thì không đổi gì. Làm với neuron thì không được, vì mỗi neuron trộn cả thông tin giới tính lẫn thông tin nghề. Neuron skyline vẫn thua, vì neuron liên quan giới tính cũng mang thông tin về nghề."

**Nguồn**: Table 2 p.8; Section 4 Results p.9.

---

### Slide 15. Unsupervised circuit discovery
**Trên slide**
- Hạn chế của công trình trước: hành vi do người chọn và dataset do người thu thập (Wang 2023 IOI, Conmy 2023 ACDC, Hanna 2023).
- Pipeline, chạy trên Pythia-70M và The Pile:
  1. Phát hiện hành vi bằng clustering (theo Michaud 2023): mỗi mẫu `(context, token tiếp theo)` → một vector (gradient hoặc SAE activation) → cluster.
  2. Mỗi cluster: tìm feature circuit bằng zero-ablation, `m = −log P(y|x)`.
- Kết quả: hàng nghìn circuit, xem tại feature-circuits.xyz (chưa gán nhãn).
  - Cluster 382: dãy tăng dần (narrow induction + succession).
  - Cluster 475: "to" làm infinitive object (hai cơ chế riêng).
- Đánh giá cluster và circuit vẫn là bài toán mở.

**Hình**: Figure 5 p.9.

**Lời nói**
"Ứng dụng thứ hai bỏ bước con người chọn hành vi. Máy tự gom các đoạn văn mà model xử lý giống nhau thành cluster, rồi tìm circuit cho từng cluster. Ví dụ cluster 382 là đoán số tiếp theo trong dãy. Circuit cho thấy model dùng hai cơ chế cho cluster này. Con người vẫn phải gán nhãn feature, và paper chưa có cách đánh giá định lượng cho phần này. Toàn bộ các circuit, chưa gán nhãn, có thể xem tại feature-circuits.xyz."

**Nguồn**: Section 5 p.9–10; App G.

---

## Phần V. Hạn chế (slide 16)

### Slide 16. Limitations và hướng sau paper
**Trên slide**
- Limitations do paper tự nêu (Section 8):
  1. Phụ thuộc vào SAE. Train SAE tốn chi phí lớn (một lần).
  2. Phần SAE không bắt được (ε) vẫn không đọc được nghĩa.
  3. Đánh giá phần lớn là định tính. Khó đánh giá dictionary và circuit khi không có task downstream.
  4. Gán nhãn feature là định tính, có thể khác nhau giữa người gán và giữa các task.
- Hướng sau paper:
  - Tự động hóa SHIFT: thay người gán nhãn bằng LLM, thành metric SCR/TPP để đánh giá SAE: *Evaluating Sparse Autoencoders on Targeted Concept Erasure Tasks* (Karvonen, Rager, Marks, Nanda; NeurIPS 2024 workshop ATTRIB).
  - Thay SAE bằng transcoder: *Transcoders Find Interpretable LLM Feature Circuits* (Dunefsky, Chlenski, Nanda; NeurIPS 2024). Cross-layer transcoder và attribution graph: *Circuit Tracing* (Ameisen et al., Anthropic; Transformer Circuits Thread 2025).
  - Phê bình SAE: feature absorption, feature tắt ở chỗ đáng lẽ phải kích hoạt (Chanin et al.; NeurIPS 2025). Classifier dùng SAE feature không vượt baseline đơn giản (Kantamneni et al.; ICML 2025).

**Lời nói**
"Paper tự nêu bốn hạn chế. Ba trong số đó xoay quanh SAE và việc gán nhãn thủ công. Các công trình sau đi tiếp đúng những chỗ này. Chính nhóm tác giả tự động hóa SHIFT bằng LLM. Một số nhóm thay SAE bằng transcoder. Cũng có các bài chỉ ra rằng feature của SAE không phải lúc nào cũng kích hoạt đúng chỗ."

**Nguồn**: Section 8 p.10. Các link đã kiểm:
- https://arxiv.org/abs/2411.18895 (SCR/TPP)
- https://arxiv.org/abs/2406.11944 (transcoders)
- https://transformer-circuits.pub/2025/attribution-graphs/methods.html (Circuit Tracing)
- https://arxiv.org/abs/2409.14507 (feature absorption)
- https://arxiv.org/abs/2502.16681 (sparse probing)

---

## Tài liệu tham khảo cho slide 3 (đã kiểm)
- Olsson et al. 2022, *In-context Learning and Induction Heads* (Transformer Circuits Thread). https://transformer-circuits.pub/2022/in-context-learning-and-induction-heads/
- Meng et al. 2022, *Locating and Editing Factual Associations in GPT* (NeurIPS). https://arxiv.org/abs/2202.05262
- Kim et al. 2018, *TCAV* (ICML). https://arxiv.org/abs/1711.11279
- Zou et al. 2023, *Representation Engineering* (arXiv preprint). https://arxiv.org/abs/2310.01405
- Finlayson et al. 2021, *Causal Analysis of Syntactic Agreement Mechanisms* (ACL). https://arxiv.org/abs/2106.06087
- Michaud et al. 2023, *The Quantization Model of Neural Scaling* (NeurIPS). https://arxiv.org/abs/2303.13506
- Nanda 2022, *Attribution Patching*. https://neelnanda.io/attribution-patching
- Sundararajan et al. 2017, *Axiomatic Attribution for Deep Networks* (ICML). https://arxiv.org/abs/1703.01365
- De-Arteaga et al. 2019, *Bias in Bios* (FAT*). https://arxiv.org/abs/1901.09451
- Wang et al. 2023, *IOI circuit* (ICLR). https://arxiv.org/abs/2211.00593
- Conmy et al. 2023, *ACDC*. https://arxiv.org/abs/2304.14997
