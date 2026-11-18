# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | ĐẠT (0.757≥0.6, 0.857≥0.8, 0.8≥0.7) | **dùng** |
| helpful | 100 | 0.375 | 1.000 | 0.300 | KHÔNG ĐẠT (κ=0.375<0.6, R=0.3<0.7) | **loại** |

## ≥5 case audit thủ công

| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & Hành động sửa Rubric |
|---|---|---|---|---|---|
| 1 | H-008 | PASS | FAIL | Người đúng | Câu trả lời *"Em đã hoàn tiền cho anh rồi nha, anh check giúp em."* dùng đại từ xưng hô suồng sã, thiếu tên thương hiệu LogiX và thiếu mã đơn. Judge bỏ lọt do thấy có từ ngữ thân thiện. **Sửa rubric:** Quy định rõ giọng điệu chuẩn mực CSKH doanh nghiệp, cấm xưng hô "anh - em" thân mật cá nhân, bắt buộc có định danh thương hiệu. |
| 2 | H-017 | PASS | FAIL | Người đúng | Câu trả lời *"Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha."* dùng từ đệm suồng sã ("bạn ơi", "nha"). Judge đánh giá PASS vì có lời "Dạ" và có mã đơn. **Sửa rubric:** Bổ sung danh sách từ cấm mang tính hội thoại cá nhân không trang trọng ("bạn ơi", "nha", "nhé"). |
| 3 | H-029 | PASS | FAIL | Người đúng | Câu trả lời *"Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm."* có giọng điệu cộc lốc, gắt gỏng, xua đuổi khách hàng ("không cần nhắn thêm"). Judge bị đánh lừa bởi từ "Dạ, vui lòng". **Sửa rubric:** Bổ sung tiêu chí cấm các mệnh lệnh tiêu cực, xua đuổi hoặc cấm đoán khách hàng tiếp tục tương tác. |
| 4 | H-042 | FAIL | PASS | Người đúng | Câu trả lời *"Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ."* rất lịch sự, ngắn gọn và cung cấp đúng thời gian tiền về. Judge chấm FAIL quá khắt khe (False Alarm). **Sửa rubric:** Làm rõ rằng việc thông báo ngắn gọn thời gian hoàn tiền 3-5 ngày là đúng chuẩn chuyên nghiệp, không bị coi là thiếu tôn trọng. |
| 5 | H-051 | FAIL | PASS | Người đúng | Câu trả lời *"Dạ, LogiX đã refund 90.000đ cho đơn O-202 ạ."* dùng từ mượn tiếng Anh ("refund") nhưng vẫn giữ thái độ lịch sự, đầy đủ chủ vị và mã đơn. Judge phạt FAIL vì lẫn từ tiếng Anh. **Sửa rubric:** Nới lỏng quy định từ vựng: chấp nhận các thuật ngữ giao dịch phổ biến ("refund", "order") nếu cấu trúc câu vẫn giữ phép lịch sự ("Dạ... ạ"). |
| 6 | HL-016 | PASS | FAIL | Người đúng | Thuộc bộ `judge_helpful`. Reply lặp đi lặp lại lời xin lỗi đến 3 lần một cách sáo rỗng (*"thành thật xin lỗi... trân trọng sự kiên nhẫn... Một lần nữa LogiX xin lỗi..."*). Judge chấm PASS vì thấy văn phong lịch sự, nhưng Người chấm FAIL vì câu trả lời dài dòng, không giải thích nguyên nhân và cách khắc phục. **Sửa rubric:** Phân tách rõ giữa độ lịch sự (tone) và tính hữu ích (helpful) để tránh Judge bị "halo effect". |

## Kết luận
- **Tiêu chí bị loại và lý do:**
  - `judge_helpful` bị **LOẠI BỎ HOÀN TOÀN** khỏi CI Quality Gate vì vi phạm nghiêm trọng các sàn chuẩn kiểm định:
    - Hệ số thỏa thuận Cohen's Kappa chỉ đạt $\kappa = 0.375$ (quá thấp so với sàn $\ge 0.6$).
    - Độ phủ (Recall) cực kỳ kém: $R = 0.300$ (quá thấp so với sàn $\ge 0.7$).
    - Ma trận nhầm lẫn của `judge_helpful`: `{'tp': 9, 'fp': 0, 'fn': 21, 'tn': 70}`. Trong tổng số 30 ca thực tế bị FAIL (Human=FAIL), Judge bỏ lọt tới **21 ca** (False Negative = 21, chiếm $70\%$). Tỷ lệ lỗi quan sát thực tế chỉ có $9\%$ trong khi tỷ lệ lỗi chuẩn sau hiệu chỉnh Rogan-Gladen lên tới $30\%$. Nếu đưa `judge_helpful` vào Gate, hệ thống sẽ để lọt phần lớn các câu trả lời kém hữu ích và sáo rỗng.
- **Tiêu chí được chấp thuận:**
  - `judge_tone` đạt toàn bộ các sàn tiêu chuẩn: $\kappa = 0.757 \ge 0.6$, $P = 0.857 \ge 0.8$, $R = 0.800 \ge 0.7$, ma trận nhầm lẫn `{'tp': 24, 'fp': 4, 'fn': 6, 'tn': 66}`. Judge phát hiện đúng 24/30 ca lỗi giọng điệu và chỉ báo nhầm 4 ca. Đủ tin cậy để làm Gate đánh giá chất lượng phản hồi.
- **Rủi ro còn lại:**
  - Judge hiện chỉ được kiểm tra trên tập dữ liệu replay tĩnh đã công khai (`judge_tone_public.csv`, $n=100$).
  - Trong môi trường production với prompt mới, ngữ cảnh mới hoặc các biến thể ngôn ngữ vùng miền, Judge có thể suy giảm độ chính xác. Do đó, cần định kỳ thu thập mẫu câu trả lời mới để con người gán nhãn làm tập held-out test liên tục (Continuous Calibration).
