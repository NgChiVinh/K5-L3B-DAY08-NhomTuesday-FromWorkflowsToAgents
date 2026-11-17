# Báo cáo cá nhân — MSSV_HoTen

> Cần thay `MSSV_HoTen` bằng họ tên không dấu và MSSV trước khi nộp. Thông tin cá nhân chưa có trong repo.

## 1. Thông tin thành viên

- Họ tên: Chưa cung cấp
- MSSV: Chưa cung cấp
- Email: Chưa cung cấp
- GitHub: Chưa cung cấp
- Vai trò: Thành viên số 5 — Eval + Report

## 2. Phần việc phụ trách

- Chạy benchmark bốn shape (`workflow`, `single`, `orchestrator`, `swarm`) trên cùng 14 public claims và các seed `11, 22, 33, 44, 55`.
- Tổng hợp kết quả để so sánh pass rate, pass^5, seed variance, cost/success và số bước; ghi giới hạn của MockLLM.
- Chạy crash suite, bổ sung test cho metric p95, và cập nhật ADR, research card, group report theo bằng chứng đo được.

## 3. Đóng góp cụ thể

- Thêm `test_p95_steps_uses_nearest_rank_for_report` trong `src/tests/test_metrics.py`, kiểm tra mean và p95 steps theo quy tắc nearest-rank.
- Sinh lại `src/reports/benchmark.json` bằng lệnh benchmark chuẩn với bốn shape, 14 claims và năm seed; mỗi shape có 70 trial.
- Chạy crash suite và sinh `src/reports/crash.json`; kết quả là 20/20.
- Cập nhật các báo cáo nhóm `report/team/decision_adr.md`, `report/team/research_card.md` và `report/team/group_report.md`.
- Commit cá nhân: Chưa có; cần ghi hash sau khi commit các thay đổi của mình.

## 4. Bằng chứng

Lệnh benchmark:

```text
python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json
```

Kết quả trong `src/reports/benchmark.json`:

| Shape | Pass rate | Pass^5 | Seed min–max | Variance | Cost/success (VND, mô phỏng) | Mean / p95 steps |
|---|---:|---:|---:|---:|---:|---:|
| workflow | 1.0000 | 1.0000 | 1.0000–1.0000 | 0.000000 | 28.80 | 4.43 / 5 |
| single | 1.0000 | 1.0000 | 1.0000–1.0000 | 0.000000 | 48.89 | 8.14 / 9 |
| orchestrator | 1.0000 | 1.0000 | 1.0000–1.0000 | 0.000000 | 46.43 | 5.36 / 6 |
| swarm | 0.9857 | 0.9286 | 0.9286–1.0000 | 0.000816 | 49.57 | 2.80 / 3 |

- Run ID ví dụ trong benchmark: `R-workflow-C-001-s11`, `R-single-C-001-s11`, `R-orchestrator-C-001-s11`, `R-swarm-C-001-s11`.
- Crash: `src/reports/crash.json` ghi `crash_safety = 20/20`.
- Test metric và robustness: `python -m pytest -q tests/test_metrics.py tests/test_benchmark_robustness.py` — 6 passed.
- Hidden benchmark chưa được chạy; không suy ra ngưỡng hidden từ benchmark public.

## 5. Công cụ AI đã dùng

- Copilot SDK trong VS Code hỗ trợ rà cấu trúc benchmark/metric, thêm test p95, chạy lệnh benchmark và soạn bản nháp báo cáo.
- Tự kiểm tra bằng cách chạy lại benchmark chuẩn, đối chiếu các số trong JSON được sinh với bảng trên, chạy crash suite và chạy nhóm test metric/robustness.
- Các số liệu nộp cần được nhóm kiểm tra lại trên commit cuối; không sửa tay JSON báo cáo.

## 6. Tự đánh giá

Phần eval hiện đã có benchmark public tái lập cho cả bốn shape, kết quả crash suite và test metric p95. Trên public benchmark hiện tại, workflow có cùng pass rate và pass^5 với single/orchestrator nhưng cost/success thấp nhất; vì vậy workflow là lựa chọn tạm thời phù hợp với mục tiêu chọn kiến trúc ít tự chủ nhất. Đây chưa phải quyết định hidden cuối cùng: cần xác nhận must-pass và pass rate hidden trên commit nộp. Hạn chế là MockLLM và giá mô phỏng không đại diện cho production; latency không được đo (`unavailable`).
