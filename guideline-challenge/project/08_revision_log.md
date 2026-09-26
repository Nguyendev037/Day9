# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Bản nháp đầu tiên | Khởi tạo guideline | - |
| v2 | Rõ ràng hóa rule đếm object và xử lý occlusion/reflection | Sau calibration phát hiện bất đồng về count (BDD05, BDD08) | BDD05 count (tai=2 vs others=3), BDD08 count (nguyen=2 vs others=1), calibration_report.csv dòng 1-4 |
| v2 | Thêm rule về minimum size (width >= 2m) cho drivable area | Loại trừ các vùng quá nhỏ gây confusion | BDD05, BDD08 |
| v2 | Standardize attribute format (không dùng pipe trong giá trị đơn) | Tránh lỗi parse trong export | BDD05 attr:needs_review, BDD08 attr:needs_review |
