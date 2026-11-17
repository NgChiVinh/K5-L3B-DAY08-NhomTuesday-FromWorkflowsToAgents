# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R2 |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Cemri et al. (2025), *Why Do Multi-Agent LLM Systems Fail?*, [arXiv:2503.13657](https://arxiv.org/abs/2503.13657); phân tích taxonomy trên trace hệ multi-agent |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Tác giả xây MAST từ phân tích 150 trace, xác định 14 failure mode trong ba nhóm: system design issues, inter-agent misalignment và task verification. |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Paper xây MAST-Data từ hơn 1.600 trace của 7 framework; phân tích gồm GPT-4, Claude 3, Qwen2.5, CodeLlama và các task coding/math/general agent. Kết quả không trực tiếp dự đoán hành vi MockLLM hay claim workflow này. |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Lỗi swarm T02 ở seed 55 có thể được phân loại gần với task verification: swarm ưu tiên delay sai trong ticket thay vì order. Dự đoán deterministic workflow sẽ tránh lỗi này; public pass rate 1.000, còn swarm khoảng 0.986 với variance dương. Đây là hypothesis về lab, không phải kết luận của paper. |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Cùng MockLLM, tool registry, 14 public claims, seeds `11,22,33,44,55`, bảng giá và evaluator. So sánh bốn shape; topology/control flow thay đổi cùng kiến trúc nên đây không phải ablation cô lập topology thuần túy. |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Xem lý do trial fail theo claim/seed, pass rate, pass^5, seed variance và cost/success. Cần phân loại T02/seed 55 là lỗi kiểm chứng nguồn dữ liệu theo cách tương tự taxonomy, không khẳng định nguyên nhân này thuộc MAST. |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json`
- Run id / file report: `R-workflow-C-001-s11`; kết quả đầy đủ trong `src/reports/benchmark.json`
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: phù hợp với hypothesis trên public set. Workflow, single và orchestrator đều pass 70/70 (pass rate/pass^5 = 1.000/1.000; variance = 0); swarm pass 69/70 (0.9857), pass^5 = 0.9286, variance = 0.000816, min–max = 0.9286–1.000. Trial swarm fail là T02/C-002 ở seed 55 (`status resolved != denied`), đúng tình huống ticket delay mâu thuẫn với order. Cost/success lần lượt: workflow 28.80, single 48.89, orchestrator 46.43, swarm 49.57 VND mô phỏng. Kết quả chỉ cho thấy tương quan trên fixture công khai; không chứng minh topology là nguyên nhân duy nhất và không thay thế hidden grading.
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
