# Dàn ý seminar + kịch bản nói + câu hỏi dự phòng

Đề xuất cho bài 45 phút (35 trình bày + 10 hỏi đáp). Cột "Nguồn" chỉ tới file chứa số liệu.

---

## Cấu trúc đề xuất

| # | Slide | Thời lượng | Nguồn |
|---|---|---|---|
| 1 | Tiêu đề, tác giả, ICLR 2025 Oral | 0:30 | 01 §1 |
| 2 | Câu hỏi mở đầu: vì sao không thể gỡ bias bằng cách "tắt neuron" | 1:30 | 03 §1 |
| **Phần I — Vấn đề (6 phút)** | | | |
| 3 | Hai dòng nghiên cứu trước và chỗ hỏng của mỗi dòng | 2:00 | 01 §2 |
| 4 | Superposition: vì sao neuron polysemantic | 2:00 | 03 §1 |
| 5 | Hai thách thức phải giải cùng lúc | 1:00 | 01 §2 |
| 6 | Dòng thời gian: hai nhánh hội tụ ở 03/2024 | 1:30 | 02 §1, §5 |
| **Phần II — Phương pháp (10 phút)** | | | |
| 7 | SAE: công thức phân tách, `f_i`, `v_i`, `ε` | 2:00 | 01 §3.1 |
| 8 | Quyết định thiết kế: giữ `ε` làm node | 1:30 | 01 §3.1, 03 §2 |
| 9 | Indirect effect và `do(·)` | 1:30 | 03 §3 |
| 10 | Attribution patching: 2 fwd + 1 bwd cho toàn bộ node | 1:30 | 06 Bảng 7 |
| 11 | Integrated gradients và chỗ attribution patching hỏng | 1:30 | 01 §3.2, ảnh trang 36 |
| 12 | Pipeline 4 bước (dùng lại Figure 2, trang 4) | 2:00 | 01 §3.3, ảnh trang 4 |
| 13 | *(tuỳ chọn)* Ba thủ thuật: stop-gradient, pass-through, JVP | 1:30 | 01 §3.4 |
| **Phần III — Thí nghiệm 1: agreement (6 phút)** | | | |
| 14 | Task và 4 cấu trúc cú pháp | 1:00 | 01 §4 |
| 15 | Ba tiêu chí: interpretability / faithfulness / completeness | 1:30 | 03 §4 |
| 16 | Kết quả: 100 vs 1 500 node; 500 vs 50 000 node | 1:30 | 06 Bảng 2 |
| 17 | Case study: hai đường xử lý + NP number tracker (Figure 4, trang 7) | 2:00 | 01 §4, ảnh trang 7 |
| **Phần IV — SHIFT (10 phút)** | | | |
| 18 | Vấn đề: khi giới tính dự báo nhãn 100% | 2:00 | 01 §5, 03 §5 |
| 19 | SHIFT: 4 bước | 1:30 | 01 §5 |
| 20 | Thiết lập BiB: ambiguous set vs balanced set | 1:30 | 01 §5 |
| 21 | Bảng kết quả, chỉ ra 4 phép so sánh | 3:00 | 06 Bảng 4 |
| 22 | Vì sao SHIFT với neuron bị bỏ hẳn | 1:00 | 01 §5 |
| 23 | Đối chiếu với literature spurious correlation | 1:00 | 06 Bảng 3 |
| **Phần V — Quy mô lớn (4 phút)** | | | |
| 24 | Clustering để phát hiện hành vi (Michaud 2023) | 1:30 | 03 §7 |
| 25 | Hai ví dụ cluster (Figure 5, trang 9) | 2:00 | 01 §6, ảnh trang 9 |
| 26 | Kết quả: tìm lại induction và succession một cách tự động | 0:30 | 01 §6 |
| **Phần VI — Đánh giá và hiện tại (6 phút)** | | | |
| 27 | Limitations do tác giả nêu | 1:00 | 01 §7 |
| 28 | Ba đợt phê bình SAE 2024–2026 | 2:00 | 05 §2 |
| 29 | Vòng phản hồi: SHIFT → SCR → SAEBench → SCR bị bác | 1:00 | 05 §1.1, §2 |
| 30 | Hướng kế nhiệm: attribution graphs, weight-sparse transformers | 1:30 | 05 §3, 06 Bảng 8 |
| 31 | Đánh giá tổng kết + câu hỏi thảo luận | 0:30 | dưới đây |

---

## Kịch bản nói cho các slide khó

### Slide 2 — mở bài
Giả sử có một classifier đọc tiểu sử để đoán nghề nghiệp, và nó đang đoán theo giới tính. Bạn muốn gỡ. Nhưng dữ liệu train của bạn chỉ có nam giáo sư và nữ y tá — giới tính dự báo nhãn đúng 100%. Không có một mẫu nào để nói cho model biết nó đang sai. Mọi phương pháp debias tiêu chuẩn đều dừng ở đây, vì chúng đều cần dữ liệu trong đó tín hiệu spurious dự báo kém hơn tín hiệu đích. Paper này lấy tín hiệu phân định từ chỗ khác: từ việc một người mở model ra, đọc xem bên trong có gì, và chỉ vào thứ không nên có.

### Slide 4 — superposition
Model có 512 chiều trong residual stream nhưng cần biểu diễn số concept lớn hơn nhiều. Nó giải quyết bằng cách gán mỗi concept một hướng, và các hướng đó không trực giao. Hệ quả: mỗi trục toạ độ — mỗi neuron — nhận đóng góp từ nhiều concept, nên kích hoạt trên các ngữ cảnh không liên quan gì nhau. Con số trong paper: crowdworker chấm neuron 32.6/100 về độ diễn giải được. Và đây là chỗ đáng chú ý — neuron có ảnh hưởng nhân quả **lớn nhất** lên task agreement chỉ được 30.2, tức còn khó hiểu hơn neuron lấy ngẫu nhiên. Không phải là chúng ta chưa cố đủ; đơn vị phân tích sai.

### Slide 8 — error node
Đây là quyết định thiết kế riêng của paper. SAE tái tạo activation có sai số; phần dư gọi là `ε`. Cách làm thông thường là vứt nó đi, và nhận lấy sai số cộng dồn qua từng layer. Paper giữ `ε` lại và đưa vào graph như một loại node riêng. Đánh đổi rõ ràng: error node là vector 512 hoặc 2304 chiều, không đọc được, phải vẽ bằng hình tam giác và gán nhãn theo vị trí. Đổi lại, circuit trung thực, và nếu error node có hiệu ứng lớn thì đó là **con số đo được** cho biết SAE còn thiếu bao nhiêu. Figure 3 cho thấy bỏ error node của residual stream là phá hỏng model — tức residual stream chứa nhiều thông tin mà SAE chưa mã hoá. Paper không giấu điều này, họ vẽ nó thành một đường riêng trên đồ thị.

### Slide 11 — vì sao cần integrated gradients
Attribution patching là khai triển Taylor bậc nhất, nên nó chỉ đúng khi metric gần tuyến tính quanh điểm gốc. Paper đo thật: Figure 25 vẽ ÎE xấp xỉ theo IE thật cho từng SAE feature và error, trên 30 input. Attribution patching bám khá sát — trừ MLP layer 0 và residual stream các layer đầu, ở đó nó ước lượng thấp. Đó đúng là chỗ activation biến động mạnh nhất giữa các token khác nhau, tức hiệu `a_patch − a_clean` lớn nhất, tức Taylor bậc nhất kém nhất. Integrated gradients lấy trung bình gradient trên 10 điểm dọc đường nội suy, nên không bị bẫy đó. Chi phí gấp 10 lần, và paper chọn trả.

### Slide 21 — đọc bảng kết quả
Đừng đọc bảng theo hàng. Đọc theo bốn phép so sánh.

Thứ nhất, SHIFT + retrain so với Oracle: 93.1 và 93.0 ở Pythia, 95.0 và 95.0 ở Gemma. Oracle được train trên nhãn thật của balanced set. SHIFT không thấy nhãn nào ngoài ambiguous set. Hai con số bằng nhau.

Thứ hai, Random ablation so với Original: 61.8 và 61.9. Ablate cùng số lượng feature nhưng chọn ngẫu nhiên thì không thay đổi gì. Vậy hiệu quả không đến từ việc ablate, mà từ việc chọn đúng cái để ablate.

Thứ ba, SHIFT so với Feature skyline ở worst-group Pythia: 76.0 và 62.9. Feature skyline được phép nhìn balanced set để chọn feature nào ảnh hưởng nhiều nhất tới accuracy giới tính. Người chú thích không được nhìn. Người thắng.

Thứ tư, Neuron skyline ở Gemma worst-group: 5.6, trong khi Original là 18.2. Ablate neuron — kể cả neuron được chọn bằng cách xem đáp án — làm mọi thứ tệ hơn lúc chưa làm gì. Lý do paper đưa ra: neuron dính líu nhiều nhất vào phân loại giới tính cũng chính là neuron cần cho phân loại nghề. Không tách được.

### Slide 29 — vòng phản hồi
Có một diễn biến đáng kể sau paper này. SHIFT được chính hai tác giả của nó (Rager và Marks) chuyển thành metric SCR để đánh giá SAE, cuối 2024. SCR vào SAEBench đầu 2025 và thành thước đo chuẩn của cộng đồng. Rồi tháng 5/2026 có một phân tích kết luận SCR và TPP ở cấu hình chuẩn trượt nhiều bài kiểm tra độ tin cậy và không nên dùng để đánh giá SAE. Đây không phải để chê paper. Nó minh hoạ đúng điều §8 Limitations đã nói: đánh giá dictionary và circuit khi không có task downstream rõ ràng là bài toán khó, và bài toán đó vẫn chưa được giải sau hai năm.

---

## Câu hỏi dự phòng

**Q: Vì sao dùng Pythia-70M, model bé như vậy có ý nghĩa gì?**
Vì §5 cần chạy circuit discovery hàng nghìn lần trên hàng nghìn cluster; ở model lớn hơn thì không kham nổi tại thời điểm 2024. Pythia cũng là suite mở hoàn toàn (mọi checkpoint, dữ liệu train công khai). Gemma-2-2B có mặt chính là để trả lời câu hỏi này — phương pháp giữ nguyên khi lên 26 layer, tuy số node cần tăng từ ~100 lên ~500. Năm 2025 Kharlapenko et al. đã đẩy tiếp lên Gemma-1 2B và sau đó Gemma 9B.

**Q: Faithfulness chỉ 0.21 ở case study — như vậy circuit giải thích được 21% thôi à?**
Đúng, và paper nói rõ vì sao: 0.21 là ngưỡng họ **cố ý** chọn để circuit đủ nhỏ cho việc chú thích thủ công (86 node ở Pythia, 223 ở Gemma). Figure 3 cho thấy nếu tăng số node thì faithfulness lên cao hơn nhiều. Đánh đổi là giữa "đọc được bằng tay" và "giải thích được nhiều".

**Q: Tại sao bỏ 1/3 circuit ở layer đầu khi đánh giá faithfulness? Nghe như chọn dữ liệu có lợi.**
Lý do paper đưa ra: component ở layer đầu gắn với **token cụ thể**, mà train split và test split không dùng chung token. Nên phần đầu circuit không đánh giá được trên held-out data. Đây là một lựa chọn phương pháp luận có thể chất vấn, và nếu người hỏi ép tiếp thì nên trả lời thẳng: paper không báo cáo faithfulness khi giữ đủ cả circuit, nên không kiểm chứng được con số đó từ paper.

**Q: SHIFT cần con người đọc 67 feature. Có scale được không?**
Không, ở dạng gốc thì không. Nhưng đúng vấn đề này đã được xử lý: Karvonen, Rager, Marks, Nanda (11/2024) thay người chú thích bằng một LLM, tạo thành metric SCR tự động. Cần nói thêm rằng độ tin cậy của SCR sau đó bị đặt câu hỏi (05/2026).

**Q: Nếu SAE có feature absorption thì circuit có còn đúng không?**
Đây là mối lo có thật. Feature absorption nghĩa là một latent trông đơn nghĩa nhưng không kích hoạt ở một phần ngữ cảnh mà đáng lẽ phải kích hoạt, vì phần đó bị latent khác hấp thụ. Hệ quả cho SFC: nhãn con người gán ở bước 2 của SHIFT có thể sai, và circuit có thể bỏ sót đường đi. Paper không biết vấn đề này ở thời điểm viết (Chanin et al. ra 09/2024). Cần nói rõ nó là giới hạn của **công cụ nền**, không phải lỗi phương pháp SFC.

**Q: Vì sao không dùng LEACE hoặc INLP cho bài toán BiB?**
Vì cả hai đều cần **nhãn của thuộc tính cần xoá**. Trong ambiguous set, nhãn giới tính trùng hoàn toàn với nhãn nghề nghiệp. Chiếu bỏ hướng giới tính cũng chính là chiếu bỏ hướng nghề nghiệp. Setting này được thiết kế để loại chính xác nhóm phương pháp đó.

**Q: SHIFT có khác gì so với chỉ prompt model "đừng dùng giới tính"?**
Khác ở chỗ SHIFT can thiệp vào trọng số/activation, không dựa vào việc model có nghe lời hay không. Nhưng đây là một baseline đáng lẽ nên có — Kantamneni et al. (2025) chính là chỉ ra rằng naive prompting là baseline mạnh mà nhiều paper SAE bỏ qua. Nếu bị hỏi, nên thừa nhận paper không có baseline này.

**Q: Attribution graphs của Anthropic có thay thế SFC không?**
Về ý tưởng thì cùng hướng, nhưng khác ba điểm. Đơn vị: cross-layer transcoder thay cho SAE một layer. Phạm vi: attribution graph giải thích **một prompt cụ thể**, SFC giải thích hành vi trung bình trên một dataset. Xử lý phần dư: attribution graph coi error node là node đầu vào, không có cạnh vào, tức không truy ngược qua nó được — SFC thì error node có cả cạnh vào và ra. Bản methods của Anthropic không trích Marks et al., nên nên gọi là hướng kế nhiệm về ý tưởng, không nói "xây trên".

**Q: SAE đang bị chê nhiều, vậy paper này còn giá trị gì?**
Các negative result đều đo SAE ở việc **hành động trên concept đã biết** — probing một concept đã đặt tên, xoá một thuộc tính đã đặt tên. Ở đó baseline có giám sát mạnh hơn, không có gì lạ. Chỗ SFC làm khác là **phát hiện concept chưa biết**: §4 con người tìm ra feature giới tính mà không được cho biết trước giới tính là vấn đề; §5 tìm ra hàng nghìn hành vi không ai soạn dữ liệu cho. Đây đúng là lập luận của position paper 2506.23845. Ngay cả nhóm DeepMind khi giảm ưu tiên SAE cũng ghi nhận SAE còn hữu ích cho debug thăm dò và phát hiện tương quan giả trong dataset — hai việc SFC làm.

**Q: Ngưỡng `T_N`, `T_E` chọn thế nào?**
Bằng tay, tuỳ mục đích. Ở §3.2 paper quét `T_N` để vẽ đường faithfulness. Ở §3.3 họ chỉnh `T_N` để circuit đủ nhỏ cho việc chú thích mà faithfulness vẫn > 0.2. Các hình circuit đầy đủ ghi rõ giá trị, ví dụ `T_N = 0.5, T_E = 0.05` cho Gemma simple agreement. Đây là một hyperparameter tuỳ ý, không có tiêu chí khách quan — đáng nêu như một giới hạn.

**Q: BOS feature là gì và vì sao phải loại?**
Trong Gemma Scope có những feature chỉ kích hoạt trên token BOS, top logit không mang thông tin, nhưng indirect effect lên output lại lớn. Vì không diễn giải được, paper loại chúng khỏi việc chú thích, khỏi SHIFT và khỏi feature skyline. Đây là một lựa chọn cần thiết nhưng cũng là một vùng của model bị bỏ ngoài phân tích.

---

## Ba câu chốt đáng để lại cuối bài

1. Đóng góp về phương pháp không phải là "dùng SAE" hay "dùng attribution patching" — cả hai đã có. Là **ghép chúng lại và giữ lại phần dư `ε` làm node**, để circuit vừa đọc được vừa trung thực, và đo được chính xác phần chưa giải thích được là bao nhiêu.

2. §4 là phần trả lời được câu hỏi khó nhất của interpretability: **circuit này dùng được vào việc gì?** Câu trả lời là một con số — 93.1 so với Oracle 93.0 — chứ không phải một hình vẽ đẹp.

3. Hai năm sau, SAE đã mất vị trí trung tâm, nhưng bài toán mà SFC đặt ra thì không: **làm sao gỡ một tín hiệu không mong muốn ra khỏi model khi dữ liệu không nói cho bạn biết tín hiệu đó sai**. Chưa có phương pháp nào khác trả lời được câu này mà không cần dữ liệu phân định.
