# Team

## Thông tin nhóm

- Tên nhóm: NhomTuesday
- Lớp: L3B
- Repo nhóm: `K5-L3B-DAY08-NhomTuesday-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt: https://github.com/NgChiVinh/K5-L3B-DAY08-NhomTuesday-FromWorkflowsToAgents, nhánh `main`, commit chốt: (điền khi nộp)

## Thành viên

| STT | Họ tên          | MSSV        | Email                     | GitHub        | Vai trò |
| --: |-----------------|-------------|---------------------------|---------------| ------- |
|   1 | Ngô Chí Vĩnh    | 2A202603724 |                           | NgChiVinh     | Nhóm trưởng: Integration + Shape A workflow |
|   2 | Lê Việt Khoa    | 2A202603737 | levietkhoa2002@gmail.com  | Chưa cung cấp | Shape B — Single agent, test budget/trace/retry/crash |
|   3 | Đỗ Thanh Mai    | 2A202603862 | 26ai.maidt@vinuni.edu.vn  |               | Shape C orchestrator + handoff |
|   4 | Lê Đức Trúc Lâm | 2A202603787 | 26ai.lamldt@vinuni.edu.vn | truclamld     | Durability / policy / crash (QA cả 3 shape) |
|   5 | Phan Trung Hiếu | 2A202603763 | 26ai.hieupt3@vinuni.edu.vn | trunghieuph  | Eval + report (benchmark, ADR, research card) |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người                      | Vai                                         | Commit                                                      | Test đã thêm hoặc sửa                                                                                                                                                                                         |
|----------------------------|---------------------------------------------|-------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Đỗ Thanh Mai — 2A202603862 | Orchestrator + Handoff; security tests      | `73ad029`                                                   | `src/tests/test_orchestrator_security.py`: `test_orchestrator_handoff_and_trace_omit_sensitive_data`; `test_orchestrator_unknown_claim_writes_valid_trace`                                                    |
| Ngô Chí Vĩnh               | Integration + Shape A workflow              | `e7e6d98` CP2/CP3: implement shape A workflow and its tests | `src/tests/test_workflow_shape.py` (18 test: ticket injection, needs_info, unknown claim, model không đổi được tiền, trace sạch)                                                                              |
| Lê Việt Khoa (2A202603737) | Single agent                                | e3bb3137301681dc779e7b7e31dc9e3a55375915                    | src/tests/test_single_agent.py — test_single_counts_model_calls_and_writes_only_after_readonly_loop; test_single_step_cap_returns_valid_result_without_writing; test_single_crash_at_each_window_resumes_once |
| Lê Đức Trúc Lâm            | Durability / policy / crash (QA cả 3 shape) | `8ffa0829a986a706b9aea92c5be0bc4b56d7ea87`                  | `src/tests/test_shape_approval_mismatch.py` - test approval của claim khác bị từ chối trên cả ba shape                                                                                                        |
| Phan Trung Hiếu — 2A202603763 | Eval + Report | Chưa commit | `src/tests/test_metrics.py`: `test_p95_steps_uses_nearest_rank_for_report` |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Báo cáo cá nhân của Lê Việt Khoa: [2A202603737_LeVietKhoa.md](report/team/2A202603737_LeVietKhoa.md).

Báo cáo cá nhân của Phan Trung Hiếu: [2A202603763_PhanTrungHieu.md](report/team/2A202603763_PhanTrungHieu.md).

Sao chép mục dưới đây cho từng thành viên.

## 2A202603724_NgoChiVinh

### Phần việc phụ trách

- Nhóm trưởng: tạo repo nhóm, chia vai, review và merge PR, giữ `python -m pytest -q` xanh, chạy `check_submission` trước khi nộp.
- Shape A `workflow` (`src/lab8/shapes/workflow.py`).

### Đóng góp cụ thể

- Viết workflow: `get_claim_view` → `lookup_order` → `compute_compensation` → `durability.issue_refund` (quote đầy đủ, truyền `crash_at`, không bắt `InjectedCrash`). MockLLM chỉ soạn câu trả lời.
- Trace chỉ ghi view đã qua allowlist; không ghi ticket, `principal_id`, claim/order row thô.
- Viết `src/tests/test_workflow_shape.py` (18 test).

### Bằng chứng

- Commit `e7e6d98` trên nhánh `p1-workflow`.
- `python -m pytest -q -m student -o addopts= -k workflow`: 62 passed.
- `python -m lab8.check_submission`: `[ OK ] shape workflow: format đúng, 12/12 claim public pass ở seed 11`.

## 2A202603862_DoThanhMai

### Phần việc phụ trách

- Họ tên: Đỗ Thanh Mai; MSSV: 2A202603862; email: 26ai.maidt@vinuni.edu.vn.
- Shape C Orchestrator + Handoff (`src/lab8/shapes/orchestrator.py`), test bảo mật và biên bản rà soát trace.

### Đóng góp cụ thể

- Triển khai luồng đọc claim → `make_handoff` → `run_readonly_investigators` → quote bằng code → `issue_refund` → `finish`.
- Handoff theo allowlist với `facts={}`; investigator chỉ nhận tool đọc. Principal lấy từ claim trong DB; chỉ orchestrator gọi cổng refund của host.
- Truyền quote đầy đủ và `crash_at` vào host; không bắt `InjectedCrash`; luôn đóng DB bằng `finally`.
- Trace không ghi ticket, parent context, headers hoặc order row thô. Thêm hai test kiểm handoff/trace không lộ dữ liệu nhạy cảm và trace hợp lệ khi claim không tồn tại.
- Viết `report/team/orchestrator_security_review.md` và báo cáo cá nhân `report/team/2A202603862_DoThanhMai.md`. Rà luồng trace workflow/single sau integration còn cần thực hiện.

### Bằng chứng

- Commit triển khai và test: `73ad029`; tài liệu rà soát: `f665498`; báo cáo cá nhân: `0cfa984` trên nhánh `Orchestrator+Handoff`.
- `python -m pytest -q tests/test_orchestrator_security.py`: 2 passed.
- `python -m pytest -q -m student -o addopts= -k orchestrator`: 62 passed.
- Cổng host tại lần kiểm tra Shape C trước integration: 56 passed, 248 deselected.
- Run id minh họa: `R-orchestrator-C-001-s11`; trace không commit theo quy định repo.

## 2A202603787_LeDucTrucLam

## Phần việc phụ trách

Shape, module hoặc phần báo cáo bạn phụ trách.

- Phụ trách QA và kiểm tra các shape (workflow, single và orchestrator) đáp ứng yêu cầu must-pass và đạt crash safety (20/20).


## Đóng góp cụ thể

Commit chính (hash), test đã thêm hoặc sửa, quyết định kỹ thuật bạn đưa ra.

- Commit chính: 8ffa0829a986a706b9aea92c5be0bc4b56d7ea87
- Chạy ma trận student cho cả ba shape và crash suite; kết quả: cả ba shape pass, crash safety 20/20.
- Thêm test `test_wrong_claim_approval_is_denied` cho cả ba shape. Test xác nhận approval thuộc claim khác bị từ chối, không tạo refund trong ledger và host ghi audit `policy_denied`.
- Khi test ban đầu trả `failed`, phát hiện fixture chưa thêm claim `C-BORROW`; thêm claim vào database test với approval của claim khác để test đi đến bước kiểm tra policy, sau đó chạy lại thành công.

## Bằng chứng

Link commit, tên test, run id hoặc trích đoạn report liên quan.

- Chạy test `python -m pytest -q` và `python -m pytest -q -m student -o addopts=` đều trả về passed, không có failed.
- Chạy crash test đạt crash safety 20/20, cụ thể trong file src/reports/crash.json
- Test mới chạy thành công trên cả 3 shape.
