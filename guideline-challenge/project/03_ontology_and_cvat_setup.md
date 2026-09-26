# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây. Thay mọi
placeholder mới là xong (gate G2).

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale                                                                           |
| ---- | -------- | ------------------------ | -------------- | ------- | -------- | ----------------------------------------------------------------------------------- |
| road | polygon  | class                    |                |         |          | Vùng chính cần gán nhãn để biểu diễn phần mặt đường có thể di chuyển được trong ảnh |

## Class hay attribute

class `road` là label duy nhất, là object chính cần detect và cần geometry riêng

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): `v2.74.1`
- **Tên task calibration** (có version guideline, ví dụ `team07-calib-v1`): TODO
- **Guide của task đã dán `02_guideline.md`?** có (có / chưa)
- \*\*Nhóm dùng Track hay Shape, vì sao: Dùng Track vì gán nhãn ảnh 2D phục vụ bài toán nhận diện, thứ hai vùng drivable là vùng tĩnh

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào
escalate. Ghi lại ai test và chỗ họ vấp:

Label cần dùng: road
Tool cần dùng: Polygon
Attribute: không có
Khi nào escalate: khi không thể xác định rõ vùng đó có thuộc phần mặt đường drivable theo guideline hay không
