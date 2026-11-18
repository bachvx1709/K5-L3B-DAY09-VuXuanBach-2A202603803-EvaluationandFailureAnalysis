# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi bạn 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, run ID: `R-seeded_regression-C-002-s11` |
| Triệu chứng | Grader `trial_pass` fail (status `resolved` != `denied`); Ledger ghi nhận `refund_vnd=20000`, `refund_count=1` (kỳ vọng nhãn là 0đ và 0 count); Reply thông báo "Dạ, LogiX đã hoàn 20.000đ cho đơn O-202." |
| Bước sai đầu tiên | Event #3: kind `model_override`, payload: `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event #1 (`handoff`) khởi tạo đúng contract và allowed_tools. Event #2 (`investigator`) truy vấn database trung thực trả về `order_id="O-202"` với `delay_days=2` (chưa đủ điều kiện hoàn vì chính sách yêu cầu trễ $\ge 3$ ngày). Đến Event #3, agent tự ý lấy số ngày trễ 5 ngày từ ticket để ghi đè `delay_days=5`, dẫn đến bước #4 (`quote`) tính sai khoản tiền bồi thường 20.000đ. |
| Failure mode | `FM-03` (ticket_trust — tin ticket hơn DB) |
| Đối chứng | Cùng claim `C-002@11` trên shape `fixed`: Không có Event `model_override`. Event #3 là `quote` với `amount_vnd=0`, `reason="no_entitlement"`, và kết quả trả về `status="denied"`, `refund_vnd=0` (PASS 100% khớp nhãn). |
| Regression test | `tests/test_public_matrix.py::test_public_single_run_cases[swarm-C-002-11]` hoặc test assertion kiểm tra metric `db_fact_kept` (phát hiện không có event `model_override`); Thử nghiệm đã thấy FAIL trên `seeded_regression` và PASS trên `fixed`. |
| Chưa chắc chắn | Trace chỉ ghi nhận kết quả `model_override`, không chứng minh được liệu mô hình bị ảnh hưởng bởi prompt injection hay do prompt parsing logic mặc định ưu tiên dữ liệu không có cấu trúc trong ticket. |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
