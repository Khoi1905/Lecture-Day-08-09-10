# Quality report - Lab Day 10

**Run tốt:** `final-clean-v3`  
**Run inject:** `inject-bad`  
**Ngày:** 2026-06-10

## 1. Tóm tắt số liệu

| Chỉ số | Inject bad | Sau fix |
|---|---:|---:|
| raw_records | 247 | 247 |
| cleaned_records | 35 | 35 |
| quarantine_records | 212 | 212 |
| Refund expectation halt | FAIL, 2 violations | PASS, 0 violations |
| Official grading pass | 8/10 | 10/10 |
| Self-eval contains expected | 20/21 (top-3 tại thời điểm inject) | 21/21 (top-5) |
| Self-eval hits forbidden | 1 | 0 |

## 2. Before / after retrieval

`gq_d10_01` trước fix có `contains_expected=true` nhưng `hits_forbidden=true` vì top-k còn “14 ngày”.
Sau fix: `contains_expected=true`, `hits_forbidden=false`, `top1_doc_matches=true`.

`gq_d10_09` và `gq_d10_10` sau fix đều đúng nguồn top-1; HR không còn marker “10 ngày phép năm”
và `access_control_sop` đã được publish.

Artifacts: `artifacts/eval/grading_inject_bad.jsonl`, `artifacts/eval/grading_run.jsonl`,
`artifacts/eval/after_inject_bad.csv`, `artifacts/eval/after_fix_eval.csv`.

## 3. Freshness

Freshness tại publish là FAIL: `latest_exported_at=2026-04-11T00:00:00` cũ hơn SLA 24 giờ.
Đây là lỗi dữ liệu mẫu có chủ đích, không phải lỗi chạy pipeline.

## 4. Corruption inject

Lệnh inject bỏ rule sửa refund và cho phép embed qua halt. Expectation phát hiện 2 vi phạm stale;
grading giảm từ 10/10 xuống 8/10 và câu refund chạm forbidden context.

## 5. Hạn chế

- Embedding multilingual nhỏ có ranking dao động; alias retrieval hiện được quản lý trong rule.
- Freshness mới đo publish boundary, chưa đo riêng ingest boundary.
