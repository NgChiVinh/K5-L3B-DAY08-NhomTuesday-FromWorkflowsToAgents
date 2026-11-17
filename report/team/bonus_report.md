# Bonus report: B2

- Topic + nguồn: B2 — gắn nhãn lỗi theo MAST; Cemri và cộng sự (2025), *Why Do Multi-Agent LLM Systems Fail?*, [arXiv:2503.13657](https://arxiv.org/abs/2503.13657).
- CLAIM của nguồn: MAST phân loại 14 failure mode quan sát trên trace MAS thành ba nhóm: system design issues, inter-agent misalignment và task verification.
- BOUNDARY: Paper xây taxonomy từ trace của nhiều framework/model/task; đây không phải bằng chứng rằng các phân loại cụ thể dưới đây là nhãn chính thức của paper. Thí nghiệm lab dùng MockLLM, public claims và fault case có kiểm soát.
- HYPOTHESIS trên LogiX: lỗi swarm T02/seed 55 là lỗi kiểm chứng dữ kiện vì ticket ghi delay khác order; handoff thiếu dữ kiện gần với inter-agent misalignment, còn bỏ qua approval gate là lỗi system design.
- Biến cố định / biến thay đổi: Benchmark public giữ cố định 14 claim, năm seed, giá, tool và evaluator. Fault case giữ cùng cấu trúc, chỉ đổi loại lỗi được cài chủ đích.
- Lệnh tái lập:

  ```text
  python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json
  python -m bonus.mast_tag --benchmark reports/benchmark.json --traces traces --manual-labels bonus/mast_manual_labels.json --out reports/bonus_mast.json
  python -m pytest -q tests/test_bonus_mast_tag.py
  ```

- File report + run id: `src/reports/bonus_mast.json`; public failure `R-swarm-C-002-s55`; fault cases `fault_handoff_drops_delay:11` … `:55`, `fault_uses_stale_clause:11` … `:55`, `fault_skips_approval_check:11` … `:55`.

## Kết quả

| Shape / fault variant | System design | Inter-agent misalignment | Task verification |
|---|---:|---:|---:|
| swarm (public benchmark failures) | 0 | 0 | 1 |
| fault_handoff_drops_delay | 0 | 5 | 0 |
| fault_uses_stale_clause | 0 | 0 | 5 |
| fault_skips_approval_check | 5 | 0 | 0 |

Tagger khớp 16 nhãn dự kiến (15 fault case có kiểm soát + một lỗi swarm public): observed agreement = 1.000; Cohen's κ = 1.000. 15 fault case là trace tổng hợp để kiểm tra các rule; chúng không phải trial benchmark của shape sản xuất. Nhãn trong `mast_manual_labels.json` cần thành viên nhóm rà soát trước khi nộp; κ cao trên fault case có kiểm soát không tương đương độ đồng thuận của annotator độc lập.

Trên public benchmark, workflow/single/orchestrator đều pass 70/70; swarm pass 69/70 và fail T02/C-002 seed 55 với `status resolved != denied`. Trace có event `model_override`; tagger phân loại đây là task verification. Pass rate và cost public nằm trong `src/reports/benchmark.json`.

## Kết luận

Lỗi swarm quan sát được phù hợp với giả thuyết task verification: giá trị trong ticket lấn át dữ liệu order có cấu trúc và dẫn tới quyết định sai. Ba fault case có kiểm soát được phân loại vào nhóm dự kiến. Kết quả này chỉ kiểm tra tagger trên signature tường minh và một lỗi public; agreement tuyệt đối không chứng minh tagger khái quát được sang lỗi MAS tự nhiên. Tập dữ liệu lớn hơn, được gán nhãn độc lập, có thể cho agreement khác.

## Giới hạn

Tagger dùng rule và trace signature minh bạch, không hiểu ngữ nghĩa như người. 15 ví dụ có kiểm soát được thiết kế sẵn cho ba nhóm mục tiêu nên điểm agreement có thiên lệch thuận lợi; nhóm cần tự rà soát nhãn trước khi nộp. MockLLM không phải model sản xuất, cost là mô phỏng, latency `unavailable`, và public corpus nhỏ. Hidden self-check không có nhãn nên không xác nhận cổng bonus: hidden pass rate chính thức ≥ 0.80 và must-pass vẫn cần grader giảng viên.
