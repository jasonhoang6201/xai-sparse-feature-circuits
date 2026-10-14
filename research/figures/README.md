# Hình từ paper — bản render 150 dpi của từng trang

Render bằng `pdftoppm -r 150 -png`. 26 file, gồm các trang 1–10, 19, 22–36 của PDF. Mỗi file là **một trang đầy đủ**, cần crop phần hình trước khi đưa lên slide.

| File | Hình trên trang | Nội dung | Dùng cho slide |
|---|---|---|---|
| `page-02.png` | Figure 1 | Tổng quan 3 loại dữ liệu vào → circuit → diễn giải → debug | Slide mở phần phương pháp |
| `page-04.png` | Figure 2 | **Pipeline 4 bước**: cache → backprop → lọc node → lọc edge | Slide 12 — hình quan trọng nhất của paper |
| `page-05.png` | Figure 3 + Bảng 1 | Faithfulness và completeness, feature vs neuron, 2 model | Slide 16 |
| `page-07.png` | Figure 4 | **Tóm tắt circuit across-RC** cho Pythia (a) và Gemma (b) | Slide 17 |
| `page-08.png` | Bảng 2 | Kết quả SHIFT trên Bias in Bios | Slide 21 |
| `page-09.png` | Figure 5 | Cluster 382 (dãy tăng) và cluster 475 ("to" nguyên mẫu) | Slide 25 |
| `page-19.png` | Figure 6 | Gộp effect qua vị trí token: templatic vs non-templatic | Slide phụ |
| `page-23.png` | Figure 7 | Circuit đầy đủ across-RC, Pythia-70M | Slide phụ / backup |
| `page-24.png` | Figure 8 | Circuit đầy đủ across-RC, Gemma-2-2B | Slide phụ / backup |
| `page-25.png` | Figure 9, 10 | Circuit across-PP, cả hai model | Backup — dùng khi nói về overlap giữa RC và PP |
| `page-26.png` | Figure 11 | Circuit simple agreement, Pythia | Backup |
| `page-27.png` | Figure 12 | Circuit simple agreement, Gemma (`T_N=0.5, T_E=0.05`) | Backup |
| `page-28.png` | Figure 13 | Circuit within-RC, Pythia (`T_N=0.2, T_E=0.02`) | Backup |
| `page-29.png` | Figure 14, 15 | Circuit within-RC Gemma; **Figure 15 = ví dụ NP number tracker** (`resid 12/13561`) | Figure 15 dùng cho slide 17 |
| `page-30.png` | Figure 16, 17 | Circuit Bias in Bios có chú thích; circuit dãy số tăng | Slide 20 hoặc 25 |
| `page-31.png` | Figure 18, 19 | Circuit "to" nguyên mẫu; **Figure 19 = ví dụ feature giới tính bị ablate** (`attn 3/22029`) | Figure 19 dùng cho slide 20 — minh hoạ cụ thể cái người chú thích nhìn thấy |
| `page-32.png` | Figure 21, 22 | **Ví dụ neuron** từ Bias in Bios — không đọc được | Slide 22 — đặt cạnh Figure 19 để so sánh trực tiếp |
| `page-33.png` | Figure 23 | Giao diện chú thích của crowdworker | Backup nếu bị hỏi về Bảng 7 |
| `page-34.png` | Figure 24 | Giao diện chú thích (tiếp) | Backup |
| `page-36.png` | Figure 25 | **Attribution patching vs Integrated gradients** — ÎE xấp xỉ theo IE thật | Slide 11 |
| `page-01.png`, `page-03.png`, `page-06.png`, `page-10.png`, `page-22.png`, `page-35.png` | — | Trang chữ, không có hình | — |

## Cặp hình nên đặt cạnh nhau

- **`page-31.png` (Figure 19) vs `page-32.png` (Figure 21/22)**: một SAE feature đọc được, đặt cạnh một neuron không đọc được, cả hai đều có indirect effect cao trên Bias in Bios. Đây là bằng chứng trực quan mạnh nhất cho luận điểm chính của paper, mạnh hơn Bảng 7.
- **`page-07.png` (Figure 4a) vs (Figure 4b)**: Pythia dùng 2 đường, Gemma dùng 2 đường + NP number tracker. Cho thấy model lớn hơn dùng cơ chế phong phú hơn cho cùng một task.

## Nếu cần độ phân giải cao hơn

```bash
pdftoppm -r 300 -png -f <trang> -l <trang> ../../xai.pdf out
```

Các hình circuit (Figure 7–14, 16–18) là đồ thị dày đặc, ở 150 dpi sẽ không đọc được nhãn. Nên render 300 dpi và crop, hoặc chỉ dùng phần tóm tắt Figure 4.
