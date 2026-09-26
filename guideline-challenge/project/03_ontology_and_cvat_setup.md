# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây. Thay mọi
placeholder mới là xong (gate G2).

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
| ---- | -------- | ------------------------ | -------------- | ------- | -------- | --------- |
| road | polygon | class | road | - | No | Class chính cho vùng đường dành cho xe di chuyển bình thường |
| state | - | attribute | __undefined__, red, yellow, green, off, unknown | __undefined__ | Yes | Trạng thái của vùng (áp dụng cho traffic light hoặc signal state) |
| needs_review | - | attribute | true / false | false | Yes | Dùng khi không thể quyết định class/region một cách chắc chắn |

**Tag mức ảnh:**
| Name | Type | Rationale |
| ---- | -------- | --------- |
| image_escalate | tag | Dùng khi uncertainty ảnh hưởng toàn ảnh, không thể đưa ra annotation đáng tin cậy |

## Class hay attribute

- **Class `road`:** Label chính duy nhất, đại diện cho vùng đường dành cho xe cơ giới di chuyển. Mỗi vùng road liên tục là 1 instance.
- **Attribute `state`:** Trạng thái của vùng (ví dụ: đèn giao thông). Giá trị mặc định `__undefined__`.
- **Attribute `needs_review`:** Đánh dấu vùng có ambiguity, cần review thêm. Chỉ dùng `true` khi không thể quyết định chắc chắn. Giá trị mặc định `false`.

Quy tắc: Dùng **class** khi object type khác nghĩa rõ rệt. Dùng **attribute** khi là thuộc tính của cùng object.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): `v2.74.1`
- **Tên task calibration** (có version guideline, ví dụ `team99-calib-v2`): team99-calib-v2
- **Guide của task đã dán `02_guideline.md`?** có

- **Nhóm dùng Shape hay Track, vì sao:** Dùng **Shape (Polygon)** vì task là ảnh tĩnh, không có temporal component. Vùng road là vùng tĩnh trong mỗi ảnh.

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào
escalate. Ghi lại ai test và chỗ họ vấp:

- **Label cần dùng:** road (class)
- **Tool cần dùng:** Polygon
- **Attribute:** state (__undefined__/red/yellow/green/off/unknown), needs_review (true/false)
- **Tag:** image_escalate (khi ambiguity toàn ảnh)
- **Khi nào escalate:** Khi không thể quyết định vùng đó có thuộc road hay không (ví dụ: ranh giới giữa làn đường và vỉa hè không rõ). Thể hiện bằng: Polygon + class `road` + attribute `needs_review=true` + **tag `image_escalate`**
- **Ai test:** Nguyen Huu Tai
- **Chỗ vấp:** Ban đầu không rõ cách xử lý reflection/glare. Sau khi đọc guideline mục 5, đã hiểu là **IGNORE** các vùng reflection.
