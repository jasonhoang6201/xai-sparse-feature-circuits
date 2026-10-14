# Sau paper — công trình kế thừa, phê bình, và trạng thái hiện tại

Tính tới 09/2026. Mỗi mục ghi mức độ kiểm chứng: **[đã đọc nguồn]** = đã mở trang gốc; **[từ kết quả tìm kiếm]** = chỉ thấy tiêu đề/tóm tắt qua tìm kiếm, nên kiểm lại trước khi đưa lên slide.

---

## 1. Công trình kế thừa trực tiếp SFC

### 1.1 SHIFT trở thành metric đánh giá SAE — SCR và TPP
**[đã đọc nguồn]** Karvonen, Rager, **Marks**, Nanda. *Evaluating Sparse Autoencoders on Targeted Concept Erasure Tasks*. arXiv:2411.18895, 28/11/2024.

Hai tác giả của SFC (Can Rager, Samuel Marks) tự chuyển SHIFT thành thước đo tự động:
- **SCR (Spurious Correlation Removal)**: chạy SHIFT nhưng **thay người chú thích bằng một LLM**. Điểm số = tỉ lệ cải thiện đạt được nhờ ablation so với mức cải thiện tối đa có thể. Hàm ý đo: SAE có tách được "giới tính" khỏi "nghề nghiệp" thành các latent riêng không.
- **TPP (Targeted Probe Perturbation)**: mở rộng SHIFT sang dataset đa lớp. Đo xem ablate tập latent của một concept có tác động **chọn lọc** lên đúng concept đó không, hay lan sang concept khác.

Đây là chuyển hoá quan trọng nhất của SFC: từ một **ứng dụng** thành một **benchmark** mà cả cộng đồng dùng.

### 1.2 SAEBench
**[đã đọc nguồn]** *SAEBench: A Comprehensive Benchmark for Sparse Autoencoders in Language Model Interpretability*. arXiv:2503.09532, 03/2025.

SCR và TPP nằm trong 8 metric của SAEBench. Benchmark so 7 kiến trúc SAE (ReLU, TopK, BatchTopK, JumpReLU, Gated, P-anneal, Matryoshka BatchTopK) × 3 bề rộng (4k, 16k, 65k) × 6 mức thưa, trên Pythia-160M và Gemma-2-2B.

Kết quả chính:
- **Matryoshka SAE thắng 5/8 metric**, đặc biệt mạnh ở TPP, SCR, RAVEL và feature absorption.
- **Mức thưa tối ưu phụ thuộc tác vụ**: L0 thấp có lợi cho interpretability; L0 cao có lợi cho reconstruction.
- **TopK và JumpReLU tái tạo tốt hơn ReLU nhưng feature absorption tệ hơn.** Đây là một đánh đổi mà SFC không có cách nào biết ở thời điểm 2024.

### 1.3 Mở rộng SFC lên model lớn hơn và task phức tạp hơn
**[từ kết quả tìm kiếm]** Kharlapenko, Shabalin, Barez, Conmy, Nanda. *Scaling sparse feature circuit finding for in-context learning*. arXiv:2504.13756, 18/04/2025.

Áp phương pháp của Marks et al. lên Gemma-1 2B (gấp ~30 lần số tham số Pythia-70M) và lên in-context learning. Tìm được SAE feature **trừu tượng** mà (i) mã hoá việc model biết đang phải làm task nào, và (ii) latent vector của nó **gây ra** task đó theo kiểu zero-shot. Có một bản mở rộng tiếp lên Gemma 9B đăng trên LessWrong.

### 1.4 Vai trò SAE error node bị đặt câu hỏi
**[từ kết quả tìm kiếm]** *What is the functional role of SAE errors?* (LessWrong).

Quan sát: ablate residual error node trong feature circuit huỷ hoàn toàn hiệu năng model — nhưng có thể bù lại bằng cách khôi phục một nhóm hẹp feature ở layer giữa-muộn. Giả thuyết được nêu: SAE error chứa biểu diễn feature trung gian của **cross-layer superposition** — tức feature trải qua nhiều layer, mà SAE một-layer không bắt được. Nếu đúng, đây là lý do kỹ thuật vì sao cross-layer transcoder (mục 3) ra đời.

---

## 2. Ba đợt phê bình SAE, 2024–2026

Đây là phần seminar cần có nếu muốn trình bày trung thực. SFC là một trong những ứng dụng mạnh nhất của SAE, nên số phận của nó gắn với số phận của SAE.

### Đợt 1 (09/2024) — feature absorption
**[từ kết quả tìm kiếm]** Chanin et al. *A is for Absorption: Studying Feature Splitting and Absorption in Sparse Autoencoders*. arXiv:2409.14507.

Với concept phân cấp A ⟹ B (India ⟹ Asia), mục tiêu thưa khiến SAE học latent cho A và latent cho "B trừ A" để giảm dư thừa. Hệ quả cụ thể: latent "bắt đầu bằng chữ S" kích hoạt trên 95% token bắt đầu bằng S nhưng **im lặng trên 5% còn lại một cách tuỳ tiện**, vì phần đó đã bị latent khác hấp thụ. Kết luận của paper: đổi bề rộng hay mức thưa **không** giải quyết được.

Ảnh hưởng tới SFC: nếu một feature không kích hoạt đúng nơi nó phải kích hoạt, thì nhãn con người gán cho nó ở bước 2 của SHIFT có thể sai, và circuit có thể bỏ sót đường đi.

### Đợt 2 (02–03/2025) — SAE thua baseline trên tác vụ downstream

**[đã đọc nguồn]** Kantamneni et al. *Are Sparse Autoencoders Useful? A Case Study in Sparse Probing*. ICML 2025, arXiv:2502.16681.
Thử SAE làm probing trong bốn chế độ khó: dữ liệu ít, mất cân bằng lớp, nhãn nhiễu, covariate shift. Kết luận: **SAE không vượt được baseline đơn giản** như logistic regression hay prompting. Thông điệp phương pháp luận: phải đánh giá interpretability method trên tác vụ downstream, có baseline mạnh.

**[đã đọc nguồn]** DeepMind Mech Interp Team. *Negative Results for Sparse Autoencoders On Downstream Tasks and Deprioritising SAE Research*. 26/03/2025 (LessWrong / Medium).
- Tác vụ: phát hiện ý đồ gây hại trong prompt, đánh giá OOD generalization.
- Baseline: dense linear probe trên residual stream.
- Số liệu: linear probe đạt AUROC ≈ 0.999 trên OOD. Single-feature SAE probe thua rõ. k-sparse SAE probe (k ≥ 5) bắt kịp in-distribution nhưng "tệ rõ rệt" trên OOD. SAE fine-tune trên chat closed ~50% khoảng cách nhưng vẫn kém. Bản thân việc tái tạo bằng SAE làm mất thông tin: chênh AUROC 0.048 trên OOD.
- Kết luận: giảm ưu tiên nghiên cứu SAE nền tảng; đánh giá rằng cộng đồng interpretability đang **đầu tư quá mức vào SAE**.
- Sắc thái cần giữ: nhóm này **không** bác bỏ hoàn toàn. Họ ghi nhận SAE vẫn hữu ích cho **debug thăm dò các lỗi chưa rõ nguyên nhân** và **phát hiện tương quan giả trong dataset** — tức đúng hai việc SFC §4 và §5 làm.

### Đợt 3 (2026) — chính benchmark bị đặt câu hỏi
**[đã đọc nguồn]** David Chanin. *Are Sparse Autoencoder Benchmarks Reliable?* arXiv:2605.18229, 18/05/2026.

Kết luận: **TPP và SCR ở cấu hình chuẩn trượt nhiều bài kiểm tra** (nhiễu khi đổi seed, tương quan với ground truth, khả năng phân biệt) và **không nên dùng để đánh giá SAE**. Biến thể sae-probes đáng tin nhất trong các metric được xét, dù cũng hạn chế khi phân biệt giữa các kiến trúc SAE.

Đây là vòng phản hồi khép kín cần nêu trong seminar: SHIFT (2024) → SCR (2024) → SAEBench (2025) → SCR bị bác (2026).

**[từ kết quả tìm kiếm]** Ngoài ra còn *Sanity Checks for Sparse Autoencoders: Do SAEs Beat Random Baselines?* (arXiv:2602.14111) và một kết quả trên setup tổng hợp có ground truth: SAE chỉ khôi phục **9% feature thật** dù đạt **71% variance explained** — tức reconstruction tốt không kéo theo tìm đúng feature. Cần kiểm chứng nguồn trước khi trích số này lên slide.

### Phản biện lại các phê bình
**[đã đọc nguồn]** Peng, Movva, Kleinberg, Pierson, Garg. *Position: Use Sparse Autoencoders to Discover Unknowns* (bản v1: *...Discover Unknown Concepts, Not to Act on Known Concepts*). arXiv:2506.23845, 30/06/2025, sửa 04/07/2026.

Luận điểm: các negative result đều đo SAE ở việc **hành động trên concept đã biết** (probing một concept đã đặt tên, xoá một thuộc tính đã đặt tên) — và ở đó baseline có giám sát đương nhiên mạnh hơn. Chỗ SAE không thay thế được là **phát hiện concept chưa biết**. Ứng dụng đề xuất: interpretability, explainability, fairness, auditing, safety; và khoa học xã hội, khoa học sức khoẻ.

**Đây là lập luận bảo vệ trực tiếp cho §5 của SFC** (pipeline không giám sát tìm hàng nghìn hành vi) và một phần cho §4 (con người phát hiện feature giới tính mà **không được cho biết trước** rằng giới tính là vấn đề).

---

## 3. Hướng thay thế và kế thừa về mặt phương pháp

### 3.1 Transcoder thay cho SAE
**[từ kết quả tìm kiếm]** Dunefsky, Chlenski, Nanda. *Transcoders Find Interpretable LLM Feature Circuits*. NeurIPS 2024.

Khác biệt cơ bản: SAE tái tạo **activation** của một component; transcoder xấp xỉ **hàm input→output** của component đó (thường là MLP) qua một bottleneck thưa. Hệ quả cho circuit analysis: circuit phân tách gọn thành phần **phụ thuộc input** và phần **bất biến theo input** (chỉ phụ thuộc trọng số), nên phân tích được ở mức trọng số chứ không chỉ mức activation.

Các biến thể sau đó: skip-transcoder (Paulo et al. 2025), cross-layer transcoder. *Transcoders Beat Sparse Autoencoders for Interpretability* (arXiv:2501.18823, 01/2025) đưa so sánh trực diện.

### 3.2 Attribution graphs — Anthropic, 03/2025
**[đã đọc nguồn]** Ameisen, Lindsey, Pearce, Gurnee, Turner et al. *Circuit Tracing: Revealing Computational Graphs in Language Models*, transformer-circuits.pub, 27/03/2025. Kèm bài ứng dụng *On the Biology of a Large Language Model* cùng ngày, chạy trên Claude 3.5 Haiku.

Đây là bản kế nhiệm gần nhất về mặt ý tưởng của SFC ở quy mô model production:

| | Sparse Feature Circuits (2024) | Circuit Tracing / Attribution Graphs (2025) |
|---|---|---|
| Đơn vị | SAE feature, per-layer | **Cross-layer transcoder (CLT)** feature: đọc residual stream tại một layer, ghi vào output của **mọi** MLP layer sau đó |
| Cạnh | ÎE bằng attribution patching / integrated gradients | hiệu ứng tuyến tính giữa node trên một prompt cụ thể |
| Xử lý phần dư | SAE error là node, có cả cạnh vào và ra | **Error node là node đầu vào, không có cạnh vào** — chỉ nhận, không giải thích |
| Model | Pythia-70M, Gemma-2-2B | Claude 3.5 Haiku |
| Kiểm chứng | faithfulness/completeness trên held-out; SHIFT | thí nghiệm nhiễu loạn (perturbation) theo sau mỗi giả thuyết |

Hạn chế Anthropic tự nêu, nhiều cái trùng với SFC: không giải thích được QK-circuit của attention (attention pattern bị cố định); lỗi tái tạo che mất phần tính toán quan trọng; không giải thích được vì sao một feature **không** kích hoạt (suppression); đồ thị vẫn khó đọc dù đã prune; chưa chắc transcoder dùng đúng cơ chế như MLP gốc.

Lưu ý khi trình bày: bản methods của Anthropic **không trích dẫn Marks et al.** trong phần tôi đọc được. Nên nói "hướng kế nhiệm về ý tưởng", không nói "xây trên SFC".

Thư viện `circuit-tracer` được nhóm Anthropic Fellows phát hành mã nguồn mở (ban đầu hỗ trợ Gemma-2-2B và Llama-3.1-1B, sau thêm CLT), chạy được trên Neuronpedia.

### 3.3 Train model để nó diễn giải được sẵn
**[từ kết quả tìm kiếm]** OpenAI. *Weight-sparse transformers have interpretable circuits*. arXiv:2511.13653, công bố 13/11/2025.

Đảo ngược hướng tiếp cận: thay vì phân tích model dày đặc bằng SAE, **train model với ràng buộc thưa ngay từ đầu** — ép ~99.9% trọng số về 0, cộng activation sparsity (AbsTopK giữ khoảng 1/4 neuron hoạt động mỗi token), giảm superposition ngay tại gốc. Kết quả nêu: ở cùng mức pretraining loss, circuit trong model thưa nhỏ hơn khoảng **16 lần** so với baseline dày đặc. Đánh đổi: thưa hơn thì năng lực kém hơn; tăng kích thước model cải thiện được đường biên năng lực–diễn giải.

Đây là phản đề đáng nêu cuối seminar: SFC nhận model như nó vốn có rồi tìm cách đọc; hướng này đổi cách train để khỏi phải đọc khó.

---

## 4. Trạng thái hiện tại (2026) — tóm tắt để kết luận seminar

**[từ kết quả tìm kiếm, cần kiểm lại từng ý]**

- Hạ tầng SAE mở đã thành chuẩn: Gemma Scope, và các bộ tiếp theo cho họ model khác (Qwen-Scope, Llama-Scope, GemmaScope 2).
- Mech interp chuyển từ hướng nghiên cứu hẹp sang một ngành công cụ cho debugging, auditing và safety; được MIT Technology Review xếp vào danh sách công nghệ đột phá 2026.
- SAE không bị bỏ, nhưng vị trí đã đổi: từ "công cụ trung tâm" thành "một công cụ, mạnh ở việc phát hiện concept chưa biết, yếu ở việc thao tác trên concept đã biết".
- Cross-layer transcoder + attribution graph là hướng đang được đầu tư nhiều nhất cho circuit trên model production.
- Việc đánh giá vẫn là bài toán mở chưa giải — đúng như §8 Limitations của SFC dự báo, và nay chính các benchmark kế thừa SFC cũng bị đặt câu hỏi.

---

## 5. Nguồn

**Đã mở trực tiếp:**
- https://arxiv.org/abs/2403.19647 — paper gốc
- https://features.baulab.info/ — trang dự án
- https://transformer-circuits.pub/2025/attribution-graphs/methods.html — Circuit Tracing
- https://www.lesswrong.com/posts/4uXCAJNuPKtKBsi28/negative-results-for-saes-on-downstream-tasks — DeepMind negative results
- https://www.neuronpedia.org/sae-bench/info — SAEBench
- https://arxiv.org/abs/2411.18895 — SCR/TPP
- https://arxiv.org/abs/2605.18229 — benchmark reliability
- https://arxiv.org/abs/2506.23845 — position paper phản biện

**Chỉ qua kết quả tìm kiếm (nên kiểm lại trước khi trích số cụ thể):**
- https://arxiv.org/abs/2502.16681 — Are SAEs Useful? Sparse probing
- https://arxiv.org/abs/2409.14507 — feature absorption
- https://arxiv.org/abs/2504.13756 — scaling SFC cho ICL
- https://arxiv.org/abs/2511.13653 — weight-sparse transformers
- https://arxiv.org/abs/2503.09532 — SAEBench paper
- https://arxiv.org/abs/2501.18823 — transcoders beat SAEs
- https://proceedings.neurips.cc/paper_files/paper/2024/file/2b8f4db0464cc5b6e9d5e6bea4b9f308-Paper-Conference.pdf — Dunefsky transcoders
- https://www.neuronpedia.org/graph/info — Circuits Research Landscape 08/2025
