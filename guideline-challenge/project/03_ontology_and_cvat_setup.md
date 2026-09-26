# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây. Thay mọi
placeholder mới là xong (gate G2).

## Ontology table

| Name           | Geometry | Type (class / attribute) | Allowed values      | Default | Mutable? | Rationale                                                                                         |
| -------------- | -------- | ------------------------ | ------------------- | ------- | -------- | ------------------------------------------------------------------------------------------------- |
| area/driveable | polygon  | class                    | area/driveable      | -       | No       | Class chính cho vùng drivable area (mặt đường xe có thể di chuyển)                                |
| area_type      | -        | attribute                | direct, alternative | direct  | No       | Phân loại làn: `direct` (làn ego-vehicle đang chạy), `alternative` (làn kề cận cùng chiều hợp lệ) |
| state          | -        | attribute                | clear, ambiguous    | clear   | Yes      | Độ rõ ràng của ranh giới: `clear` (vạch rõ), `ambiguous` (vạch mờ/che khuất)                      |
| needs_review   | -        | attribute                | true, false         | false   | Yes      | Đánh dấu vùng có ambiguity, cần review thêm. Chỉ dùng `true` khi không thể quyết định chắc chắn   |

**Tag mức ảnh:**
| Name | Type | Rationale |
| ---- | -------- | --------- |
| image_escalate | tag | Dùng khi uncertainty ảnh hưởng toàn ảnh (sương mù, chói lóa, ban đêm không rõ làn, đường ngập nước, tuyết phủ...), không thể đưa ra annotation đáng tin cậy |

## Class hay attribute

- **Class `area/driveable`:** Label chính duy nhất, đại diện cho vùng drivable area (mặt đường di chuyển được). Mỗi vùng drivable liên tục là 1 instance (polygon).
- **Attribute `area_type`:** Phân tách loại làn theo chuẩn BDD100K. Giá trị mặc định `direct`. Thuộc tính bắt buộc, không thay đổi sau khi gán.
- **Attribute `state`:** Độ rõ ràng của ranh giới vạch kẻ/mép đường. Giá trị mặc định `clear`. Có thể cập nhật khi phát hiện vạch mờ/che khuất trong quá trình gán nhãn.
- **Attribute `needs_review`:** Đánh dấu vùng có ambiguity cục bộ, cần review thêm. Chỉ dùng `true` khi không thể quyết định ranh giới chắc chắn. Giá trị mặc định `false`. Có thể cập nhật khi phát hiện vùng tranh chấp.

Quy tắc: Dùng **class** khi object type khác nghĩa rõ rệt. Dùng **attribute** khi là thuộc tính của cùng object.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): `v2.74.1`
- **Tên task calibration** (có version guideline, ví dụ `team99-calib-v1`): team99-calib-v1
- **Guide của task đã dán `02_guideline.md`?** có

- **Nhóm dùng Shape hay Track, vì sao:** Dùng **Shape (Polygon)** vì task là ảnh tĩnh, không có temporal component. Vùng drivable area là vùng tĩnh trong mỗi ảnh.

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào
escalate. Ghi lại ai test và chỗ họ vấp:

- **Label cần dùng:** area/driveable (class)
- **Tool cần dùng:** Polygon
- **Attribute:** area_type (direct/alternative), state (clear/ambiguous), needs_review (true/false)
- **Tag:** image_escalate (khi ambiguity toàn ảnh)
- **Khi nào escalate:**
  - **Ambiguity cục bộ:** Gán `needs_review=true` + `state=ambiguous` cho vùng bị che khuất nặng (50-80%) hoặc vạch mờ.
  - **Ambiguity toàn ảnh:** Đặt **tag `image_escalate`** khi điều kiện thời tiết khắc nghiệt (mưa bão, sương mù, ban đêm chói sáng, đường ngập nước, tuyết phủ) không thể phân định làn.
  - Thể hiện bằng: Polygon + class `area/driveable` + attribute `area_type` + `state=ambiguous` + `needs_review=true` + **tag `image_escalate`** (nếu toàn ảnh)
- **Ai test:** Nguyễn Công Khải
- **Chỗ vấp:** Ban đầu không rõ cách xử lý reflection/glare. Sau khi đọc guideline mục 5, đã hiểu là **IGNORE** các vùng reflection.
