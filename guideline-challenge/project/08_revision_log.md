# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Bản nháp đầu tiên | Khởi tạo guideline | - |
| v2 | Thống nhất semantic: guideline dùng `drivable_area`, CVAT class `area/driveable` | Mapping: CVAT `area/driveable` = guideline `drivable_area` | 03_cvat_labels.json (class=area/driveable), guideline (semantic=drivable_area) |
| v2 | Rõ ràng hóa rule đếm object và xử lý occlusion/reflection | Sau calibration phát hiện bất đồng về count (BDD05, BDD08) | BDD05 count (tai=2 vs others=3), BDD08 count (nguyen=2 vs others=1), calibration_report.csv dòng 1-4 |
| v2 | Thêm rule về minimum size (width >= 2m) cho drivable area | Loại trừ các vùng quá nhỏ gây confusion | BDD05, BDD08 |
| v2 | Standardize attribute format (không dùng pipe trong giá trị đơn) | Tránh lỗi parse trong export | BDD05 attr:needs_review, BDD08 attr:needs_review |
| v2 | Thống nhất CVAT class: `area/driveable` (sửa từ `road`) | Đồng bộ với 03_cvat_labels.json hiện tại | 03_cvat_labels.json, 02_guideline.md |
| v2 | Thêm rule minimum polygon complexity (≥ 10 điểm) | Calibration phát hiện bất đồng geometry: BDD05 (13-31 điểm), BDD10 (khác nhau) | 06_calibration_report.csv dòng 1-2, 06_calibration_measure.csv |
| v2 | Làm rõ rule ranh giới gore area | BDD05: các annotator vẽ khác nhau vùng vạch chéo; cần rule loại bỏ hoàn toàn gore area | 06_calibration_report.csv dòng 1, BDD05 |
| v2 | Làm rõ rule ranh giới parking bay | BDD10: bất đồng về boundary bãi đỗ; polygon phải dừng tại vạch sơn phân cách | 06_calibration_report.csv dòng 2, BDD10 |
| v2 | Thêm rule occlusion threshold (51-80% che → needs_review=true) | BDD13: xử lý occlusion xe bán tải khác nhau; cứu rõ ngưỡng che khuất | 06_calibration_report.csv dòng 3, BDD13 |
| v2 | Làm rõ rule guardrail boundary | BDD21: polygon phải kết thúc TẠI CHÂN hộ lan, không vượt qua | 06_calibration_report.csv dòng 5, BDD21 |
| v2 | Thêm rule cho mảng bê tông vá (surface transition) | BDD19: ranh giới mảng vá không rõ → gán state=ambiguous | 06_calibration_report.csv dòng 4, BDD19 |
| v2 | Cập nhật examples minh họa cho các rule mới | Thêm chi tiết vào bảng Examples (mục 6) và Common mistakes (mục 7) | 02_guideline.md mục 6 & 7 |
