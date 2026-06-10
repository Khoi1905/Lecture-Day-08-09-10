# Data contract - Lab Day 10

Contract máy đọc được nằm tại `contracts/data_contract.yaml`. Owner: **AI Platform and Data Engineering**.

## 1. Source map

| Nguồn | Ingest | Failure mode chính | Metric / alert |
|---|---|---|---|
| `policy_refund_v4` | batch CSV từ policy export | chunk stale 14 ngày | `refund_no_stale_14d_window` halt |
| `hr_leave_policy` | batch CSV từ HR export | bản 2025 mang ngày mới | `hr_leave_no_stale_10d_annual` halt |
| `access_control_sop` | batch CSV từ IT Security | nguồn hợp lệ bị thiếu allowlist | `all_canonical_docs_present` halt |
| `sla_p1_2026` | batch CSV từ incident catalog | duplicate / vocabulary mismatch | dedupe count + eval P1 |
| `it_helpdesk_faq` | batch CSV từ helpdesk | duplicate / missing text | quarantine count |

## 2. Schema cleaned

| Cột | Kiểu | Bắt buộc | Quy tắc |
|---|---|---|---|
| `chunk_id` | string | Có | duy nhất, khóa upsert |
| `doc_id` | string | Có | thuộc 5 nguồn canonical |
| `chunk_text` | string | Có | dài >= 8, không mơ hồ/stale |
| `effective_date` | date | Có | `YYYY-MM-DD` |
| `exported_at` | datetime | Có | ISO datetime có ký tự `T` |

## 3. Quarantine và drop

Pipeline không drop im lặng. Dòng vi phạm được ghi vào `artifacts/quarantine/quarantine_<run_id>.csv`
kèm `reason`. Data Quality Owner xem reason, sửa upstream/contract, rồi chạy lại toàn bộ snapshot.
Không merge thủ công trực tiếp vào Chroma.

## 4. Phiên bản và canonical

Canonical refund là `data/docs/policy_refund_v4.txt`, cửa sổ hiện hành 7 ngày.
Canonical HR là `data/docs/hr_leave_policy.txt`, hiệu lực từ `2026-01-01`; nội dung “10 ngày phép năm”
là bản 2025 và luôn bị quarantine. `access_control_sop` là nguồn canonical thứ năm.
