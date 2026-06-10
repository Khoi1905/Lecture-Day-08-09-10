# Runbook - Lab Day 10

## Symptom

Agent trả lời “14 ngày” thay vì 7 ngày, trả lời HR 2025, thiếu access-control, hoặc retrieval không thấy
thông tin đã có trong source.

## Detection

- `expectation[...] FAIL (halt)` trong `artifacts/logs/run_<run_id>.log`.
- `hits_forbidden=yes` hoặc `contains_expected=no` trong CSV eval.
- `freshness_check=FAIL` trong log/manifest.
- Số vector khác `cleaned_records`.

## Diagnosis

| Bước | Việc làm | Kết quả mong đợi |
|---|---|---|
| 1 | Kiểm tra manifest và log theo `run_id` | count raw/clean/quarantine khớp |
| 2 | Nhóm quarantine theo `reason` | thấy failure mode upstream |
| 3 | Kiểm tra freshness trước model/prompt | xác nhận snapshot mới/cũ |
| 4 | Chạy `python eval_retrieval.py` | xác định query/slice lỗi |
| 5 | Chạy `python grading_run.py` | xác minh 10 tiêu chí chính thức |

Thứ tự debug: **Freshness/version -> Volume/errors -> Schema/contract -> Lineage/run_id -> model/prompt**.

## Mitigation

1. Dừng publish nếu expectation halt.
2. Sửa allowlist/rule/nguồn upstream, rồi chạy `python etl_pipeline.py run --run-id <new-id>`.
3. Pipeline upsert snapshot và prune vector stale; không sửa trực tiếp collection.
4. Chạy lại eval và quick check trước khi mở serving.

## Prevention

- Giữ owner, allowlist và canonical sources đồng bộ trong contract.
- Alert `#data-pipeline-alerts` khi freshness hoặc expectation halt fail.
- Duy trì golden eval và kịch bản inject trước mỗi thay đổi rule.
- Review quarantine reason tăng bất thường trước publish.

## Freshness status

- `PASS`: tuổi snapshot <= SLA 24 giờ.
- `WARN`: manifest thiếu/không parse được timestamp.
- `FAIL`: tuổi snapshot > SLA. Run `final-clean-v3` FAIL hợp lý vì snapshot mới nhất là
  `2026-04-11T00:00:00`, trong khi lab chạy ngày `2026-06-10`.
