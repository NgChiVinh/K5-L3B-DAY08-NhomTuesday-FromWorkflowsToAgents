# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: `27b35cc`
- Run id dẫn chứng: `R-workflow-C-001-s11` (`src/reports/benchmark.json`)
- Pass rate / pass^5 / seed min-max / variance: 1.000 / 1.000 / 1.000–1.000 / 0.000000 (70 public trials, 14 claims × 5 seeds)
- Cost per success (mô phỏng, VND): 28.80
- Crash safety: 20/20 (`src/reports/crash.json`); ma trận student: 248 passed
- Secret leak / approval bypass: 0 / 0 lỗi public; hidden self-check có 0 leak suspect nhưng không có nhãn để tính approval bypass

## Vì sao shape này

Chúng tôi chọn `workflow`: trên cùng 14 public claims và 5 seeds, workflow đạt 70/70 pass (pass rate và pass^5 đều 1.000), bằng single và orchestrator, đồng thời có cost/success thấp nhất (28.80 VND mô phỏng so với 48.89, 46.43 và 49.57). Đây cũng là kiến trúc ít tự chủ nhất: code cố định chuỗi `get_claim_view` → `lookup_order` → `compute_compensation` → `issue_refund` → `finish()`, không để model quyết số tiền hay ghi refund. Kết quả này ủng hộ chọn workflow trên public benchmark; chưa phải bằng chứng về bộ hidden.

## Có đáng đổi lấy multi-agent không

Không đáng đổi trong kết quả hiện tại: single/orchestrator không tăng pass rate public so với workflow và tốn chi phí mô phỏng cao hơn; swarm thấp hơn một trial và tốn hơn. Chỉ đổi quyết định nếu benchmark hidden trên cùng commit cho thấy workflow không đạt ngưỡng 0.75 hoặc một kiến trúc khác cải thiện pass rate đủ để biện minh cho chi phí/tự chủ tăng thêm, đồng thời vẫn qua cả ba must-pass.

## Trace và bằng chứng

Trace ghi run id và event đã sanitize như tool/status/quote/decision; cố ý không ghi `parent_secret`, headers, ticket nguyên văn, claim/order row thô, số điện thoại, địa chỉ, ghi chú nội bộ hay tên hàng. Public benchmark 70/70 workflow trial pass, tức là trace contract và forbidden-string assertions của các nhãn public đều đạt. Đây không thay thế scan hidden của giảng viên.

## Giới hạn

MockLLM không phải model sản xuất; cost là mô phỏng và latency là `unavailable`. Hidden self-check trên 40 claim × 5 seed cho mỗi shape đã chạy hết 200 trial/shape, không có error hoặc leak suspect; do không có nhãn, kết quả không phải pass rate hidden và không xác nhận ngưỡng 0.75/0.80. Commit cuối cần được cập nhật sau khi nhóm commit toàn bộ thay đổi.
