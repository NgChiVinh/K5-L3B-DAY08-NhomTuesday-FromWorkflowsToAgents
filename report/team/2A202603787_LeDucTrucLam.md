# Báo cáo cá nhân — MSSV_HoTen

Đổi tên file theo quy tắc `MSSV_HoTen.md`.

Ví dụ: `20241234_NguyenVanAn.md`.

## 1. Thông tin thành viên

- Họ tên: Lê Đức Trúc Lâm
- MSSV: 2A202603787
- Email: 26ai.lamldt@vinuni.edu.au
- GitHub: truclamld

## 2. Phần việc phụ trách

Shape, module hoặc phần báo cáo bạn phụ trách.

- Phụ trách QA và kiểm tra các shape (workflow, single và orchestrator) đáp ứng yêu cầu must-pass và đạt crash safety (20/20).


## 3. Đóng góp cụ thể

Commit chính (hash), test đã thêm hoặc sửa, quyết định kỹ thuật bạn đưa ra.

- Commit chính: 8ffa0829a986a706b9aea92c5be0bc4b56d7ea87
- Chạy ma trận student cho cả ba shape và crash suite; kết quả: cả ba shape pass, crash safety 20/20.
- Thêm test `test_wrong_claim_approval_is_denied` cho cả ba shape. Test xác nhận approval thuộc claim khác bị từ chối, không tạo refund trong ledger và host ghi audit `policy_denied`.
- Khi test ban đầu trả `failed`, phát hiện fixture chưa thêm claim `C-BORROW`; thêm claim vào database test với approval của claim khác để test đi đến bước kiểm tra policy, sau đó chạy lại thành công.

## 4. Bằng chứng

Link commit, tên test, run id hoặc trích đoạn report liên quan.

- Chạy test `python -m pytest -q` và `python -m pytest -q -m student -o addopts=` đều trả về passed, không có failed.
- Chạy crash test đạt crash safety 20/20, cụ thể trong file src/reports/crash.json
- Test mới chạy thành công trên cả 3 shape.

## 5. Công cụ AI đã dùng

Công cụ, dùng vào việc gì, và bạn đã tự kiểm tra kết quả thế nào (xem [RULES.md](../../RULES.md)).

- Có dùng ChatGPT/Codex để hiểu checklist QA và nhận gợi ý về test.
- Tự kiểm tra bằng pytest; xác nhận test mới pass và đối chiếu kết quả với các test host liên quan.

## 6. Tự đánh giá

Việc QA diễn ra suôn sẻ nhờ các shape Workflow, Single và Orchestrator đều hoàn thiện. Phần repo đã gần như hoàn chỉnh và vượt qua cả 3 must-pass của đề bài.
