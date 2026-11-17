# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: NhomTuesday
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`): chờ commit toàn bộ thay đổi báo cáo và artifact

## 2. Mục tiêu và cách tiếp cận

Mục tiêu chính của bài lab là xử lý claim hoàn phí vận chuyển theo bốn kiến trúc—workflow, single, orchestrator, swarm—và chọn kiến trúc ít tự chủ nhất mà vẫn đạt hidden threshold. Cách tiếp cận là giữ cùng một claim corpus và cùng 5 seed, so sánh theo cùng bảng giá, cùng tool và cùng evaluator. Báo cáo này dựa trên `src/reports/benchmark.json` và `src/reports/crash.json` được sinh ra bằng lệnh chuẩn của lab.

## 3. Giải pháp

Các shape A/B/C xác định logic ở code, model chỉ làm phần dịch/soạn text hoặc chọn tool đọc; hình thức `issue_refund` chỉ do host là người ghi ledger. `workflow` là dạng cố định nhất, `single` là một vòng ReAct tối đa 8 bước, `orchestrator` là điều phối và handoff theo allowlist. Handoff chỉ chứa các field được phép; không ghi `parent_secret`, header, SĐT, địa chỉ, ghi chú nội bộ, tên hàng. Trace đã được tác nghiệp theo contract v1: file JSON có `run_id` và `events`, đồng thời không chứa secret và PII. Tất cả đều phải đọc theo `docs/architecture.md` và `docs/contracts.md` để tránh fail-closed.

## 4. Kết quả

Benchmark công khai dùng 14 claim × 5 seed = 70 trial cho mỗi shape:

| Shape | Pass rate | Pass^5 | Seed min–max | Variance | Cost/success (VND, mô phỏng) | Mean / p95 steps |
|---|---:|---:|---:|---:|---:|---:|
| workflow | 1.0000 | 1.0000 | 1.0000–1.0000 | 0.000000 | 28.80 | 4.43 / 5 |
| single | 1.0000 | 1.0000 | 1.0000–1.0000 | 0.000000 | 48.89 | 8.14 / 9 |
| orchestrator | 1.0000 | 1.0000 | 1.0000–1.0000 | 0.000000 | 46.43 | 5.36 / 6 |
| swarm | 0.9857 | 0.9286 | 0.9286–1.0000 | 0.000816 | 49.57 | 2.80 / 3 |

Workflow/single/orchestrator đều 70/70 pass trên public; swarm 69/70, fail T02/C-002 ở seed 55 do quyết định không khớp nhãn khi delay trong ticket mâu thuẫn với order. Workflow đạt cùng pass rate với hai shape còn lại nhưng cost/success thấp nhất nên được chọn cho public benchmark. Không suy diễn kết quả public thành điểm hidden.

Crash suite trong `src/reports/crash.json` đạt `crash_safety = 20/20`. Cổng host chạy `92 passed, 248 deselected`; toàn bộ ma trận student chạy `248 passed, 92 deselected`. `python -m lab8.check_submission` báo `0 FAIL, 1 WARN`; cảnh báo duy nhất là thay đổi chưa commit. Không có thư mục `student_drop/` trong workspace nên hidden self-check chưa chạy.

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

Benchmark ban đầu được tạo khi ba shape còn stub nên không đại diện cho code hoàn thiện. Sau khi các shape và test tích hợp sẵn sàng, benchmark được chạy lại bằng lệnh chuẩn. Một lỗi được tái hiện ở swarm là T02/C-002 seed 55: ticket và order nêu delay khác nhau, swarm đưa ra trạng thái không khớp expected. Research card phân loại lỗi này theo hướng task verification như một phép diễn giải, không khẳng định paper MAST đã phân tích hệ thống của lab.

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và báo cáo cá nhân `report/team/MSSV_HoTen.md`.

## 7. Hạn chế và hướng cải thiện

MockLLM không phải model sản xuất; cost là mô phỏng và latency `unavailable`. Public set chỉ có 14 claim nên không đại diện cho toàn bộ hidden set. Chưa có `student_drop/`, do đó pass rate hidden và must-pass trên bộ grader không thể xác nhận trong workspace này. Kết luận kiến trúc sẽ cần được xem xét lại nếu workflow không đạt ngưỡng hidden 0.75 hoặc một shape tự chủ hơn có lợi ích pass-rate đáng kể trong khi vẫn giữ crash, approval và secret checks bằng 0 lỗi.

## 8. Tài liệu tham khảo

- `README.md`
- `docs/business_rules.md`
- `docs/contracts.md`
- `docs/metrics.md`
- `docs/architecture.md`
- `src/reports/benchmark.json`
- `src/reports/crash.json`
