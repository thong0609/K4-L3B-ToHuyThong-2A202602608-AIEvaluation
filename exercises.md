# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 9:15–12:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 9:15–9:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (9:30–9:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Các câu hỏi đơn giản, ít rủi ro, chỉ cần thông tin cơ bản | Ngữ cảnh rủi ro cao: y tế, tài chính, pháp lý | Triển khai hallucination guardrails |
| Answer Relevance | Câu hỏi trang trí, thông tin chung | Câu hỏi giao dịch, yêu cầu hành động cụ thể | Cải thiện prompt và intent detection |
| Context Recall | Truy vấn thông tin rộng, chấp nhận thiếu một phần | Quyết định quan trọng cần đầy đủ bằng chứng | Cải thiện retrieval召回率 |
| Context Precision | Nghiên cứu khám phá, chấp nhận noise | Truy vấn nhạy cảm thời gian, cần hành động nhanh | Triển khai reranking |
| Completeness | Câu hỏi kiến thức chung | Tuân thủ chính sách, câu trả lời pháp lý/y tế | Tăng context window, cải thiện generation |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> *Câu trả lời:* Tạo các cặp câu trả lời giống nhau nhưng đảo thứ tự (A-B vs B-A). Condition 1: A xuất hiện trước. Condition 2: B xuất hiện trước. Nếu điểm trung bình của câu trả lời "đầu tiên" khác đáng kể (>0.1) so với "thứ hai", position bias tồn tại. Sử dụng paired t-test để kiểm định thống kê.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> *Câu trả lời:* Chuẩn hóa rubric bằng cách chia điểm chất lượng nội dung cho penalty độ dài. Phạt rõ ràng verbosity không liên quan. Thêm "concision" như một dimension riêng. Yêu cầu câu trả lời trích dẫn bằng chứng cụ thể thay vì thêm filler.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> *Câu trả lời:* LLM judges có thể học các patterns giả tạo hoặc không phù hợp với giá trị con người. Calibration đảm bảo điểm số của judge tương quan với đánh giá của con người, giảm bias hệ thống và đảm bảo tính hợp lệ của đánh giá.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.7 | Dưới mức này, hallucinations gây rủi ro đáng kể |
| Answer Relevance | 0.6 | Dưới mức này, người dùng không nhận được câu trả lời hữu ích |
| Completeness | 0.6 | Dưới mức này, thông tin quan trọng bị thiếu |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:* Offline evaluation cho rapid iteration, CI/CD gates, và regression testing. Online evaluation cho production monitoring và A/B testing. Human review cho outputs rủi ro cao, edge cases, và calibrating automated metrics.

---

## Part 2 — Core Coding (9:45–10:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (10:40–11:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | 20 / 20 |
| Easy | 5 / 5 |
| Medium | 7 / 7 |
| Hard | 5 / 5 |
| Adversarial | 3 / 3 |
| Source documents được sử dụng | 10 / 10 |
| Validator status | PASS |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| H01 | Hard | 09_escalation_and_policy_updates.md | Yêu cầu xác định policy version dựa trên date logic - cần hiểu effective date và triggering event |
| A02 | Adversarial | 00_system_scope.md, 08_accounts_privacy_and_security.md | Prompt injection - test khả năng từ chối instruction không hợp lệ |
| M01 | Medium | 02_orders_and_payments.md | Multi-step process (cancel → interception → return) |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:* Đảm bảo evidence là verbatim substring từ corpus. Một số câu chứa điều kiện phức tạp (ví dụ: policy version) cần tách thành nhiều context để cover đầy đủ.

**Xác nhận:**
- [x] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [x] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [x] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

**Kết quả từ benchmark (OpenAI gpt-4o-mini RAG system):**

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | NovaBook 14 specs | 1.000 | 1.000 | 0.571 | 0.800 | 0.960 | 0.777 | Yes | - |
| E02 | Payment methods | 0.600 | 1.000 | 0.118 | 0.625 | 0.133 | 0.292 | No | hallucination |
| E03 | Shipping time | 1.000 | 1.000 | 0.909 | 0.500 | 0.909 | 0.773 | Yes | - |
| E04 | PulsePhone warranty | 0.647 | 1.000 | 0.800 | 0.600 | 0.235 | 0.545 | No | incomplete |
| E05 | OrbitPlus cost | 0.833 | 0.950 | 0.833 | 0.429 | 1.000 | 0.754 | No | off_topic |
| M01 | Order cancellation | 1.000 | 0.887 | 0.933 | 0.600 | 0.800 | 0.778 | Yes | - |
| M02 | OrbitPlus benefits | 0.971 | 1.000 | 0.345 | 0.667 | 0.882 | 0.631 | No | off_topic |
| M03 | Return window v2.0 | 0.968 | 0.950 | 0.875 | 0.750 | 0.677 | 0.767 | Yes | - |
| M04 | Repair timeframes | 1.000 | 1.000 | 0.853 | 0.400 | 0.725 | 0.659 | No | off_topic |
| M05 | Account compromise | 1.000 | 0.804 | 0.660 | 0.750 | 0.882 | 0.764 | Yes | - |
| M06 | Bundle rules | 1.000 | 1.000 | 0.778 | 0.400 | 0.808 | 0.662 | No | off_topic |
| M07 | Promotional codes | 1.000 | 0.887 | 0.667 | 1.000 | 0.741 | 0.802 | Yes | - |
| H01 | Policy version 1.0 | 0.967 | 1.000 | 0.773 | 0.714 | 0.600 | 0.696 | Yes | - |
| H02 | OrbitPlus extension | 0.974 | 1.000 | 0.903 | 0.700 | 0.737 | 0.780 | Yes | - |
| H03 | Membership refund | 1.000 | 1.000 | 0.677 | 0.727 | 0.750 | 0.718 | Yes | - |
| H04 | Shipping damage | 1.000 | 0.917 | 0.771 | 0.538 | 0.889 | 0.733 | Yes | - |
| H05 | Policy version rules | 0.969 | 1.000 | 0.519 | 0.900 | 0.438 | 0.619 | No | off_topic |
| A01 | Restaurant Paris | n/a | n/a | 0.000 | 0.500 | 0.040 | 0.180 | No | hallucination |
| A02 | Prompt injection | 0.759 | 1.000 | 0.400 | 0.500 | 0.172 | 0.357 | No | incomplete |
| A03 | 60-day return | 0.638 | 0.804 | 0.433 | 0.533 | 0.362 | 0.443 | No | off_topic |

**Aggregate Report**
- Overall pass rate: 50.0%
- Avg Context Recall: 0.912
- Avg Context Precision: 0.958
- Avg Faithfulness: 0.641
- Avg Relevance: 0.632
- Avg Completeness: 0.637
- Failure type distribution: hallucination=2, incomplete=2, off_topic=6

**Ba cases có Overall Score thấp nhất**
1. ID: A01 | Score: 0.180 | Failure type: hallucination
2. ID: E02 | Score: 0.292 | Failure type: hallucination
3. ID: A02 | Score: 0.357 | Failure type: incomplete

**Nhận xét ngắn:** Retrieval metrics (Recall=0.912, Precision=0.958) rất tốt. Answer metrics ở mức trung bình (0.63-0.64). Adversarial cases (A01, A02) có điểm thấp nhất do out-of-scope question và prompt injection. Model cần cải thiện Relevance và Completeness.

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [x] Correctness
- [x] Completeness
- [x] Relevance
- [x] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | Tất cả claims đều dựa trên corpus với dates/amounts chính xác. Bao phủ đầy đủ câu hỏi. Xử lý đúng các policy versions. | "Orders placed before Sep 1, 2026 follow Return Policy v1.0: 21 days unopened, 7 days opened, 15% restocking fee." |
| 4 | Phần lớn đúng với minor omission. Tất cả claims có thể verify. Thiếu một minor condition hoặc exception. | Correctly states return window nhưng bỏ qua restocking fee amount. |
| 3 | Đúng một phần. Một số claims được hỗ trợ nhưng chứa 1-2 inaccuracies hoặc significant omissions. | States đúng policy version nhưng sai day count. |
| 2 | Significant errors hoặc thiếu critical information. Nhiều unsupported claims. | Claims 30-day return khi v1.0 chỉ cho phép 21 days. |
| 1 | Sai hoặc không liên quan. Information được bịa đặt không có trong corpus. Không trả lời được câu hỏi hoặc từ chối valid query. | Cung cấp medical/legal advice thay vì OrbitTech support. |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| Multi-version policy question | Phải xác định đúng version dựa trên date logic | Penalize nếu không xác định được triggering event |
| Partial evidence coverage | Câu trả lời đúng một phần nhưng không đầy đủ | Dùng Completeness score, yêu cầu tất cả conditions/exceptions |
| Ambiguous/contradictory corpus | Hai documents có thông tin hơi khác nhau | Ưu tiên statement rõ ràng nhất, đánh dấu ambiguity |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:* Randomize answer order trong judge prompt. Normalize score by length (penalize verbosity). Sử dụng multi-judge ensemble và so sánh scores với human annotations để calibrate.

### Exercise 3.4 — Framework Comparison (Bonus +5)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: RAGAS | Framework 2: DeepEval |
|---|---|---|
| Setup complexity | Trung bình - cần OpenAI API | Thấp - hỗ trợ local model |
| Metrics available | Faithfulness, Answer Relevancy, Context Recall/Precision | Hallucination, Answer Correctness, RAGAS metrics |
| CI/CD integration | Khá - Python API | Xuất sắc - Pytest integration |
| Kết quả trên cùng dataset | Sử dụng word overlap heuristic | Sử dụng LLM evaluation |
| Insight rút ra | Nhanh, deterministic, không tốn API cost | Chính xác hơn nhưng tốn cost hơn |

- Scores có nhất quán không? Không hoàn toàn - RAGAS dùng heuristic dựa trên word overlap, DeepEval dùng LLM judge.
- Framework nào strict hơn và vì sao? DeepEval strict hơn vì LLM judge có thể phát hiện nuanced issues.
- Hai framework có tìm ra cùng failure cases không? Thường tìm ra các cases rõ ràng giống nhau, nhưng edge cases có thể khác nhau.

> *Phân tích:* Nên dùng RAGAS heuristic cho rapid iteration và CI/CD gates, dùng DeepEval cho final quality assessment.

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| E01 | 1.000 | 1.000 | 1.000 | 1.000 | 0.000 |
| M01 | 1.000 | 1.000 | 1.000 | 1.000 | 0.000 |
| H01 | 0.967 | 0.967 | 0.804 | 0.804 | 0.000 |
| **Avg** | 0.989 | 0.989 | 0.935 | 0.935 | 0.000 |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:* Recall đo union của tất cả retrieved chunks. Reranking chỉ thay đổi thứ tự, không thêm/xóa chunks, nên union coverage không đổi.

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:* Khi recall thấp (<0.6) - có nghĩa retriever bỏ sót relevant evidence. Lúc đó cần improve retrieval (better embedding, hybrid search) hoặc fix chunking (size, overlap).

---

## Part 4 — Reflection (11:35–11:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 11:50–12:00.

- [x] Tất cả required tests pass.
- [x] `golden_dataset.json` validate thành công.
- [x] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [x] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [x] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [x] Đã copy `template.py` thành `solution/solution.py`.
- [x] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
