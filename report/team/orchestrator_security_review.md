# Rà soát Orchestrator + Handoff

- Ngày kiểm tra: 17/11/2026.
- Nhánh: `Orchestrator+Handoff`.
- Commit triển khai: `73ad029`.

## Phạm vi và kết quả

Shape C dùng `make_handoff()` với `facts={}` và các tool `lookup_order`,
`lookup_policy`, `compute_compensation`. Principal lấy từ claim trong DB.
Investigator không được cấp tool ghi; chỉ orchestrator gọi `issue_refund()`.
Quote do code tính từ order và policy, không lấy số ngày trễ trong ticket.
Quote được truyền đầy đủ để host kiểm approval và trừ refund trước đó.
`crash_at` được chuyển vào host; `InjectedCrash` truyền ra harness.

Trace chỉ ghi kết quả tìm claim, handoff, trạng thái lookup order, quote,
kết quả host và quyết định. Không ghi nguyên claim, ticket, order DB,
parent context, headers, phone, address, item name hay internal note.

## Bằng chứng

Chạy trong `src/` với Python của `.venv`:

- `python -m pytest -q tests/test_orchestrator_security.py`: 2 passed.
- `python -m pytest -q`: 56 passed, 248 deselected.
- Ma trận Shape C: 62 passed, 242 deselected (sau khi thêm hai test security).

```powershell
python -m pytest -q -m student -o addopts= -k orchestrator
```

Trong môi trường chạy của Codex, pytest dùng thư mục tạm riêng trong workspace
và tắt cache do thư mục temp mặc định của Windows bị từ chối truy cập.
Không thay đổi evaluator, fixture nguồn hoặc logic test để xử lý lỗi môi trường.

Test bảo mật chèn secret, phone và address vào ticket của DB tạm rồi kiểm
handoff thực tế, RunResult, trace và stdout/stderr không chứa chúng.
Test bổ sung xác nhận claim không tồn tại vẫn tạo trace đúng run_id.
Ma trận có sẵn kiểm retry và crash/resume qua T11, T12.

## Phần cần kiểm tra sau integration

`workflow.py` và `single.py` tại commit trên vẫn trả `not_implemented`.
Đã đọc mã hiện tại nhưng chưa thể chứng nhận trace của luồng xử lý thật.
Sau khi chủ file hoàn thiện và merge, cần:

1. Chạy ma trận riêng từng shape và kiểm trace của mọi nhánh trả sớm.
2. Kiểm mọi `trace.add`, `print`, logger và thao tác ghi file.
3. Thử ticket có secret/PII, kiểm kết quả, trace, side files và stdout/stderr.
4. Xác nhận không dump claim/order DB hoặc coi `safe_dump` là sanitizer.

Chưa có kết quả hidden; public pass không chứng minh hidden pass rate.
Crash suite chung kiểm host và swarm, không thay thế T12 của Shape C.
