# Báo cáo cá nhân - Lab Day 10

**Tên:** Trần Đức Đăng Khôi - 2A202600889
**Vai trò:** Cleaning, Quality và Retrieval Evaluation  
**Run tham chiếu:** `final-clean-v3`, `inject-bad`

## Phần phụ trách

Tôi phụ trách phân tích `data/raw/policy_export_dirty.csv`, mở rộng
`transform/cleaning_rules.py`, bổ sung expectation trong `quality/expectations.py` và kiểm chứng retrieval.
Khi đối chiếu raw data với allowlist, tôi phát hiện `access_control_sop` là nguồn canonical hợp lệ nhưng
đang bị quarantine như nguồn lạ. Tôi thêm nguồn này vào allowlist và contract. Tôi cũng phát hiện rule HR
baseline chỉ dựa vào `effective_date`, vì vậy một số chunk chứa nội dung bản 2025 “10 ngày phép năm”
vẫn có thể lọt qua nếu mang ngày hiệu lực năm 2026.

Các rule mới có tác động đo được gồm: chặn `exported_at` không đúng ISO datetime, chặn HR stale theo
marker nội dung, quarantine chunk có prefix “Nội dung không rõ ràng”, sửa token “làm việc” bị lặp do sync,
và thêm alias truy vấn giữ nguyên nghĩa cho escalation P1. Kết quả run tốt là 247 raw records, 35 cleaned
records và 212 quarantine records. Trong quarantine có 7 dòng sai `exported_at`, 6 dòng HR stale theo
nội dung và 5 dòng parser mơ hồ.

## Quyết định kỹ thuật

Quyết định quan trọng là dùng mức `halt` cho các lỗi có thể làm agent trả lời sai nghiệp vụ hoặc làm
snapshot publish không đầy đủ. Các expectation `all_canonical_docs_present`, `only_registered_doc_ids`,
`unique_nonempty_chunk_ids` và `exported_at_iso_datetime` đều là halt. Marker parser mơ hồ được đặt mức
warn ở expectation, nhưng record đã được quarantine ở transform; cách này vừa chặn publish dữ liệu xấu
vừa cho phép theo dõi xu hướng parser. Publish dùng upsert theo `chunk_id` và prune ID ngoài snapshot.
Rerun `final-clean-v3` giữ collection ở đúng 35 vector, chứng minh không phình index.

## Sự cố và cách xử lý

Kịch bản inject `--no-refund-fix --skip-validate` tạo hai vi phạm stale refund. Expectation phát hiện đúng
nhưng pipeline ban đầu lỗi trên Windows vì log chứa ký tự mũi tên Unicode không mã hóa được bằng
`cp1252`. Tôi đổi thông báo vận hành sang ASCII để entrypoint chạy ổn định. Sau inject, grading chỉ đạt
8/10 và `gq_d10_01` có `hits_forbidden=true`. Sau khi chạy snapshot sạch, pipeline prune vector stale,
grading đạt 10/10.

Một lỗi khác là câu escalation P1 không lọt top-k dù source có “10 phút”, do câu hỏi dùng cụm
“auto escalate” còn source dùng “tự động escalate”. Tôi thêm alias giữ nguyên nghĩa tại transform và
đồng bộ self-eval với grading ở top-5. Kết quả cuối: self-eval đạt 21/21 `contains_expected`, không có
forbidden context.

## Cải tiến trong hai giờ tiếp theo

Tôi sẽ chuyển synonym/alias khỏi code sang catalog có version, thêm unit test cho từng quarantine reason,
và đo freshness ở cả ingest boundary lẫn publish boundary để phân biệt nguồn chậm với pipeline chậm.
