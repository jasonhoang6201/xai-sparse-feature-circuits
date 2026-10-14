# Paper gốc — Sparse Feature Circuits

> Nguồn: `../xai.pdf` (36 trang). Bản v3 trên arXiv ghi ngày 27/03/2025; bản v1 là 28/03/2024.

## 1. Thông tin thư mục

| Trường | Giá trị |
|---|---|
| Tên | Sparse Feature Circuits: Discovering and Editing Interpretable Causal Graphs in Language Models |
| Tác giả | Samuel Marks, Can Rager, Eric J. Michaud, Yonatan Belinkov, David Bau, Aaron Mueller |
| Đơn vị | Northeastern University (Marks, Bau, Mueller), Independent (Rager), MIT (Michaud), Technion – IIT (Belinkov) |
| Hội nghị | ICLR 2025, nhận dạng **Oral** |
| arXiv | 2403.19647 |
| Code | github.com/saprmarks/feature-circuits |
| Demo cluster + circuit | feature-circuits.xyz |
| Trang dự án | features.baulab.info |
| Duyệt feature | neuronpedia.org/p70d-sm (SAE Pythia-70M của nhóm) |
| Liên hệ | s.marks@northeastern.edu, aa.mueller@northeastern.edu |

Ghi chú về tên file: `xai.pdf` ở đây là viết tắt của *explainable AI*, không liên quan tới công ty xAI.

## 2. Problem — bài toán paper đặt ra

Paper mở đầu bằng một phát biểu về mục tiêu của interpretability: giải thích được **nhiều** hành vi **không lường trước** của mạng nơ-ron, ở quy mô lớn. Hai dòng nghiên cứu trước đó đều không đạt cả hai điều kiện:

**Dòng 1 — circuit trên component thô (coarse-grained).**
Các công trình như induction heads (Olsson et al., 2022), factual recall qua MLP (Meng et al., 2022; Geva et al., 2023) quy hành vi về attention head hoặc MLP module. Vấn đề: những component này **polysemantic** (Elhage et al., 2022) — một head/neuron tham gia nhiều chức năng khác nhau, nên nhãn gán cho nó không dùng được cho tác vụ downstream. Bạn không thể "tắt head số 7" để bỏ một thiên kiến, vì head đó còn làm việc khác.

**Dòng 2 — đơn vị mịn nhưng cần giả thuyết trước (supervised).**
TCAV (Kim et al., 2018), probing (Belinkov, 2022), causal abstraction (Geiger et al., 2023), representation engineering (Zou et al., 2023) khớp nội bộ model vào một giả thuyết do người nghiên cứu nêu, trên dữ liệu do người nghiên cứu soạn. Vấn đề: chỉ chạy được khi đã biết trước cần tìm gì. Không áp dụng được cho các hành vi mà người nghiên cứu chưa hình dung nổi.

Từ đó paper nêu hai thách thức kỹ thuật phải giải quyết cùng lúc:

1. **Chọn đơn vị phân tích mịn mà vẫn diễn giải được.** Neuron thì hiếm khi diễn giải được; hướng tìm bằng probing thì cần giả thuyết có sẵn.
2. **Chịu được chi phí tìm kiếm.** Số đơn vị mịn rất lớn (hàng chục nghìn feature × mỗi submodule × mỗi vị trí token), nên không thể patch từng cái một.

Paper cũng định nghĩa "neuron" theo nghĩa rộng: một hướng trùng trục cơ sở (basis-aligned direction) trong không gian ẩn, không nhất thiết đứng sau một phi tuyến.

## 3. Solution — hai thành phần

### 3.1 Đơn vị phân tích: SAE feature (giải quyết thách thức 1)

Sparse autoencoder phân tích một activation `x ∈ R^d` thành:

```
x = x̂ + ε(x) = Σ_{i=1..d_SAE} f_i(x)·v_i + b + ε(x)      (Eq. 1)
```

- `v_i`: feature vector, là vector đơn vị (cột của `W_D`)
- `f_i(x) ≥ 0`: activation của feature, thưa
- `b`: bias
- `ε(x) ∈ R^d`: **SAE error**, phần SAE không tái tạo được

Quyết định thiết kế đáng chú ý nhất: **giữ lại `ε` và đưa nó vào circuit như một loại node riêng**, thay vì vứt đi. Hệ quả: circuit phân tách hành vi thành phần do feature diễn giải được gây ra, cộng với phần mà SAE hiện chưa nắm bắt. Nếu error node có indirect effect lớn, đó là bằng chứng định lượng rằng SAE còn thiếu.

Hai bộ SAE dùng trong paper:

| | Pythia-70M | Gemma-2-2B |
|---|---|---|
| Nguồn | nhóm tự train | Gemma Scope (Lieberum et al., 2024) |
| Encoder | ReLU tuyến tính | JumpReLU |
| Bề rộng | `d_SAE = 64 × d` | paper nêu `8 × d`; App. B.2 nói dùng SAE width 16384 |
| Submodule có SAE | embedding, attn out, MLP out, residual stream | attn, MLP, residual (không có embedding — SAE embedding của Gemma Scope chỉ 4000 chiều nên bị loại) |
| Vị trí train SAE attn/MLP | output | attn: input của out-projection; MLP: output của LayerNorm sau MLP |
| Chọn bản nào | — | bản có L0 gần 100 nhất, trùng lựa chọn của Neuronpedia |

Chi tiết train SAE Pythia (App. B.1): loss `‖x̂ − x‖² + λ‖f‖₁` với λ = 0.1, lr 1e-4, Adam biến thể giữ cột `W_D` là vector đơn vị, dữ liệu The Pile (128 token đầu mỗi document), buffer 10 000 context, batch 2^14, 120 000 step ≈ 2 tỉ token. Feature chết (dead feature) được reinit mỗi 25 000 step nếu không kích hoạt trong 12 500 step trước đó, kèm warmup tuyến tính 1000 step sau mỗi lần resample.

Chất lượng SAE Pythia (App. B.1.3, Bảng 3–6): embedding đạt 96% variance explained, L0 = 3, 36% feature còn sống, CE diff 0.17, 98% CE recovered. Các SAE attn đạt 83–92% variance explained với L0 khoảng 127–215 và chỉ 12–20% feature còn sống. Các SAE MLP đạt 85–99% variance explained, L0 từ 5 đến 160, 25–44% feature còn sống. Tức là **tỉ lệ feature chết rất cao** — con số này đáng nêu trong seminar vì nó là một giới hạn thật của bộ SAE dùng trong paper.

### 3.2 Chi phí tìm kiếm: xấp xỉ tuyến tính indirect effect (giải quyết thách thức 2)

Định nghĩa gốc theo causal mediation (Pearl, 2001):

```
IE(m; a; x_clean, x_patch) = m(x_clean | do(a = a_patch)) − m(x_clean)      (Eq. 2)
```

Ví dụ trong paper: `x_clean` = "The teacher", `x_patch` = "The teachers", metric `m(x) = log P("are"|x) − log P("is"|x)`.

Tính trực tiếp Eq. 2 cho từng node thì cần một forward pass cho mỗi node. Paper thay bằng hai xấp xỉ:

**Attribution patching** (khai triển Taylor bậc nhất; Nanda 2022; Syed et al. 2023; Kramár et al. 2024):
```
ÎE_atp = ∇_a m |_{a=a_clean} · (a_patch − a_clean)      (Eq. 3)
```
Chi phí: 2 forward + 1 backward pass cho **toàn bộ** node song song.

**Integrated gradients** (Sundararajan et al., 2017; Hanna et al., 2024):
```
ÎE_ig = (1/N) Σ_α ∇_a m |_{α·a_clean + (1−α)·a_patch} · (a_patch − a_clean)      (Eq. 4)
```
với N = 10 giá trị α cách đều. Chi phí tăng tuyến tính theo N và theo độ sâu nối tiếp của computation graph.

Kết quả đo chất lượng xấp xỉ (App. H, Figure 25, trang 36): attribution patching bám khá sát IE thật, **trừ MLP layer 0 và residual stream các layer đầu**, ở đó nó ước lượng thấp. Integrated gradients cải thiện rõ ở đúng những chỗ đó. Vì vậy các thí nghiệm chính dùng `ÎE_ig`.

Khi chỉ có một input duy nhất (§4 và §5), paper chuyển sang **zero-ablation**: `IE(m; a; x) = m(x | do(a = 0)) − m(x)`, tức thay `a_patch` bằng 0 trong Eq. 3 và 4.

### 3.3 Quy trình 4 bước (Figure 2, trang 4)

1. **Cache activation và metric.** Chạy model trên `x_clean` và `x_patch`, lưu `f_i` và `ε` tại mọi submodule có SAE.
2. **Backpropagate, lưu gradient** `∇_{f_i} m`, `∇_{ε} m`.
3. **Tính effect cho node, lọc theo ngưỡng `T_N`.** Lấy trung bình `ÎE(m; a; x)` trên `x ~ D`, giữ node có `|ÎE| > T_N`.
4. **Tính effect cho edge, lọc theo ngưỡng `T_E`.**

Trọng số edge (App. A.1): với edge từ node upstream `u` tới node downstream `d`, gọi `M` là tập node trung gian giữa chúng, trọng số là hiệu ứng lên `m` khi can thiệp
```
d = d(x_clean | do(u = u_patch, m = m_clean ∀m ∈ M))
```
tức hiệu ứng gián tiếp của `u` lên `m` **chỉ qua đường trực tiếp tới `d`**, cắt bỏ mọi đường vòng qua node trung gian. Xấp xỉ tuyến tính:
```
ÎE(m; e) = ∇_d m|_{d_clean} · ∇_{u, stop(M)} d |_{u_clean} · (u_patch − u_clean)      (Eq. 5)
```

**Gộp effect qua vị trí token và example** (App. A.2, Figure 6 trang 19):
- Dữ liệu templatic (§3.2, §3.3 — các câu cùng khuôn, token cùng vị trí cùng vai trò): giữ nguyên vị trí, cùng một feature ở hai vị trí khác nhau được coi là hai node khác nhau; lấy mean qua example.
- Dữ liệu không templatic (§4, §5): **cộng** effect qua các vị trí trước, rồi lấy mean qua example. Mỗi feature xuất hiện một lần trong circuit.

### 3.4 Ba thủ thuật kỹ thuật (App. A.3) — phần dễ bị bỏ qua nhưng quyết định method chạy được

**(a) Can thiệp forward pass để autograd nhìn thấy `f_i` và `ε`.** Sau khi tính `f_i(x)` và `ε(x)`, gán lại
```
x ← Σ_i f_i(x)·v_i + b + ε(x)      (Eq. 6)
```
Về số học `x` không đổi, nhưng sau phép gán, computation graph của PyTorch chứa `f_i` và `ε` như biến, nên backward pass tự động sinh gradient cho chúng.

**(b) Stop-gradient trên `ε`.** Nếu tính `ε(x) = x − x̂` một cách trực tiếp thì
`∇_{f_i} m = ∇_{x_d} m · ∇_{f_i}(x̂ + x_u − x̂) = 0` — gradient triệt tiêu hoàn toàn. Khắc phục bằng `x_d = x̂ + stopgrad(x_u − x̂)`.

**(c) Pass-through gradient.** Stop-gradient ở (b) lại chặn gradient chảy ngược lên các node phía trên. Paper can thiệp backward pass, đặt `∇_{x_u} m ← ∇_{x_d} m` để khôi phục gradient chính xác.

**(d) Jacobian-vector product.** Khi `d` là SAE error, tính Eq. 5 theo cách ngây thơ tốn `O(d_model)` backward pass. Dùng đẳng thức `A∇_x y = ∇_x(Ay)` với `A` là hằng số 1×n, chỉ cần 2 backward pass: tính `∇_d m|_{d_clean}` trước, detach nó, rồi tính `∇_u(∇_d m|_{d_clean} · d)`.

## 4. Thí nghiệm 1 (§3) — subject-verb agreement

### Dữ liệu
Bốn biến thể, phỏng theo Finlayson et al. (2021), mỗi cặp chỉ khác nhau ở số ngữ pháp của chủ ngữ (Bảng 1, trang 5):

| Cấu trúc | Ví dụ input | Metric output |
|---|---|---|
| Simple | The parents | p(is) − p(are) |
| Within RC | The athlete that the managers | p(likes) − p(like) |
| Across RC | The athlete that the managers like | p(do) − p(does) |
| Across PP | The secretaries near the cars | p(has) − p(have) |

### Ba tiêu chí đánh giá

**Interpretability.** Crowdworker chấm 0–100 (App. F, Bảng 7):

| Loại đơn vị | Điểm trung bình |
|---|---|
| Dense (neuron) ngẫu nhiên | 32.6 |
| Dense, top-30 theo ÎE trên agreement | 30.2 |
| Dense, top-30 theo ÎE trên Bias in Bios | 36.0 |
| Sparse (SAE feature) ngẫu nhiên | 52.8 |
| Sparse, top-30 trên agreement | 62.3 |
| Sparse, top-30 trên Bias in Bios | **81.5** |

Đọc bảng này: feature thưa hơn neuron khoảng 20 điểm khi lấy ngẫu nhiên, và feature **nằm trong circuit** còn cao hơn feature ngẫu nhiên. Với neuron thì điều ngược lại xảy ra — neuron có ÎE cao trên agreement (30.2) còn khó hiểu hơn neuron ngẫu nhiên (32.6).

**Faithfulness.** Với circuit `C`, `m(C)` là giá trị trung bình của metric khi mean-ablate mọi node ngoài `C` (theo Wang et al. 2023, ablate về giá trị trung bình **theo từng vị trí token**):
```
faithfulness = (m(C) − m(∅)) / (m(M) − m(∅))
```
`∅` = circuit rỗng, `M` = full model. Lý tưởng = 1. Lưu ý phương pháp luận: paper **bỏ qua 1/3 circuit ở các layer đầu** khi đánh giá, vì các component layer đầu xử lý token cụ thể mà train split và test split không dùng chung token.

**Completeness.** Đo bằng faithfulness của phần bù `M \ C`. Lý tưởng = 0.

### Kết quả (Figure 3, trang 5)

| | Feature circuit | Neuron circuit |
|---|---|---|
| Pythia-70M: số node để giải thích phần lớn hành vi | ~100 | ~1 500 |
| Gemma-2-2B: số node để giải thích phần lớn hành vi | ~500 | ~50 000 |

Tức là chênh khoảng 15× ở Pythia và 100× ở Gemma.

Về completeness: chỉ cần ablate vài node trong feature circuit là xoá sạch khả năng làm task, **kể cả khi giữ nguyên toàn bộ SAE error node**. Với neuron cần hàng trăm (Pythia) đến hàng nghìn (Gemma).

Paper tự nêu một cảnh báo so sánh: SAE error node là vector nhiều chiều và thô, không so trực tiếp với neuron được. Vì thế Figure 3 vẽ thêm hai đường: circuit bỏ hết error node, và circuit bỏ error node của attn/MLP. Kết quả: **bỏ residual stream error node phá hỏng model nghiêm trọng**, bỏ error node attn/MLP thì ít ảnh hưởng hơn.

### Case study (§3.3, Figure 4 trang 7)

Chọn ngưỡng sao cho circuit đủ nhỏ để chú thích tay mà vẫn có faithfulness > 0.2:

| Model | Số node | Faithfulness |
|---|---|---|
| Pythia-70M | 86 | 0.21 |
| Gemma-2-2B | 223 | 0.21 |

Thuật toán đọc ra từ circuit — cả hai model chọn dạng động từ qua **hai đường song song**:
1. Feature phát hiện số của chủ ngữ chính → trực tiếp thúc đẩy dạng động từ khớp.
2. Cùng xuất phát như trên, nhưng chuyển thông tin số tới cuối mệnh đề quan hệ bằng **PP/RC boundary detector** (feature phát hiện ranh giới cụm giới từ / mệnh đề quan hệ).

Riêng Gemma-2-2B có thêm loại feature thứ ba: **NP number tracker** — phát hiện số của danh từ trung tâm của một noun phrase, và **giữ kích hoạt trên mọi token cho tới hết NP đó**, mạnh nhất ở token cuối NP. Figure 15 (trang 29) minh hoạ feature `resid 12/13561`: kích hoạt trên token của NP có danh từ trung tâm số nhiều, nhưng **không** kích hoạt trên danh từ số ít đóng vai trò distractor bên trong NP số nhiều đó.

Overlap giữa các circuit: across-RC và across-PP chồng nhau nhiều (hai cấu trúc cú pháp khác nhau nhưng model xử lý gần như đồng nhất); simple và within-RC thì ít chồng hơn — khớp với Finlayson et al. (2021).

Ngưỡng cụ thể của từng hình circuit đầy đủ (App. C.1): ví dụ `T_N = 0.5, T_E = 0.05` cho Gemma simple agreement; `T_N = 0.2, T_E = 0.02` cho Pythia within-RC.

## 5. Thí nghiệm 2 (§4) — SHIFT

### Vấn đề cụ thể
Gần như mọi phương pháp xử lý spurious correlation trước đó đều cần **disambiguating labeled data**: dữ liệu trong đó tín hiệu không mong muốn kém dự báo nhãn hơn tín hiệu mong muốn. Paper nêu ba lý do giả định đó có thể sai:
- Input của các class có thể đến từ các nguồn dữ liệu khác nhau (Zech et al., 2018 — model phát hiện viêm phổi học theo đặc điểm máy chụp X-quang của từng bệnh viện).
- Trong RLHF với easy-to-hard domain shift (Burns et al., 2023; Hase et al., 2024), "được người giám sát chấp thuận" và "hành vi đúng mong muốn" dự báo reward label ngang nhau (Ngo et al., 2024; Casper et al., 2023).
- Lý do nguyên tắc: vấn đề của tín hiệu không mong muốn nằm ở chỗ nó **không mong muốn**, chứ không phải ở chỗ nó kém dự báo.

### Phương pháp SHIFT — Sparse Human-interpretable Feature Trimming

Cho dữ liệu có nhãn `D = {(x_i, y_i)}`, classifier `C` dựa trên LM, và SAE cho các component của `C`:

1. Tính feature circuit giải thích độ chính xác của `C` trên `(x, y) ~ D`, metric `m = −log C(y|x)`.
2. Người xem từng feature trong circuit, đánh giá có liên quan tới task hay không.
3. Ablate các feature bị đánh giá là không liên quan → được `C'`.
4. (Tuỳ chọn) Fine-tune lại `C'` trên chính `D`.

Bước 3 gỡ phụ thuộc vào tín hiệu không mong muốn nhưng có thể làm hỏng luôn hiệu năng cho tín hiệu mong muốn; bước 4 khôi phục phần đó.

### Thiết lập thí nghiệm
Dataset Bias in Bios (De-Arteaga et al., 2019): tiểu sử nghề nghiệp, nhãn đích là nghề, nhãn spurious là giới tính. Paper lấy hai tập con:

- **Ambiguous set**: chỉ gồm tiểu sử của **nam giáo sư** (nhãn 0) và **nữ y tá** (nhãn 1). Giới tính dự báo nhãn đúng **100%** — trường hợp xấu nhất.
- **Balanced set**: số lượng bằng nhau của nam giáo sư, nam y tá, nữ giáo sư, nữ y tá.

Bài toán: **chỉ được dùng ambiguous set**, tạo ra một classifier nghề nghiệp chạy đúng trên balanced set.

Classifier được tạo bằng cách mean-pool residual stream ở layer ℓ trên các token không phải padding, rồi train linear head bằng logistic regression (AdamW, lr 0.01, 1 epoch). Chọn layer: Pythia lấy ℓ = 4 (penultimate) theo accuracy trên test split của ambiguous set. Gemma có nhiều layer hiệu năng ngang nhau, nên **chỉ cho riêng lựa chọn này** paper dùng balanced set để chọn ℓ = 22 — layer mà baseline probe generalize **tệ nhất**, để tạo dư địa cải thiện lớn nhất. Paper nhấn mạnh không bao giờ tune hyperparameter theo hiệu năng SHIFT trên balanced set.

Circuit thu được và số feature bị loại:

| Model | Số feature trong circuit | Số bị đánh giá không liên quan |
|---|---|---|
| Pythia-70M | 67 | 55 |
| Gemma-2-2B | 46 | 43 |

Việc diễn giải feature dùng giao diện Neuronpedia (Lin & Bloom, 2023): xem các đoạn văn bản kích hoạt feature mạnh nhất trên một corpus lớn, cùng với direct effect của feature lên output logit. Ví dụ feature bị loại: feature thúc đẩy ngôn ngữ gắn với nữ giới trong tiểu sử của phụ nữ (Figure 19, trang 31, `attn 3/22029`). Bước này dùng thêm dữ liệu **không nhãn**, nhưng không dùng thêm bất kỳ dữ liệu có nhãn nào.

Với Gemma Scope còn một vướng mắc riêng: **BOS features** — feature chỉ kích hoạt trên token BOS, top logit không mang thông tin, nhưng indirect effect lên logit lại lớn. Paper loại chúng khỏi việc chú thích, khỏi SHIFT (không ablate), và khỏi feature skyline.

### Kết quả (Bảng 2, trang 8)

Accuracy đo trên balanced set. ↑ = cao hơn tốt hơn; với Gender ↓ = càng gần 50 (đoán ngẫu nhiên) càng tốt.

| Phương pháp | Pythia ↑Profession | Pythia ↓Gender | Pythia ↑Worst group | Gemma ↑Profession | Gemma ↓Gender | Gemma ↑Worst group |
|---|---|---|---|---|---|---|
| Original | 61.9 | 87.4 | 24.4 | 67.7 | 81.9 | 18.2 |
| CBP | 83.3 | 60.1 | 67.7 | 90.2 | 50.1 | 86.7 |
| Random ablation | 61.8 | 87.5 | 24.4 | 67.3 | 82.3 | 18.0 |
| **SHIFT** | 88.5 | 54.0 | 76.0 | 76.0 | 51.5 | 50.0 |
| **SHIFT + retrain** | **93.1** | **52.0** | **89.0** | **95.0** | 52.4 | **92.9** |
| Neuron skyline | 75.5 | 73.2 | 41.5 | 65.1 | 84.3 | 5.6 |
| Feature skyline | 88.5 | 54.3 | 62.9 | 80.8 | 53.7 | 56.7 |
| Oracle | 93.0 | 49.4 | 91.9 | 95.0 | 50.6 | 93.1 |

"Worst group accuracy" = accuracy nghề thấp nhất trong bốn nhóm (nam giáo sư, nam y tá, nữ giáo sư, nữ y tá).

Các điểm rút ra:
- **SHIFT + retrain gần bằng Oracle** — 93.1 vs 93.0 (Pythia), 95.0 vs 95.0 (Gemma) ở profession accuracy; 89.0 vs 91.9 và 92.9 vs 93.1 ở worst-group. Oracle được train trên nhãn thật của balanced set, tức là SHIFT đạt gần mức đó mà không dùng nhãn nào ngoài ambiguous set.
- Với Gemma, ablation làm hỏng hiệu năng (profession tụt từ 67.7 xuống 76.0 vẫn tăng nhưng worst-group chỉ 50.0); retrain khôi phục lên 95.0/92.9 **mà không đưa bias quay lại** (gender 52.4).
- **Random ablation gần như không đổi gì** (61.8 vs 61.9) — chứng minh hiệu quả đến từ việc chọn đúng feature, không phải từ việc ablate nói chung.
- **SHIFT ≥ feature skyline** ở Pythia (76.0 vs 62.9 worst-group). Feature skyline được phép dùng balanced set để chọn feature; SHIFT chỉ dùng phán đoán của người. Tức con người chọn tốt bằng hoặc hơn thuật toán có quyền xem đáp án.
- **SHIFT với neuron bị bỏ hẳn.** Paper thử nhưng gần như không neuron nào diễn giải được nên không có cơ sở để quyết định nên ablate cái nào (App. D.2). Thay bằng neuron skyline — vốn có lợi thế không công bằng là được xem balanced set — mà vẫn thua: Gemma worst-group 5.6, thấp hơn cả Original 18.2. Lý do paper đưa ra: neuron dính líu nhiều nhất vào phân loại giới tính **cũng chính là** neuron hữu ích cho phân loại nghề.

### Baseline CBP (App. E.2)
Concept Bottleneck Probing, phỏng theo Yan et al. (2023) cho model ảnh/văn bản đa phương thức. Quy trình: (1) chọn N = 20 keyword liên quan task (nurse, healthcare, hospital, patient, medical, clinic, triage, medication, emergency, surgery, professor, academia, research, university, tenure, faculty, dissertation, sabbatical, publication, grant); (2) lấy biểu diễn layer penultimate ở token cuối mỗi keyword, trừ đi vector trung bình; (3) biểu diễn input `x` thành `z ∈ R^N` gồm cosine similarity với từng concept vector; (4) train linear probe trên `z`.

CBP mạnh trên Gemma (90.2 profession, 86.7 worst-group) nhưng vẫn thua SHIFT + retrain, và quan trọng hơn: CBP **cần biết trước task là gì** để soạn 20 keyword. SHIFT không cần.

## 6. Thí nghiệm 3 (§5) — pipeline không giám sát ở quy mô lớn

Động lực: các công trình circuit trước đó (Wang et al. 2023; Conmy et al. 2023; Hanna et al. 2023) đều cần dataset do người soạn để định nghĩa hành vi. Nhưng LM có vô số hành vi, nhiều cái con người không nghĩ ra. Thí nghiệm này chạy toàn bộ trên Pythia-70M.

**Bước 1 — phát hiện hành vi bằng clustering** (theo Michaud et al., 2023):
- Corpus: một tập con lớn của The Pile.
- Lọc: chỉ giữ cặp (context, answer token) mà Pythia-70M dự đoán đúng và tự tin, cross-entropy < 0.1 hoặc 0.3 nat tuỳ thí nghiệm.
- Loại mẫu induction: bỏ mẫu mà bigram (token cuối context, answer token) đã xuất hiện trước đó trong context. Lý do: model luôn đạt loss thấp trên induction nên nó sẽ lấn át các cluster khác.
- Vector `v_i` gán cho mỗi mẫu, thử nhiều lựa chọn (App. G.2): dense activation, sparse (SAE) activation, dense component indirect effect, sparse indirect effect, và gradient theo tham số `∇_θ log P_θ(y_i|x_i)` như Michaud et al.
- Thuật toán: spectral clustering trên 8192 mẫu, dùng ma trận cosine similarity chuẩn hoá về angular similarity `Ĉ_ij = 1 − arccos(C_ij)/π`, cài đặt scikit-learn. Với gradient theo tham số thì scale lên 100 000 mẫu và chuyển sang k-means, clustering trên **sparse random projection** xuống 30 000 chiều, ma trận chiếu có entry trong {−1, 0, 1}, xác suất khác 0 là 32/30000 (chỉ xét tham số không phải embedding và không phải layernorm).
- Chọn k sao cho tối đa hoá số cluster có mặt trong nhiều hơn một input context.

**Bước 2 — tìm circuit**: với mỗi subcorpus `D`, chạy biến thể zero-ablation của §3 với metric `m = −log P(y_i|x_i)`.

**Hai ví dụ cluster (Figure 5, trang 9)** — đều cho thấy một điều: cluster **trông như** một cơ chế, nhưng circuit cho thấy nó là **hợp của nhiều cơ chế riêng biệt**.

- *Cluster 382 — dãy tăng dần* (`var input = [1, 2, 3, 4...`, `Step 1 / Step 2 / Step 3 / Step 4`). Circuit chứa hai loại feature: **succession feature** (Chapter 1 → Chapter 2; A, B, C; I, II, III, IV) và **narrow induction feature** nhận mẫu `A3 ... A` rồi copy thông tin từ token `3`, kết hợp với succession để ra `A3 ... A → 4`.
- *Cluster 475 — "to" làm tân ngữ nguyên mẫu*. Circuit chứa hai đường riêng: một feature thúc đẩy "to" sau các từ nhận tân ngữ nguyên mẫu (*allowed to*, *would like ... to*, *asked to*); một feature khác kích hoạt trên tân ngữ của động từ/giới từ và thúc đẩy "to" như object complement (*Direct the user to*, *It's up to you to*, *According to*, *leads to*).

Paper còn tự động tìm lại được attention feature dính líu tới **succession** và **induction** — hai hiện tượng trước đó chỉ được nghiên cứu ở mức attention head với dữ liệu do người soạn (Olsson et al., 2022; Gould et al., 2023).

Paper thừa nhận rõ: **đánh giá các cluster và circuit này là một bài toán mở chưa giải.**

## 7. Limitations (§8, nguyên văn của tác giả)

1. Phương pháp phụ thuộc vào việc **có sẵn SAE** cho model. Train SAE tốn compute lớn (dù chỉ một lần).
2. **Component nào SAE không nắm bắt được thì vẫn không diễn giải được** sau khi áp dụng phương pháp.
3. **Phần lớn đánh giá là định tính.** Chỉ §4 cho bằng chứng định lượng rằng feature circuit hữu ích. Đánh giá dictionary và circuit khi không có task downstream là bài toán khó.
4. **Gán nhãn feature là quá trình định tính**: nhãn có thể khác nhau giữa các annotator và thay đổi theo task quan tâm.

## 8. Nguồn tài trợ (để nhắc nếu seminar có phần bối cảnh cộng đồng)
S. Marks nhận grant alignment của Open Philanthropy; C. Rager nhận Manifund Regrants; E. J. Michaud nhận NSF GRFP (2141064); Y. Belinkov nhận Israel Science Foundation (448/20) và Azrieli Early Career Faculty Fellowship; Y. Belinkov và D. Bau cùng nhận một grant alignment của Open Philanthropy; A. Mueller nhận Zuckerman postdoctoral fellowship. Phần cảm ơn có nêu Stephen Casper, Buck Shlegeris, Ryan Greenblatt, Neel Nanda (thảo luận ý tưởng dẫn tới §4), Logan Riggs và Jannik Brinkmann (giúp train SAE), Josh Engels và Max Tegmark (thảo luận về clustering).
