# Báo cáo cá nhân — Orchestrator + Handoff (bản nháp)

Đổi tên thành `MSSV_HoTen.md` sau khi xác nhận thông tin cá nhân.
Giữ nguyên file mẫu `MSSV_HoTen.md`.

## 1. Thông tin thành viên

- Họ tên: Chờ xác nhận.
- MSSV: Chờ xác nhận.
- Email: Chờ xác nhận (Git hiện cấu hình thanhmaido24@gmail.com).
- GitHub: Chờ xác nhận; không suy ra từ tài khoản chủ repo.

## 2. Phần việc phụ trách

Triển khai Shape C trong `src/lab8/shapes/orchestrator.py`, handoff allowlist,
investigator chỉ đọc và kiểm tra bảo mật trace. Rà trace workflow/single sau
khi hai shape được tích hợp.

## 3. Đóng góp cụ thể

- Commit triển khai: `73ad029`.
- Tạo handoff tối thiểu, không truyền ticket, secret hoặc headers.
- Dùng principal từ DB và tool lookup có kiểm scope.
- Dùng quote do code tính; chỉ orchestrator gọi cổng refund của host.
- Truyền quote đầy đủ và `crash_at`; đóng DB bằng `finally`.
- Dùng `finish()` cho các nhánh kết quả để ghi trace và đọc ledger thực tế.
- Thêm `test_orchestrator_handoff_and_trace_omit_sensitive_data` và
  `test_orchestrator_unknown_claim_writes_valid_trace` trong
  `src/tests/test_orchestrator_security.py`.

## 4. Bằng chứng

- Test bảo mật: 2 passed.
- Cổng host: 56 passed, 248 deselected.
- Người dùng đã chạy ma trận orchestrator: 62 passed.
- Run id minh họa: `R-orchestrator-C-001-s11` (trace không commit).
- Chi tiết rà soát và giới hạn: `orchestrator_security_review.md`.

## 5. Công cụ AI đã dùng

ChatGPT/Codex hỗ trợ đọc contract, đề xuất triển khai, viết test bảo mật,
kiểm tra mã và chạy test. Kết quả được đối chiếu với test thực tế và contract
trong repo. Không dùng API model thật, không tạo số liệu benchmark/hidden.

## 6. Tự đánh giá

Shape C đã có bằng chứng public và test bảo mật. Cần tiếp tục rà workflow và
single sau integration; chưa kết luận về hidden hay lựa chọn kiến trúc của nhóm.
Thông tin cá nhân cần được xác nhận trước khi nộp.
