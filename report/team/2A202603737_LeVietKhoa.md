# Báo cáo cá nhân — 2A202603737_LeVietKhoa

## 1. Thông tin thành viên

- Họ tên: Lê Việt Khoa.
- MSSV: 2A202603737.
- Email: levietkhoa2002@gmail.com (theo tác giả commit triển khai).
- GitHub: chưa cung cấp tên tài khoản.
- Vai trò: Shape B — Single agent; kiểm thử các ranh giới đọc/ghi, budget, trace và retry/crash của shape này.

## 2. Phần việc phụ trách

Hoàn thiện `src/lab8/shapes/single.py` từ stub, giữ nguyên chữ ký `run_claim(claim_id, *, seed, store_path, crash_at=None) -> RunResult`. Phạm vi bao gồm vòng ReAct tối đa 8 lượt, tool chỉ đọc, tính quote bằng code sau vòng lặp, gọi host để ghi refund và trả kết quả đúng contract.

Các tài liệu dùng để đối chiếu: [README](../../README.md), [CHECKPOINTS](../../CHECKPOINTS.md), [luật nghiệp vụ](../../docs/business_rules.md), [output và trace contract](../../docs/contracts.md), [kiến trúc](../../docs/architecture.md), [metric](../../docs/metrics.md), [RULES](../../RULES.md), [RUBRIC](../../RUBRIC.md) và [SUBMISSION](../../SUBMISSION.md).

## 3. Đóng góp cụ thể

### 3.1. Cách tổ chức Single agent

Vòng `run_react` chạy với `max_steps=8`, `worker=True`. Hàm `decide()` lưu kết quả đã đọc và chọn bước tiếp theo theo observation: `get_claim_view` → `lookup_order` → `lookup_policy` → `lookup_refund_history` → kết thúc điều tra. Các trường hợp thiếu claim, thiếu `order_id` hoặc không nhìn thấy đơn kết thúc sớm.

Mỗi lượt `decide()` gọi `lab8.mock_llm.complete` với cùng seed của trial. Hàm này cập nhật `Budget`, bao gồm số lần gọi, token và chi phí mô phỏng; không cộng lượt model bằng số tự khai. Tool trong vòng lặp được tính phí qua dispatcher. Lời gọi `compute_compensation` sau vòng lặp được cộng thêm một lượt tool.

MockLLM có sẵn chỉ trả văn bản và usage mô phỏng, không sinh JSON lệnh gọi tool. Vì vậy, phần chọn tool là adapter Python dựa trên observation; kết quả hiện tại không chứng minh khả năng lập kế hoạch hoặc hiểu ngôn ngữ của một LLM thật.

### 3.2. Ranh giới nghiệp vụ và bảo mật

| Yêu cầu | Cách xử lý trong phần Single |
| --- | --- |
| Không tin số ngày hay lệnh trong ticket | Dùng `order_id` và principal từ claim; quote lấy từ order đã qua kiểm tra phạm vi sở hữu. Không lấy số tiền, ngày trễ hoặc lệnh thực thi từ ticket. |
| Model chỉ dùng tool đọc | `worker=True` buộc dispatcher kiểm allowlist; yêu cầu tool ghi bị chặn và trả kết quả `failed/tool_rejected`. |
| Code quyết định tiền | Sau ReAct, gọi `compute_compensation` với order đã đọc; helper dùng policy hiện hành, có hiệu lực và chọn entitlement lớn nhất. |
| Không trừ tiền hoàn lịch sử hai lần | Truyền quote đầy đủ vào `issue_refund`; host tự trừ số tiền claim trước đã nhận theo cùng đơn. |
| Approval hỏng phải có audit | Quote dương vẫn đi qua host. Host từ chối thì giữ nguyên `proposal_amount`, trả `denied` và có audit `policy_denied`. |
| Retry và conflict | Dùng ledger/idempotency của host; conflict trả `failed`, giữ nguyên refund đã commit. |
| Crash thật sự xảy ra | Truyền nguyên `crash_at`, không bắt `InjectedCrash`; `finally` ghi trace và đóng connection khi exception đi ra ngoài. |
| Kết quả và trace đúng contract | Mọi nhánh trả về dùng `finish()`; refund và committed audit được đọc từ ledger. Trace có run id đúng và được ghi trong lần chạy hiện tại. |
| Không lộ dữ liệu nhạy cảm | Trace chỉ ghi tên tool, trạng thái, quote và thông tin quyết định. Không ghi ticket, tool arguments, claim/order thô, secret, header hay PII. |
| Chạm giới hạn 8 lượt | Trả `RunResult` hợp lệ với `failed/step_cap`, không tiếp tục ghi refund. |

### 3.3. Code, test và commit

- Code: [single.py](../../src/lab8/shapes/single.py).
- Test bổ sung: [test_single_agent.py](../../src/tests/test_single_agent.py), hiện có 14 trường hợp sau khi mở rộng parametrization.
- Commit triển khai đã có trong lịch sử: `e3bb3137301681dc779e7b7e31dc9e3a55375915` — `update single`. Commit này chứa Single agent, bộ test ban đầu và benchmark riêng cho Single.
- Lần rà soát này bổ sung `test_single_changed_quote_on_retry_preserves_committed_refund`: chạy refund 35.000 VND, đổi phí ship thành 25.000 VND rồi retry; kết quả phải là conflict, ledger vẫn giữ 35.000 VND và một committed audit.
- Test conflict, báo cáo cá nhân, cập nhật `TEAM.md` và artifact crash mới cần được commit bổ sung; chúng chưa nằm trong commit triển khai nêu trên tại thời điểm viết báo cáo.

Nguồn tham khảo mã là các helper có sẵn trong starter (`runtime`, `tools`, `mock_llm`, `durability`, `trace`, `shapes.common` và swarm tham chiếu). Không sửa host, metric, config hoặc fixture gốc.

## 4. Bằng chứng

### 4.1. Lệnh kiểm tra đã chạy

Các lệnh dưới đây chạy trong thư mục `src/`; kết quả được xác nhận lại trong phiên rà soát ngày 17/11/2026 (GMT+7).

| Lệnh | Kết quả thực tế |
| --- | --- |
| `python -m lab8.smoke` | `lab8.smoke pass` |
| `python -m pytest -q` | `68 passed, 248 deselected` |
| `python -m pytest -q -m student -o addopts= -k 'single]'` | `62 passed, 253 deselected` |
| `python -m lab8.benchmark --shapes single --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/single_benchmark.json` | 70/70 trial pass |
| `python -m lab8.crash_test --count 20 --out reports/crash.json` | `crash_safety 20/20` |

Lọc bằng `single]` để chọn đúng tham số shape. Dùng `-k single` còn khớp tên `test_public_single_run_cases` và kéo theo test của các shape khác.

Môi trường kiểm tra thực tế: Windows AMD64, Python 3.14.10. README quy định Python 3.11–3.13; kết quả trên máy hiện tại không thay thế việc chạy lại trên phiên bản được lab hỗ trợ trước khi nộp.

### 4.2. Kết quả benchmark riêng cho Single

Nguồn số liệu: [src/reports/single_benchmark.json](../../src/reports/single_benchmark.json), do CLI sinh và đã chạy lại để đối chiếu. Mỗi trial dùng SQLite mới, cùng corpus 14 claim, 5 seed và bảng giá mô phỏng của lab.

| Metric | Giá trị |
| --- | ---: |
| Số trial / số pass | 70 / 70 |
| Pass rate | 1,000 |
| Pass^5 | 1,000 (14/14 claim đúng ở cả 5 seed) |
| Seed variance | 0,000000 |
| Seed min / max | 1,000 / 1,000 |
| Tổng cost mô phỏng | 3.422 VND |
| Cost / success mô phỏng | 48,89 VND |
| Mean model calls | 4,642857 |
| Mean tool calls | 4,5 |
| Mean / p95 trace steps | 8,142857 / 9 |
| Latency | `unavailable` |

`steps` trong metric là số event của trace, gồm cả quote và quyết định sau ReAct. Vì vậy p95 bằng 9 không có nghĩa vòng ReAct vượt 8 lượt. Giới hạn vòng lặp được kiểm riêng bằng `test_single_step_cap_returns_valid_result_without_writing`, xác nhận dừng ở 8 lần gọi model và không refund.

Run id đối chiếu trong benchmark: `R-single-C-001-s11`, `R-single-C-002-s55`, `R-single-C-013-s11`, `R-single-C-014-s11`.

### 4.3. Các bằng chứng nghiệp vụ và an toàn

- T01: quote và refund 35.000 VND; có một audit committed.
- T02: dùng `delay_days` từ order, không tin số ngày trong ticket; vẫn đúng ở seed 55.
- T11/T12: gọi lại và crash trước checkpoint không tạo refund thứ hai.
- T13: quote 35.000 VND nhưng đơn đã hoàn đủ; host từ chối và ghi audit.
- T14: quote đầy đủ 150.000 VND; host ghi phần chênh 120.000 VND sau refund cũ 30.000 VND.
- Test bổ sung kiểm tra tool ghi bị chặn, budget có model usage, giới hạn bước, trace khi return sớm, ticket injection chứa secret/PII, approval revoked/expired/zero/sai claim, cả ba cửa sổ crash và conflict khi retry.

Artifact [crash.json](../../src/reports/crash.json) là suite chung có sẵn của host và swarm tham chiếu, đạt 20/20. Bằng chứng crash riêng của Single là T12 cùng ba trường hợp trong `test_single_crash_at_each_window_resumes_once`; không diễn giải suite chung thành 20 điểm crash riêng cho Single.

## 5. Công cụ AI đã dùng

Sử dụng Codex để đọc tài liệu, hỗ trợ triển khai Single agent, viết test, rà soát contract, thực thi lệnh kiểm tra và soạn báo cáo từ artifact thực tế. Các kết quả trong mục 4 là output đã chạy trong workspace; không phải số liệu do AI ước lượng. Sinh viên cần tự chạy lại các lệnh và giải thích được các quyết định trong mục 3 khi bảo vệ bài.

Mọi lời gọi model trong code lab đều dùng MockLLM có sẵn; không gọi API model thật, không dùng API key. Không dùng nguồn code ngoài repo. Báo cáo ghi rõ giới hạn của MockLLM và không tự nhận điểm hidden hay điểm rubric.

## 6. Tự đánh giá

Về kết quả kỹ thuật có thể kiểm chứng, phần Single đáp ứng các yêu cầu đã đối chiếu của CP2, các nhánh retry/crash liên quan ở CP3 và các kiểm tra approval/trace ở CP4 trên dữ liệu public. Các test host vẫn xanh. Lần rà soát chưa phát hiện lỗi cần sửa trong `single.py`; đã bổ sung kiểm thử cho nhánh conflict còn thiếu bằng chứng trực tiếp ở cấp shape.

Chưa có `student_drop/` trong workspace, nên chưa chạy hidden self-check và chưa có cơ sở khẳng định đạt ngưỡng hidden 0,75. Benchmark riêng của Single cũng chưa đủ để kết luận đây là kiến trúc tốt nhất: quyết định đó cần benchmark chung bốn shape và ADR của nhóm. Chi phí chỉ là mô phỏng, không phải hóa đơn API.

Trước khi nộp cần chạy lại trên Python 3.11–3.13, commit các bổ sung và kiểm tra trên branch tích hợp. Các deliverable chung như benchmark bốn shape, ADR, research card, báo cáo nhóm và `check_submission` phải được hoàn thiện ở cấp nhóm. Phần tự đánh giá về mức độ hiểu bài và quá trình học của cá nhân cần do sinh viên xác nhận sau khi tự chạy lại và đọc mã.
