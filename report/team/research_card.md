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
| CÂU | R1 / R2 / R3 / R4 | R4 · Evaluation confounders: chứng minh chênh lệch giữa workflow (A) và single (B) đến từ vòng ReAct, không phải từ prompt, tool hay seed. |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Zhang và cộng sự, *Rethinking the Evaluation of Efficiency Methods for Multi-Agent Systems*, EMNLP 2026 (main conference), https://aclanthology.org/2026.emnlp-main.798/ (arXiv:2609.05933). Loại bằng chứng: benchmark chẩn đoán có kiểm soát, bài hội nghị có bình duyệt. |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Các đánh giá hiện nay có thể phóng đại lợi ích của những phương pháp làm hệ multi-agent rẻ hơn (bỏ agent, bỏ cạnh giao tiếp, tìm cấu trúc gọn). Nhiều kết quả tốt xuất hiện ở task không thật sự cần nhiều agent, nơi một agent đơn hoặc hệ bị cắt ngẫu nhiên vẫn làm tốt, nên nhiều mức cải thiện phụ thuộc vào setup hơn là vào thay đổi cấu trúc. |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Paper đo các phương pháp tối ưu hiệu quả của MAS chạy trên LLM thật, cùng một backbone, cùng agent registry và runtime, rồi thay đổi topology, số agent, độ sâu và quyền dùng tool; abstract không công bố số liệu định lượng. Lab này khác ở ba điểm: MockLLM không suy luận, chỉ có 14 claim public, và phép so là workflow với một agent chứ không phải cắt tỉa một hệ nhiều agent. Vì vậy nhóm chỉ mượn phần phương pháp: so trong điều kiện kiểm soát và luôn đặt một baseline đơn giản bên cạnh. |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Đổi A (workflow) → B (single ReAct), giữ nguyên mọi thứ khác: pass rate và pass^5 không đổi (cùng 1.0), seed variance vẫn 0, vì ở cả hai shape trạng thái và tiền đều do code quyết. Cost/success tăng khoảng 10–20% vì B thêm khoảng 3 lần gọi MockLLM mỗi trial (mỗi lần 1–2 VND, trong khi phần tool khoảng 30 VND). Mean steps tăng khoảng 1. Nếu đúng, claims là task không cần agent: baseline đơn giản đã bão hòa. |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Giữ cố định: MockLLM và seed 11, 22, 33, 44, 55; bộ tool đọc (get_claim_view, lookup_order, compute_compensation); prompt soạn trả lời `status=… proposal=…`; corpus `data/public_claims.jsonl` (14 claim); giá `config/pricing.yaml`; host, evaluator `lab8.metrics`, DB SQLite mới cho mỗi trial. Biến duy nhất được thay đổi: điều khiển bằng các bước Python cố định (A) hay bằng vòng ReAct tối đa 8 bước, mỗi bước một lần gọi model (B). |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Lỗi dự kiến của chính phép so, đúng loại confounder paper cảnh báo: chênh cost bị gán cho topology trong khi thật ra hai shape gọi số tool khác nhau hoặc dùng prompt khác nhau. Đo: pass rate, pass^5, seed variance, cost/success, mean/p95 steps, mean_model_calls, mean_tool_calls trong report, cộng kiểm status và tool_calls bằng nhau trên từng cặp (claim, seed). Leak và bypass kiểm qua `trial_pass` (forbid list và đối chiếu ledger). |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn, trong `src/`): `python -m lab8.benchmark --shapes workflow,single --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/ablation_workflow_vs_single.json`. Kiểm từng cặp (claim, seed), lệnh in ra `70 / 70`: `python -c "import json; r=json.load(open('reports/ablation_workflow_vs_single.json', encoding='utf-8')); t={(x['shape'], x['claim_id'], x['seed']): x for x in r['trials']}; k=[(c, s) for (sh, c, s) in t if sh == 'workflow']; print(sum(t[('workflow', c, s)]['status'] == t[('single', c, s)]['status'] and t[('workflow', c, s)]['tool_calls'] == t[('single', c, s)]['tool_calls'] for c, s in k), '/', len(k))"`
- Run id / file report: `src/reports/ablation_workflow_vs_single.json`, sinh trên commit `bcea95e`; chạy lại lần hai cho kết quả giống hệt. 70 trial mỗi shape, run id dạng `R-<shape>-<claim>-s<seed>`, ví dụ `R-workflow-C-002-s55` và `R-single-C-002-s55` (C-002 seed 55 là trial swarm fail; cả A và B đều ra `denied`).
- Kết quả so với hypothesis: **đúng**. Hai shape ra cùng status trên 70/70 cặp (claim, seed) và cùng số tool call, nên toàn bộ chênh lệch nằm ở vòng ReAct: thêm 2,93 lần gọi model mỗi trial mà không thêm trial pass nào.

  | Metric | workflow (A) | single (B) |
  | --- | ---: | ---: |
  | pass rate | 1.000 | 1.000 |
  | pass^5 | 1.000 | 1.000 |
  | seed variance (min–max) | 0 (1.0–1.0) | 0 (1.0–1.0) |
  | cost/success (VND mô phỏng) | 28,87 | 32,71 (+3,84; +13,3%) |
  | mean / p95 steps | 4,43 / 5 | 5,43 / 6 |
  | mean model calls | 1,00 | 3,93 |
  | mean tool calls | 2,79 | 2,79 |

  Trên LogiX, B không đáng đổi so với A. Đây đúng là tình huống paper cảnh báo: baseline đơn giản đã bão hòa, nên một kết luận kiểu "agent tốt hơn" chỉ có nghĩa khi đặt baseline này bên cạnh. Kết luận sẽ đảo nếu task có bước mà code không quyết được (ví dụ ticket thiếu dữ liệu có cấu trúc, cần đọc hiểu để hỏi lại khách), vì khi đó pass rate của A có thể thấp hơn 1.0.
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`. MockLLM không suy luận, nên ablation chỉ đo được chi phí của vòng agent, không đo được lợi ích có thể có của suy luận. Giá là mô phỏng: mỗi lần gọi tool 10 VND chiếm phần lớn chi phí, model chỉ 20/80 VND mỗi 1.000 token; nếu giá model tương đối cao hơn, chênh lệch sẽ lớn hơn 13%. Chỉ có 14 claim public, chưa chạy trên `student_drop/`. `steps` là số event trong trace nên phụ thuộc cách mỗi shape ghi trace.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
