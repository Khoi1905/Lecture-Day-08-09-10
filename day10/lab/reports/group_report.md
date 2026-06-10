# Báo cáo nhóm - Lab Day 10

**Tên nhóm:** Lab team
**Thành viên:**
Trần Đức Đăng Khôi - 2A202600889
Lê Thiên Khang - 2A202600726
Nguyễn Thụy Như Quỳnh - 2A202600557
Phạm Thành Nam - 2A202600832
**Ngày nộp:** 2026-06-10
**Run tốt:** `final-clean-v3`
**Run inject:** `inject-bad`

## 1. Pipeline tổng quan

Pipeline đọc 247 dòng từ raw CSV, chuẩn hóa và phân loại thành 35 dòng cleaned cùng 212 dòng
quarantine. Quality gate chạy trước embed; nếu có expectation mức `halt`, publish bị dừng trừ kịch bản
inject có chủ đích. Chroma được publish theo snapshot bằng `chunk_id` upsert và prune ID cũ.
Manifest, log, cleaned/quarantine CSV và metadata vector đều mang `run_id`.

Lệnh tái lập:

```powershell
python etl_pipeline.py run --run-id final-clean-v3; python eval_retrieval.py --out artifacts/eval/after_fix_eval.csv; python grading_run.py --out artifacts/eval/grading_run.jsonl
```

## 2. Cleaning và expectation

Các mở rộng chính gồm đăng ký `access_control_sop`, validate `exported_at`, chặn HR stale theo nội dung,
chặn parser marker mơ hồ, sửa token lặp và thêm alias retrieval giữ nguyên nghĩa. Expectation mới kiểm tra
đủ canonical docs, chỉ có registered docs, `chunk_id` duy nhất, timestamp ISO và marker mơ hồ.

### Metric impact

| Rule / expectation         |                                Trước |                   Sau / inject | Chứng cứ                            |
| -------------------------- | -----------------------------------: | -----------------------------: | ----------------------------------- |
| access-control allowlist   |             8 dòng bị xem là unknown | 6 cleaned, 2 quarantine hợp lệ | cleaned/quarantine `final-clean-v3` |
| invalid `exported_at`      |      vi phạm contract có thể publish |              7 dòng quarantine | quarantine reason                   |
| stale HR content marker    |   6 dòng HR 2025 có ngày mới lọt qua |              6 dòng quarantine | quarantine reason                   |
| ambiguous parser marker    | nội dung low-confidence có thể embed |              5 dòng quarantine | quarantine reason                   |
| canonical docs expectation |   access source có thể thiếu im lặng |               PASS, missing=[] | log `final-clean-v3`                |
| refund stale expectation   |               inject có 2 violations |          clean có 0 violations | hai log run                         |

## 3. Before / after retrieval

Inject dùng `--no-refund-fix --skip-validate`, khiến hai chunk “14 ngày” được embed dù expectation halt.
Official grading đạt 8/10; `gq_d10_01` chạm forbidden context. Sau clean, snapshot prune vector stale,
grading đạt 10/10 và self-eval đạt 21/21 `contains_expected`, 0 forbidden. Alias P1 giải quyết vocabulary
mismatch giữa “tự động escalate” và “auto escalate” mà không thay đổi giá trị nghiệp vụ 10 phút.

## 4. Freshness và monitoring

SLA là 24 giờ tại publish boundary. `final-clean-v3` báo FAIL vì dữ liệu mẫu mới nhất là ngày
2026-04-11 trong khi chạy ngày 2026-06-10. Runbook quy định kiểm tra freshness/version trước volume,
schema, lineage và cuối cùng mới kiểm tra model/prompt.

## 5. Liên hệ Day 09

Retriever/agent Day 09 có thể dùng collection `day10_kb`. Quality gate bảo đảm agent không đọc vector
stale hoặc nguồn chưa đăng ký.

## 6. Rủi ro còn lại

- Chưa có freshness riêng cho ingest boundary.
- Alias retrieval cần catalog có version khi mở rộng production.
- Thông tin thành viên và individual report cần người nộp điền theo đóng góp thực tế.
