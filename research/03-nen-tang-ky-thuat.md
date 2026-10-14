# Nền tảng kỹ thuật — những gì phải giải thích trước khi vào paper

File này dành cho các slide nền. Mỗi mục nêu: khái niệm là gì, vì sao paper cần nó, và chi tiết có thể đưa vào slide phụ.

## 1. Superposition và polysemanticity

**Phát biểu.** Một LM có `d` chiều trong residual stream nhưng cần biểu diễn số concept lớn hơn `d` rất nhiều. Elhage et al. (2022) chỉ ra mạng giải quyết bằng cách gán mỗi concept một **hướng** trong `R^d`, và các hướng này không trực giao. Khi số feature vượt số chiều, chúng chồng lấn — gọi là superposition.

**Hệ quả trực tiếp.** Một neuron (một trục toạ độ) sẽ nhận đóng góp từ nhiều feature khác nhau → nó kích hoạt trên các ngữ cảnh không liên quan gì nhau → **polysemantic**. Đây là lý do Bảng 7 của SFC cho neuron điểm 32.6/100 còn SAE feature được 52.8–81.5.

**Vì sao paper cần điều này.** Nếu neuron đã diễn giải được thì không cần SAE, và circuit trên neuron đã đủ. Toàn bộ §2 của SFC tồn tại vì superposition.

**Điều kiện để SAE giải quyết được superposition** (linear representation hypothesis): concept được mã hoá tuyến tính, và tại mỗi thời điểm chỉ một số ít concept đang hoạt động. Nếu giả định này sai thì SAE sai — đây là điểm tấn công chính của các phê bình năm 2025.

## 2. Sparse autoencoder

### Kiến trúc ReLU (bản SFC train cho Pythia-70M, App. B.1.1)

Tham số: `W_E ∈ R^{d_SAE × d_model}`, `W_D ∈ R^{d_model × d_SAE}`, `b_E ∈ R^{d_SAE}`, `b_D ∈ R^{d_model}`, các cột của `W_D` bị ràng buộc là vector đơn vị.

```
f = ReLU(W_E (x − b_D) + b_E)          # tied pre-encoder bias: trừ b_D trước khi encode
x̂ = W_D f + b_D
L = ‖x̂ − x‖² + λ‖f‖₁                   # λ = 0.1
```

Feature vector `v_i` là cột thứ `i` của `W_D`.

### Các biến thể encoder cần biết (để đối chiếu Pythia vs Gemma)

| Kiến trúc | Cơ chế | Vấn đề nó giải quyết |
|---|---|---|
| ReLU + L1 (Bricken 2023) | `ReLU(W_E x + b_E)`, phạt L1 | Bản gốc. L1 vừa ép thưa vừa **co nhỏ** giá trị activation (shrinkage) → tái tạo kém |
| Gated (Rajamanoharan 2024a) | Tách nhánh quyết định "có kích hoạt không" khỏi nhánh quyết định "bao nhiêu" | Bỏ shrinkage |
| TopK (Gao et al. 2024) | Giữ đúng k activation lớn nhất, bỏ L1 | Kiểm soát L0 trực tiếp, không có shrinkage |
| **JumpReLU** (Rajamanoharan 2024b) | Ngưỡng `θ` học được: activation nào dưới `θ` thì về 0 | **Kiến trúc của Gemma Scope → của phần Gemma trong SFC** |
| BatchTopK, Matryoshka | TopK theo batch; latent phân cấp nhiều mức | Ra sau SFC, xem file 05 |

### Các chỉ số đánh giá SAE (App. B.1.3 — dùng để đọc Bảng 3–6 của paper)

- **Variance explained** = `1 − Var(x − x̂)/Var(x)`.
- **L0** = số feature kích hoạt trung bình mỗi token. Đây là thước đo độ thưa thực tế.
- **L1** = tổng độ lớn activation.
- **% alive** = tỉ lệ feature kích hoạt ít nhất một lần trên batch 512 token. Trong SFC con số này chỉ 12–44% — phần lớn feature là **dead feature**.
- **CE diff** = chênh lệch cross-entropy của model khi thay `x` bằng `x̂`.
- **% CE recovered** = CE diff chuẩn hoá theo chênh lệch giữa CE gốc và CE khi zero-ablate `x`.

### SAE error — điểm thiết kế riêng của SFC

`ε(x) = x − x̂` là phần dư. Hai lựa chọn khả dĩ:
- Vứt đi → circuit "sạch" nhưng không trung thực, sai số cộng dồn qua các layer.
- **Giữ lại làm node** → SFC chọn cách này.

Đánh đổi: circuit trung thực hơn, nhưng error node là vector `d_model` chiều, không đọc được. Figure 3 của paper cho thấy bỏ residual-stream error node phá hỏng model; bỏ error node attn/MLP thì ít hại hơn. Hàm ý: residual stream error chứa lượng lớn thông tin mà SAE chưa mã hoá.

## 3. Activation patching và indirect effect

### Activation patching cơ bản
Chạy model trên `x_clean`, lưu mọi activation. Chạy lại trên `x_clean` nhưng **ép** một node `a` mang giá trị nó có khi chạy `x_patch`. Chênh lệch metric là indirect effect của `a`.

Công thức SFC (Eq. 2):
```
IE(m; a; x_clean, x_patch) = m(x_clean | do(a = a_patch)) − m(x_clean)
```

Ký hiệu `do(·)` là toán tử can thiệp của Pearl — phân biệt với điều kiện hoá. Nó nói: ép `a` nhận giá trị đó, cắt mọi nguyên nhân sinh ra `a`.

### Ba kiểu ablation trong paper

| Kiểu | Định nghĩa | SFC dùng ở đâu |
|---|---|---|
| **Patching** (contrastive pair) | `a ← a_patch` | §3, subject-verb agreement |
| **Zero-ablation** | `a ← 0` | §4 (SHIFT), §5 (cluster) — vì chỉ có một input, không có cặp tương phản |
| **Mean-ablation** | `a ←` giá trị trung bình của `a` trên `D`, tính riêng theo từng vị trí token | Đánh giá faithfulness: mọi node **ngoài** circuit bị mean-ablate |

Lý do mean-ablation cho faithfulness: nó biểu diễn hiệu năng "tiên nghiệm" của model khi biết về task nhưng không biết gì về input cụ thể. Zero-ablation sẽ đưa model ra khỏi phân phối dữ liệu.

### Vì sao cần xấp xỉ
Model có `L` layer × 4 submodule × `d_SAE` feature × `T` vị trí token node. Với Pythia-70M: 6 layer, `d_SAE = 64 × 512 = 32 768`. Chạy patching riêng cho từng node là bất khả thi. Attribution patching cho toàn bộ node trong **2 forward + 1 backward pass**.

### Khi nào attribution patching sai
Xấp xỉ Taylor bậc nhất chỉ đúng khi `m` gần tuyến tính quanh `a_clean`. Nó hỏng khi:
- Component bão hoà (ReLU, softmax ở vùng cực trị).
- Hiệu `a_patch − a_clean` lớn.

SFC đo và xác nhận (Figure 25): hỏng ở **MLP layer 0** và **residual stream các layer đầu** — đúng chỗ activation biến động mạnh nhất giữa các token khác nhau. Integrated gradients lấy gradient trung bình dọc đường thẳng từ `a_patch` tới `a_clean` nên không bị bẫy này.

## 4. Faithfulness và completeness

```
faithfulness(C) = (m(C) − m(∅)) / (m(M) − m(∅))
completeness(C) = faithfulness(M \ C)
```
- `m(C)`: chạy model, mean-ablate mọi thứ ngoài `C`.
- `m(∅)`: mean-ablate tất cả.
- `m(M)`: model đầy đủ.

Đọc: faithfulness = tỉ lệ hiệu năng model mà circuit giải thích được. Lý tưởng 1. Completeness lý tưởng 0 — nghĩa là phần bù của circuit không làm được task.

Hai chi tiết phương pháp luận quan trọng cần nêu nếu có người hỏi:
- Faithfulness có thể **vượt quá 1** nếu circuit chạy tốt hơn full model (hiện tượng đã thấy trong các paper circuit khác).
- SFC **bỏ 1/3 circuit ở layer đầu** khi đánh giá, vì component layer đầu gắn với token cụ thể mà train/test split không dùng chung. Đây là một lựa chọn có thể bị chất vấn.

## 5. Nghề nghiệp vs giới tính — đặt bài toán spurious correlation cho đúng

**Spurious correlation**: một đặc trưng tương quan với nhãn trong dữ liệu train nhưng không phải nguyên nhân của nhãn, và tương quan đó không giữ khi phân phối đổi.

**Worst-group accuracy**: chia dữ liệu theo tổ hợp (nhãn đích × thuộc tính spurious), lấy accuracy thấp nhất trong các nhóm. Trong BiB: 4 nhóm = {nam, nữ} × {giáo sư, y tá}. Đây là chỉ số chuẩn của literature group robustness (Sagawa et al. 2020).

**Disambiguating data** — khái niệm then chốt của §4. Là dữ liệu trong đó tín hiệu spurious **kém dự báo nhãn hơn** tín hiệu đích. Gần như mọi phương pháp trước SHIFT đều cần nó ở dạng nào đó:
- Group DRO (Sagawa 2020): cần nhãn nhóm.
- JTT (Liu et al. 2021): không cần nhãn nhóm, nhưng cần trong dữ liệu **có** những mẫu mà tín hiệu spurious dự báo sai.
- Reweighting/balancing (Idrissi 2022, Nam 2020/2022, Utama 2020): cần dữ liệu để ước lượng nhóm.
- Concept erasure (INLP — Ravfogel 2020; LEACE — Belrose 2023): cần nhãn của **thuộc tính cần xoá**.

Trong ambiguous set của SFC, giới tính dự báo nhãn **100%**. Mọi phương pháp trên đều không có gì để bám vào. Đó là điểm SHIFT khác về bản chất: nó lấy tín hiệu phân định từ **phán đoán của người đọc feature**, không từ dữ liệu.

## 6. Hai model được dùng

| | Pythia-70M | Gemma-2-2B |
|---|---|---|
| Nguồn | Biderman et al. 2023 (EleutherAI) | Gemma Team 2024 (Google DeepMind) |
| Layer | 6 | 26 |
| Mục đích trong paper | model nhỏ để chạy đầy đủ mọi thí nghiệm, kể cả §5 quy mô lớn | kiểm tra phương pháp có scale không |
| SAE | nhóm tự train, ReLU, `64 × d` | Gemma Scope, JumpReLU, width 16384 |

Pythia được chọn vì nó là suite mở hoàn toàn (mọi checkpoint, mọi dữ liệu train công khai), đúng mục đích mà Biderman et al. thiết kế nó.

## 7. Clustering để phát hiện hành vi (nền cho §5)

Ý tưởng gốc: **Michaud et al. 2023, The Quantization Model of Neural Scaling**. Giả thuyết: năng lực của LM phân rã thành một tập rời rạc các "quantum" kỹ năng; scaling law xuất hiện từ phân phối tần suất của các quantum đó. Cách tìm quantum: gán cho mỗi mẫu `(x_i, y_i)` vector `∇_θ log P_θ(y_i|x_i)` — gradient theo tham số — rồi clustering. Hai mẫu cùng cluster nghĩa là model dùng **cùng bộ tham số theo cùng cách** để dự đoán chúng.

SFC lấy nguyên ý này rồi thay/thêm các lựa chọn vector đầu vào (dense activation, sparse activation, dense IE, sparse IE), và nối kết quả vào pipeline tìm circuit. Đóng góp: từ "đây là một cluster hành vi" tiến tới "đây là cơ chế tạo ra hành vi đó".

**Spectral clustering** dùng ở đây: xây ma trận tương đồng `C_ij` = cosine similarity, đổi sang angular similarity `Ĉ_ij = 1 − arccos(C_ij)/π` để mọi phần tử nằm trong [0,1], rồi chạy spectral clustering (cài đặt scikit-learn, giai đoạn cuối dùng k-means). Với 100 000 mẫu thì spectral clustering không chạy nổi nên chuyển sang k-means thuần, trên **sparse random projection** của gradient xuống 30 000 chiều — theo Johnson–Lindenstrauss, phép chiếu ngẫu nhiên giữ gần đúng khoảng cách đôi một.

## 8. Bảng thuật ngữ Anh–Việt

| Tiếng Anh | Cách gọi trong slide | Giải thích một câu |
|---|---|---|
| sparse feature circuit | giữ nguyên | đồ thị con nhân quả gồm các SAE feature và SAE error |
| SAE feature | feature | một hướng trong không gian ẩn, do SAE học ra |
| SAE error | error node | phần activation SAE không tái tạo được, giữ làm node trong circuit |
| polysemantic | đa nghĩa | một đơn vị tham gia nhiều chức năng không liên quan |
| superposition | giữ nguyên | model nén nhiều feature hơn số chiều bằng các hướng không trực giao |
| indirect effect (IE) | hiệu ứng gián tiếp | thay đổi của metric khi ép một node nhận giá trị khác |
| activation patching | giữ nguyên | ép activation của một node sang giá trị từ input khác |
| attribution patching | giữ nguyên | xấp xỉ Taylor bậc nhất của activation patching |
| integrated gradients | giữ nguyên | trung bình gradient dọc đường nội suy giữa hai giá trị |
| zero-ablation / mean-ablation | ablate về 0 / về trung bình | hai cách vô hiệu hoá một node |
| faithfulness | độ trung thực | phần hiệu năng model mà circuit giải thích được |
| completeness | độ đầy đủ | đo bằng faithfulness của phần bù circuit |
| spurious correlation | tương quan giả | đặc trưng tương quan với nhãn nhưng không phải nguyên nhân |
| disambiguating data | dữ liệu phân định | dữ liệu trong đó tín hiệu spurious kém dự báo hơn tín hiệu đích |
| worst-group accuracy | accuracy nhóm kém nhất | accuracy thấp nhất trên các tổ hợp (nhãn × thuộc tính spurious) |
| dead feature | feature chết | feature của SAE không bao giờ kích hoạt |
| feature absorption | giữ nguyên | latent A "nuốt" một phần latent B khi A ⟹ B |
| NP number tracker | giữ nguyên | feature theo dõi số của danh từ trung tâm trong noun phrase |
| PP/RC boundary detector | giữ nguyên | feature phát hiện ranh giới cụm giới từ / mệnh đề quan hệ |
