# Bối cảnh và lịch sử — con đường dẫn tới Sparse Feature Circuits

File này dựng dòng thời gian của hai nhánh nghiên cứu độc lập, và chỉ ra chính xác SFC hợp chúng lại ở đâu.

## 1. Hai nhánh hội tụ

SFC = **circuit analysis** (nhánh A) chạy trên **SAE feature** (nhánh B), với chi phí được kéo xuống bằng **xấp xỉ gradient** (nhánh C).

```
Nhánh A: circuit trên component thô        Nhánh B: tìm đơn vị diễn giải được
  Zoom In (2020)                             Toy Models of Superposition (2022)
  Math Framework for Transformers (2021)     Towards Monosemanticity (2023)
  Induction heads (2022)                     Cunningham et al. SAE (2023/24)
  IOI circuit (2023)                         Gemma Scope (2024)
  ACDC (2023)                                        |
        |                                            |
        +------------- Nhánh C: attribution patching (2022), AtP* (2024),
        |                       integrated gradients (2017)
        v
   SPARSE FEATURE CIRCUITS (03/2024 → ICLR 2025 Oral)
        |
        v
   Attribution graphs / cross-layer transcoder (Anthropic 03/2025)
```

## 2. Nhánh A — circuit analysis

| Năm | Công trình | Đóng góp và giới hạn |
|---|---|---|
| 1992, 2001 | Robins & Greenland; Pearl, *Direct and indirect effects* | Định nghĩa indirect effect trong causal mediation analysis. Đây là công thức Eq. 2 của SFC, nhập khẩu nguyên vẹn từ thống kê nhân quả. |
| 1973 | Lewis, *Counterfactuals* | Nền triết học cho can thiệp phản thực (counterfactual intervention). |
| 2020 | Vig et al., *Investigating gender bias in LMs using causal mediation analysis* | Lần đầu áp dụng causal mediation vào LM, và đúng trên bài toán gender bias — tiền thân trực tiếp của §4 SFC. |
| 2020 | Olah et al., *Zoom In: An Introduction to Circuits* (Distill) | Đặt ra từ "circuit". Không có trong reference list của SFC nhưng là nguồn của toàn bộ từ vựng. |
| 2021 | Elhage et al., *A Mathematical Framework for Transformer Circuits* | Phân rã transformer thành residual stream + head, khái niệm QK/OV circuit. Không nằm trong reference list nhưng cần cho phần nền của seminar. |
| 2021 | Finlayson et al., *Causal analysis of syntactic agreement mechanisms* | **Nguồn dữ liệu cho §3 của SFC.** Cũng là nơi phát hiện rằng model xử lý simple agreement khác với agreement qua RC/PP — SFC xác nhận lại ở mức feature. |
| 2022 | Olsson et al., *In-context learning and induction heads* | Induction head. SFC §5 tìm lại được induction ở mức feature một cách tự động, không cần dữ liệu người soạn. |
| 2022 | Meng et al., ROME (*Locating and editing factual associations in GPT*) | Causal tracing, chỉ ra MLP tầng giữa lưu factual association. Đại diện cho "circuit trên component thô". David Bau là đồng tác giả cả ROME lẫn SFC. |
| 2023 | Wang et al., *Interpretability in the Wild: IOI circuit in GPT-2 small* | Circuit thủ công, chi tiết nhất thời điểm đó: duplicate token head, S-inhibition head, name mover head. Cũng là nguồn của cách mean-ablate theo vị trí token mà SFC dùng. |
| 2023 | Hanna et al., *How does GPT-2 compute greater-than?* | Circuit thủ công cho tác vụ số học. |
| 2023 | Conmy et al., **ACDC** | Tự động hoá việc tìm circuit bằng activation patching từng edge. Chính xác nhưng chi phí tỉ lệ với số edge. |
| 2023 | Gould et al., *Successor heads* | Head thực hiện phép "kế tiếp". SFC §5 tìm lại được ở mức feature (cluster 382). |
| 2023 | Geiger et al., *Causal abstraction*; Geiger et al. 2021, 2022 | Kiểm tra xem một causal graph giả định có khớp với cơ chế trong LM không. SFC định vị mình ngược lại: **tìm cơ chế mà không cần giả thuyết trước**. |
| 2024 | Mueller et al., *The Quest for the Right Mediator* | Survey + nền lý thuyết cho causal interpretability. Aaron Mueller và Samuel Marks đều là đồng tác giả — đây là bài survey nên đọc trước SFC. |

## 3. Nhánh B — từ superposition tới SAE

| Năm | Công trình | Nội dung |
|---|---|---|
| 1992 | Schmidhuber, *Learning factorial codes by predictability minimization* | Tiền sử xa của feature disentanglement. |
| 2013 | Makhzani & Frey, *k-sparse autoencoders* | Kiến trúc SAE với ràng buộc L0, quay lại thành TopK SAE của OpenAI năm 2024. |
| 2017 | Higgins et al., β-VAE; 2018 Kim & Mnih | Disentanglement trong thị giác máy tính. |
| 2018 | Kim et al., **TCAV** | Concept activation vector: cần người định nghĩa concept trước, cần dữ liệu ví dụ. SFC coi đây là ví dụ của "supervised, cần giả thuyết trước". |
| 2022 | **Elhage et al., Toy Models of Superposition** | Bài quan trọng nhất của nhánh này. Chỉ ra mạng nơ-ron nén nhiều feature hơn số chiều bằng cách biểu diễn mỗi feature là một **hướng**, không phải một neuron. Hệ quả trực tiếp: neuron polysemantic, và muốn có đơn vị mịn diễn giải được thì phải đi tìm hướng chứ không phải neuron. Đây là lý do tồn tại của SAE. |
| 2022 | Belinkov, *Probing classifiers: promises, shortcomings, and advances* | Tổng kết giới hạn của probing: probe đo tương quan, không chứng minh model dùng thông tin đó. |
| 10/2023 | **Bricken et al., Towards Monosemanticity** (Anthropic, Transformer Circuits Thread) | Dictionary learning bằng SAE trên một transformer 1 layer. Kiến trúc ReLU + tied pre-encoder bias mà SFC dùng cho Pythia chính là kiến trúc này; thủ tục resample dead feature cũng lấy từ đây. |
| 2023/2024 | **Cunningham et al., Sparse autoencoders find highly interpretable features** (ICLR 2024) | Bản song song, độc lập, từ giới học thuật. Chứng minh SAE feature diễn giải được hơn neuron và có thể can thiệp nhân quả. |
| 05/2024 | Templeton et al., **Scaling Monosemanticity** (Claude 3 Sonnet) | Chứng minh SAE chạy được trên model production. Nổi tiếng với feature "Golden Gate Bridge". |
| 06/2024 | Gao et al., *Scaling and evaluating sparse autoencoders* (OpenAI) | **TopK SAE**: thay L1 bằng ràng buộc L0 cứng, bỏ được shrinkage do L1. |
| 2024a | Rajamanoharan et al., **Gated SAE** | Tách quyết định "feature có kích hoạt không" khỏi "kích hoạt bao nhiêu", giảm shrinkage. |
| 07/2024b | Rajamanoharan et al., **JumpReLU SAE** | Ngưỡng học được thay cho ReLU. Kiến trúc của Gemma Scope, tức là kiến trúc SFC dùng cho Gemma-2-2B. |
| 07/2024 | **Lieberum et al., Gemma Scope** | Bộ SAE mở cho mọi sublayer của cả họ Gemma 2. Hạ tầng dùng chung; SFC dùng trực tiếp. |
| 2023– | Lin & Bloom, **Neuronpedia** | Giao diện duyệt feature. SFC dùng nó ở bước con người đọc feature trong SHIFT. Không có công cụ này thì bước 2 của SHIFT rất tốn công. |
| 09/2024 | Chanin et al., *A is for Absorption* | Phát hiện **feature absorption**: với concept phân cấp A ⟹ B, sparsity đẩy SAE học latent cho A và latent cho "B trừ A", nên latent "bắt đầu bằng chữ S" có thể không kích hoạt trên 5% token bắt đầu bằng S. Đây là phê bình có sức nặng nhất với giả định "một feature = một concept" mà SFC dựa vào. |

## 4. Nhánh C — làm cho việc tìm circuit chạy nổi

| Năm | Công trình | Nội dung |
|---|---|---|
| 2017 | Sundararajan et al., **Integrated Gradients** | Axiomatic attribution. SFC dùng làm Eq. 4 với N = 10 bước. |
| 2022 | Nanda, **Attribution patching** (blog) | Xấp xỉ Taylor bậc nhất cho activation patching: 2 forward + 1 backward pass cho toàn bộ node. Đây là Eq. 3 của SFC. |
| 2023 | Syed, Rager & Conmy, **EAP** (*Attribution patching outperforms automated circuit discovery*) | Áp attribution patching lên edge. Can Rager là đồng tác giả cả EAP lẫn SFC. |
| 2024 | Kramár et al., **AtP\*** | Phân tích khi nào attribution patching sai và cách sửa. SFC trích dẫn để giải thích vì sao cần integrated gradients ở layer đầu. |
| 2024 | Hanna, Pezzelle & Belinkov, *Have faith in faithfulness* (**EAP-IG**) | Ghép integrated gradients vào EAP, và lập luận rằng nên đánh giá circuit bằng faithfulness chứ không phải bằng độ chồng lấp với circuit đã biết. Yonatan Belinkov là đồng tác giả cả hai. |

## 5. Mốc thời gian gọn (dùng làm một slide)

| Thời điểm | Sự kiện |
|---|---|
| 2020 | "Circuits" thành từ khoá (Distill, Olah et al.) |
| 09/2022 | Toy Models of Superposition → nêu rõ vì sao neuron không dùng được |
| 2022–2023 | Circuit thủ công đạt đỉnh: induction heads, IOI, greater-than |
| 2023 | ACDC + attribution patching → tự động hoá, nhưng vẫn trên component thô |
| 10/2023 | Towards Monosemanticity → SAE thành công cụ chính |
| 03/2024 | **SFC arXiv v1 (2403.19647)** |
| 05–07/2024 | Scaling Monosemanticity, TopK/Gated/JumpReLU SAE, Gemma Scope |
| 09/2024 | Feature absorption — phê bình đầu tiên có trọng lượng |
| 11/2024 | SHIFT được chuyển thành metric đánh giá SAE (arXiv 2411.18895) |
| 02/2025 | *Are Sparse Autoencoders Useful?* — SAE thua baseline ở sparse probing |
| 03/2025 | SAEBench; DeepMind công bố negative results và giảm ưu tiên nghiên cứu SAE |
| 03/2025 | Anthropic công bố Circuit Tracing + On the Biology of a Large Language Model |
| **2025** | **SFC được nhận Oral tại ICLR 2025**; bản v3 ngày 27/03/2025 |
| 04/2025 | Kharlapenko et al. mở rộng SFC lên Gemma-1 2B cho in-context learning |
| 11/2025 | OpenAI, weight-sparse transformers — hướng thay thế: train model để nó dễ diễn giải sẵn |

Chi tiết các mốc từ 2024 trở đi nằm ở `05-sau-paper-va-tranh-luan.md`.

## 6. Vì sao thời điểm 03/2024 là thời điểm SFC phải xuất hiện

Ba điều kiện cần vừa đủ chín cùng lúc:

1. **SAE đủ tốt và có sẵn công khai** (Bricken 10/2023, Cunningham 2023). Trước đó không có đơn vị mịn nào vừa diễn giải được vừa không cần giả thuyết trước.
2. **Attribution patching đã được chứng minh đủ chính xác** (Nanda 2022, Syed 2023). Không có nó thì việc patch ~32 768 feature × 4 submodule × 6 layer × mỗi vị trí token là bất khả thi.
3. **Có động lực từ AI safety**: cần gỡ tín hiệu không mong muốn khỏi model trong tình huống không có dữ liệu phân định (Burns et al. 2023 weak-to-strong, Ngo et al. 2024). §4 của SFC trả lời trực tiếp mối lo này.

Đóng góp riêng của SFC, tách khỏi ba điều kiện trên: **giữ SAE error làm node trong graph**, và **cho thấy circuit trên feature dùng được cho một tác vụ downstream đo được bằng số**, chứ không chỉ để đọc cho vui.
