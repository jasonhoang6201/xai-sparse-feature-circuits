# Paper liên quan — thư mục có chú giải

Nhóm theo vai trò đối với SFC. Cột "Vai trò" nói rõ paper đó **được SFC dùng làm gì**, để chọn slide nhanh.

Ký hiệu: ⭐ = bắt buộc nhắc trong seminar; ◆ = nên có nếu còn thời gian; ○ = tra cứu khi bị hỏi.

---

## A. Paper mà SFC xây trực tiếp lên trên

| | Paper | Vai trò với SFC |
|---|---|---|
| ⭐ | **Elhage et al. 2022**, Toy Models of Superposition. Transformer Circuits Thread | Lý do tồn tại của SAE. Nguồn của từ "polysemantic" mà SFC dùng để chê circuit trên neuron/head |
| ⭐ | **Bricken et al. 2023**, Towards Monosemanticity: Decomposing Language Models With Dictionary Learning. Anthropic | SFC copy nguyên kiến trúc SAE (ReLU, tied pre-encoder bias, `d_SAE = 64×d`) và thủ tục resample dead feature từ đây |
| ⭐ | **Cunningham, Ewart, Riggs, Huben, Sharkey 2024** (ICLR), Sparse Autoencoders Find Highly Interpretable Features in LMs | Bản độc lập từ giới học thuật. Logan Riggs sau đó giúp nhóm SFC train SAE |
| ⭐ | **Lieberum et al. 2024**, Gemma Scope: Open Sparse Autoencoders Everywhere All At Once on Gemma 2 | Nguồn SAE cho toàn bộ phần Gemma-2-2B của SFC. JumpReLU, width 16384, chọn bản L0 ≈ 100 |
| ⭐ | **Nanda 2022**, Attribution Patching: Activation Patching At Industrial Scale (blog) | Eq. 3 của SFC |
| ⭐ | **Sundararajan, Taly, Yan 2017**, Axiomatic Attribution for Deep Networks (ICML) | Integrated gradients — Eq. 4 của SFC, N = 10 |
| ⭐ | **Pearl 2001**, Direct and Indirect Effects (UAI) | Định nghĩa indirect effect — Eq. 2 |
| ⭐ | **Finlayson, Mueller, Gehrmann, Shieber, Linzen, Belinkov 2021** (ACL), Causal Analysis of Syntactic Agreement Mechanisms in Neural LMs | **Nguồn dữ liệu §3.** Cũng là nơi khẳng định model xử lý simple agreement khác agreement qua RC/PP — SFC xác nhận lại ở mức feature |
| ⭐ | **De-Arteaga et al. 2019** (FAT*), Bias in Bios | **Dataset của §4.** Nghiên cứu gốc về thiên kiến biểu diễn ngữ nghĩa trong bối cảnh rủi ro cao |
| ⭐ | **Michaud, Liu, Girit, Tegmark 2023** (NeurIPS), The Quantization Model of Neural Scaling | **Phương pháp clustering của §5.** Eric Michaud là đồng tác giả cả hai |
| ◆ | **Syed, Rager, Conmy 2023** (NeurIPS workshop), Attribution Patching Outperforms Automated Circuit Discovery | EAP — attribution patching áp lên edge. Can Rager là đồng tác giả cả hai |
| ◆ | **Hanna, Pezzelle, Belinkov 2024** (ICML workshop), Have Faith in Faithfulness (EAP-IG) | Ghép IG vào EAP; lập luận đánh giá circuit bằng faithfulness thay vì circuit overlap |
| ◆ | **Kramár, Lieberum, Shah, Nanda 2024**, AtP\*: An Efficient and Scalable Method for Localizing LLM Behaviour to Components | Phân tích khi nào attribution patching sai — SFC trích để biện minh cho việc dùng IG ở layer đầu |
| ◆ | **Wang, Variengien, Conmy, Shlegeris, Steinhardt 2023** (ICLR), Interpretability in the Wild: IOI circuit in GPT-2 small | Nguồn của cách mean-ablate theo từng vị trí token |
| ◆ | **Gandelsman, Efros, Steinhardt 2024**, Interpreting CLIP's Image Representation via Text-Based Decomposition | **Cảm hứng trực tiếp cho SHIFT**: diễn giải rồi ablate attention head không mong muốn trong CLIP |
| ◆ | **Yan et al. 2023**, Robust and Interpretable Medical Image Classifiers via Concept Bottleneck Models | Baseline CBP của SFC được phỏng theo bài này |
| ○ | **Biderman et al. 2023** (ICML), Pythia | Model chính của SFC |
| ○ | **Gemma Team 2024**, Gemma 2 | Model thứ hai |
| ○ | **Gao et al. 2020**, The Pile | Corpus train SAE và corpus của §5 |
| ○ | **Lin & Bloom 2023**, Neuronpedia | Giao diện để con người đọc feature ở bước 2 của SHIFT |
| ○ | **Rajamanoharan et al. 2024b**, JumpReLU SAE | Kiến trúc encoder của Gemma Scope |
| ○ | **Wang et al. 2023** / **Nanda 2023**, Open source replication of Anthropic dictionary learning | Thủ tục buffer activation khi train SAE |

---

## B. Circuit analysis — bối cảnh mà SFC định vị mình đối lập

| | Paper | Vì sao nhắc |
|---|---|---|
| ⭐ | **Olsson et al. 2022**, In-context Learning and Induction Heads | Ví dụ điển hình của circuit trên component thô. §5 của SFC tìm lại induction ở mức feature một cách tự động |
| ⭐ | **Meng, Bau, Andonian, Belinkov 2022**, Locating and Editing Factual Associations in GPT (ROME) | Causal tracing trên MLP. David Bau đồng tác giả cả hai. Dùng để nêu "đây là cách làm trước đây" |
| ⭐ | **Conmy et al. 2023** (NeurIPS), Towards Automated Circuit Discovery (ACDC) | Baseline khái niệm: tự động nhưng chạy patching từng edge nên đắt, và vẫn trên component thô |
| ◆ | **Geva, Bastings, Filippova, Globerson 2023** (EMNLP), Dissecting Recall of Factual Associations | Cùng nhóm bằng chứng "MLP lưu fact" |
| ◆ | **Hanna, Liu, Variengien 2023** (NeurIPS), How does GPT-2 compute greater-than? | Circuit thủ công cho tác vụ số |
| ◆ | **Gould, Ong, Ogden, Conmy 2023**, Successor Heads | §5 của SFC tìm lại succession ở mức feature (cluster 382) |
| ◆ | **Geiger, Lu, Icard, Potts 2021**; **Geiger et al. 2022**; **Geiger, Potts, Icard 2023** | Causal abstraction: cho trước một causal graph giả định, tìm cơ chế khớp trong LM. SFC định vị ngược lại: tìm cơ chế **không cần** giả thuyết trước |
| ◆ | **Mueller et al. 2024**, The Quest for the Right Mediator: A History, Survey, and Theoretical Grounding of Causal Interpretability | Survey do chính tác giả SFC viết. Bài nền tốt nhất để đọc trước |
| ○ | **Vig et al. 2020** (NeurIPS), Investigating Gender Bias in LMs Using Causal Mediation Analysis | Tiền thân trực tiếp của §4: causal mediation + gender bias |
| ○ | **Nanda, Rajamanoharan, Kramár, Shah 2023**, Fact Finding: Reverse-Engineering Factual Recall on the Neuron Level | Được trích ở đoạn mở đầu |
| ○ | **Prakash, Rott Shaham, Haklay, Belinkov, Bau 2024** (ICLR), Fine-tuning Enhances Existing Mechanisms | Circuit sau fine-tuning |
| ○ | **Todd, Li, Sen Sharma, Mueller, Wallace, Bau 2024** (ICLR), Function Vectors in LLMs | Hướng lân cận: một vector mã hoá cả một task |
| ○ | **Yu, Merullo, Pavlick 2023** (EMNLP), Characterizing Mechanisms for Factual Recall | |
| ○ | **Robins & Greenland 1992**, Identifiability and Exchangeability for Direct and Indirect Effects | Nền thống kê |
| ○ | **Lewis 1973**, Counterfactuals | Nền triết học |

---

## C. Đơn vị mịn nhưng cần giám sát — nhóm SFC phê bình

| | Paper | Phê bình của SFC |
|---|---|---|
| ⭐ | **Kim et al. 2018** (ICML), TCAV | Cần người định nghĩa concept và cung cấp ví dụ trước |
| ⭐ | **Belinkov 2022** (Computational Linguistics), Probing Classifiers: Promises, Shortcomings, and Advances | Probe cần giả thuyết có sẵn; đo tương quan không phải nhân quả |
| ◆ | **Zou et al. 2023**, Representation Engineering: A Top-Down Approach to AI Transparency | Cùng nhóm "cần giả thuyết trước" |

---

## D. Spurious correlation — literature mà SHIFT tách ra khỏi

Điểm chung của cả nhóm D: **đều cần disambiguating labeled data**. Đây là câu chốt của §4.

| | Paper | Hướng tiếp cận |
|---|---|---|
| ⭐ | **Sagawa, Koh, Hashimoto, Liang 2020** (ICLR), Distributionally Robust Neural Networks (Group DRO) | Tối ưu trực tiếp worst-group accuracy; cần nhãn nhóm |
| ⭐ | **Liu et al. 2021** (ICML), Just Train Twice (JTT) | Train lần 1, upweight mẫu sai, train lần 2; cần dữ liệu có mẫu mà spurious dự báo sai |
| ⭐ | **Ravfogel et al. 2020** (ACL), Null It Out (INLP) | Xoá concept bằng chiếu lặp lên nullspace; cần nhãn của thuộc tính cần xoá |
| ⭐ | **Belrose et al. 2023** (NeurIPS), LEACE: Perfect Linear Concept Erasure in Closed Form | Xoá concept tuyến tính, có nghiệm đóng; cần nhãn concept |
| ◆ | **Kirichenko, Izmailov, Wilson 2023**, Last Layer Re-training is Sufficient for Robustness to Spurious Correlations (DFR) | Chỉ train lại layer cuối trên dữ liệu cân bằng — đối chiếu trực tiếp với bước 4 của SHIFT |
| ◆ | **Zech et al. 2018** (PLOS Medicine), Variable generalization performance of a deep learning model to detect pneumonia in chest radiographs | **Ví dụ thực tế SFC dùng để biện minh cho setting của mình**: input của các class đến từ bệnh viện khác nhau, không có dữ liệu phân định |
| ◆ | **Oikarinen, Das, Nguyen, Weng 2023** (ICLR), Label-free Concept Bottleneck Models | Một trong số ít phương pháp **không** cần disambiguating data — nhưng cho model ảnh/đa phương thức |
| ○ | Nam et al. 2020 (LfF); Nam et al. 2022 (Spread Spurious Attribute) | Reweighting tự động |
| ○ | Utama, Moosavi, Gurevych 2020 | Debias NLU khỏi bias chưa biết |
| ○ | Creager, Jacobsen, Zemel 2021 (EIIL); Idrissi et al. 2022; Sohoni et al. 2022 (BARACK) | Suy ra nhóm / cân bằng dữ liệu |
| ○ | Orgad & Belinkov 2023 (BLIND); Yaghoobzadeh et al. 2021 | Bỏ bias không cần nhãn nhân khẩu học |
| ○ | Zhang et al. 2021; Zhang et al. 2022 (Correct-N-Contrast); Oren et al. 2019 | DRO và contrastive |
| ○ | Ravfogel et al. 2022a, 2022b; Iskander et al. 2023, 2024; Wang et al. 2020 (Double-Hard Debias) | Concept erasure biến thể |

---

## E. AI safety — động lực của §4

| | Paper | Luận điểm |
|---|---|---|
| ⭐ | **Burns et al. 2023** (OpenAI), Weak-to-Strong Generalization | Khi model mạnh được giám sát bởi model yếu, "được chấp thuận" và "đúng" dự báo reward ngang nhau |
| ◆ | **Hase, Bansal, Clark, Wiegreffe 2024** (ACL), The Unreasonable Effectiveness of Easy Training Data for Hard Tasks | Easy-to-hard domain shift |
| ◆ | **Ngo, Chan, Mindermann 2024**, The Alignment Problem from a Deep Learning Perspective | Nêu lo ngại về misalignment trong đúng setting đó |
| ◆ | **Casper et al. 2023**, Open Problems and Fundamental Limitations of RLHF | Cùng lo ngại |
| ○ | **Christiano et al. 2023**, Deep RL from Human Preferences | RLHF gốc |

---

## F. Feature disentanglement — tiền sử xa

| | Paper | Ghi chú |
|---|---|---|
| ○ | Schmidhuber 1992, Learning Factorial Codes by Predictability Minimization | Tiền sử xa nhất trong reference list |
| ○ | Makhzani & Frey 2013, k-Sparse Autoencoders | Quay lại thành TopK SAE năm 2024 |
| ○ | Higgins et al. 2017 (β-VAE); Burgess et al. 2017; Kim & Mnih 2018; Chen et al. 2016, 2018 | Disentanglement trong thị giác |
| ○ | Desjardins, Courville, Bengio 2012; He et al. 2022; Peebles et al. 2020; Schneider & Vlachos 2021 | Các hướng khác |
| ○ | **Gao et al. 2024** (OpenAI), Scaling and Evaluating Sparse Autoencoders | TopK SAE — SFC trích ở đoạn "scaling SAE là out-of-scope" |
| ○ | **Rajamanoharan et al. 2024a**, Gated Sparse Autoencoders | |
| ○ | **Templeton et al. 2024** (Anthropic), Scaling Monosemanticity: Claude 3 Sonnet | Chứng minh SAE chạy trên model production |

---

## G. Ba paper nên đọc trước nếu chỉ có thời gian cho ba

1. **Elhage et al. 2022, Toy Models of Superposition** — để hiểu vì sao phải có SAE.
2. **Bricken et al. 2023, Towards Monosemanticity** — để hiểu SAE là gì và train thế nào.
3. **Mueller et al. 2024, The Quest for the Right Mediator** — survey do chính tác giả SFC viết, đặt SFC vào đúng vị trí trong causal interpretability.
