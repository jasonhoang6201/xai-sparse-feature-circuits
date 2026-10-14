# Hồ sơ nghiên cứu — Sparse Feature Circuits (ICLR 2025)

Tài liệu chuẩn bị cho agent làm slide seminar. Paper gốc: `../xai.pdf`, 36 trang.

> Ghi chú tên file: `xai.pdf` = *explainable AI*, không liên quan công ty xAI. Paper là **Sparse Feature Circuits: Discovering and Editing Interpretable Causal Graphs in Language Models**, Marks, Rager, Michaud, Belinkov, Bau, Mueller — ICLR 2025 Oral, arXiv 2403.19647.

## Các file

| File | Nội dung | Dùng khi nào |
|---|---|---|
| **`01-paper-tom-tat.md`** | Toàn bộ paper: problem, solution, 3 thí nghiệm, mọi con số, mọi công thức, limitations | Nguồn sự thật chính. Mọi số liệu trên slide phải đối chiếu file này |
| **`02-boi-canh-lich-su.md`** | Dòng thời gian 3 nhánh nghiên cứu hội tụ vào SFC; vì sao SFC xuất hiện đúng 03/2024 | Slide mở bài, slide bối cảnh |
| **`03-nen-tang-ky-thuat.md`** | Khái niệm nền phải giải thích trước: superposition, SAE và các biến thể, activation patching, faithfulness, spurious correlation, clustering. Kèm bảng thuật ngữ Anh–Việt | Slide nền; tra cứu khi cần diễn đạt một khái niệm |
| **`04-paper-lien-quan.md`** | Thư mục có chú giải, chia 7 nhóm theo vai trò với SFC, đánh dấu ⭐/◆/○ theo độ ưu tiên | Chọn paper nào cần nhắc; slide related work |
| **`05-sau-paper-va-tranh-luan.md`** | Công trình kế thừa 2024–2026, ba đợt phê bình SAE, hướng thay thế, trạng thái hiện tại | Phần cuối seminar — phần làm bài nói khác biệt so với chỉ đọc paper |
| **`06-so-sanh-doi-chieu.md`** | 9 bảng đối chiếu, mỗi bảng dùng được trực tiếp làm một slide | Copy thẳng lên slide |
| **`07-dan-y-slide.md`** | Dàn ý 31 slide có phân bổ thời gian, kịch bản nói cho 5 slide khó, 11 câu hỏi dự phòng, 3 câu chốt | Khung xây deck |
| **`figures/`** | 26 trang PDF đã render PNG 150 dpi + `README.md` ánh xạ trang ↔ hình ↔ slide | Lấy hình minh hoạ |

## Đọc theo thứ tự nào

Nếu chỉ đọc một file: `07-dan-y-slide.md` (có dàn ý và trỏ ngược tới mọi file khác).
Nếu đọc ba: `01` → `07` → `06`.
Nếu làm deck đầy đủ: `01` → `03` → `02` → `06` → `05` → `07` → `figures/README.md`.

## Ba luận điểm của paper, nói gọn

1. **Đơn vị phân tích sai thì mọi thứ sau đó sai.** Neuron polysemantic do superposition. Crowdworker chấm neuron 32.6/100, SAE feature 52.8–81.5/100. Đáng chú ý hơn: neuron có ảnh hưởng nhân quả lớn nhất (30.2) còn khó hiểu hơn neuron ngẫu nhiên (32.6); với feature thì ngược lại.
2. **Có thể tìm circuit trên feature với chi phí không đổi theo số node**, nhờ attribution patching (2 forward + 1 backward) và integrated gradients cho các layer đầu. Kết quả: 100 node feature thay cho 1 500 neuron ở Pythia-70M; 500 thay cho 50 000 ở Gemma-2-2B.
3. **Circuit đó dùng được vào việc đo được bằng số.** SHIFT gỡ phụ thuộc giới tính khỏi classifier trong tình huống giới tính dự báo nhãn 100%, đạt 93.1 so với Oracle 93.0 — mà không dùng thêm nhãn nào.

## Bốn con số phải nhớ

| | |
|---|---|
| 100 vs 1 500 | số node feature vs neuron để giải thích phần lớn hành vi, Pythia-70M |
| 500 vs 50 000 | như trên, Gemma-2-2B |
| 93.1 vs 93.0 | SHIFT + retrain vs Oracle, profession accuracy, Pythia |
| 61.8 vs 61.9 | Random ablation vs Original — ablate bừa không có tác dụng |

## Ba điều làm bài nói khác biệt so với đọc paper

1. **Vòng phản hồi khép kín**: SHIFT (2024) → metric SCR do chính tác giả tạo (11/2024) → vào SAEBench (03/2025) → SCR bị kết luận không đáng tin (05/2026). Xem `05` §1.1 và §2.
2. **Bối cảnh SAE mất vị trí trung tâm** trong 2025–2026, và vì sao §4/§5 của SFC vẫn đứng vững trước phê bình đó (chúng làm việc *phát hiện concept chưa biết*, không phải *hành động trên concept đã biết*). Xem `05` §2.
3. **Hướng kế nhiệm**: attribution graphs với cross-layer transcoder (Anthropic, 03/2025) và weight-sparse transformers (OpenAI, 11/2025) — hai câu trả lời khác nhau cho cùng vấn đề. Xem `05` §3 và `06` Bảng 8.

## Lưu ý về độ tin cậy

- File `01`, `03`, `04`, `06` (phần số liệu paper): **trích trực tiếp từ PDF**, đã đối chiếu.
- File `02` và `05`: thu thập từ web tháng 09/2026. Mỗi mục trong `05` có đánh dấu **[đã đọc nguồn]** hoặc **[từ kết quả tìm kiếm]**. Chỉ đưa số liệu cụ thể lên slide từ các mục đã đọc nguồn.
- Trong `02`, hai mục **không** nằm trong reference list của paper nhưng cần cho phần nền: Olah et al. 2020 *Zoom In* và Elhage et al. 2021 *A Mathematical Framework for Transformer Circuits*. Đã ghi chú rõ trong file.
