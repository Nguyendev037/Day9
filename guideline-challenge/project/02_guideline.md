# Annotation guideline — Drivable Area Segmentation

**Version:** v2

<!--
v0 = chưa có bản nháp. Đổi dòng Version ở trên thành v1 khi xong bản nháp đầu, v2 sau calibration, v3 sau blind
handoff; mỗi lần tăng version ghi một dòng vào 08_revision_log.md. `make freeze` đòi v2 trở lên.

File này là thứ nhóm peer nhận nguyên văn trong blind pack và là Guide dán vào CVAT. Peer KHÔNG nhận
edge_case_cards.md, gold_decisions.csv hay sample_pack.csv. Rule nào peer cần biết phải nằm ở đây.
No hidden rules: rule chỉ giải thích bằng miệng thì coi như không tồn tại.
Ví dụ trong guideline chỉ dùng ảnh split example hoặc calibration, không dùng ảnh blind.
-->

## 1. Objective + scope

Label **drivable area** (vùng đường có thể lái xe) trên ảnh BDD100K để phục vụ **autonomous vehicle perception system**.

- **Trong scope (bắt buộc label):** Vùng đường có thể lái xe (lane, làn đường chính)
- **Ngoài scope (IGNORE):** Vỉa hè, bãi đỗ xe, lề đường, vùng cỏ, vùng đất trống, reflection/glare

## 2. Annotation unit

- **Đơn vị:** Image-level (ảnh tĩnh)
- **Object type:** Region (polygon)
- **Instance rule:** Mỗi vùng drivable **liên tục** là 1 instance. Không tách theo làn nếu cùng direction.

## 3. Geometry rule

- **Shape:** Polygon
- **Type:** Visible-only (chỉ vùng nhìn thấy)
- **Tolerance:** Polygon ôm **toàn bộ vùng drivable nhìn thấy**, lệch <= 2px mỗi cạnh là chấp nhận
- **Endpoint rule:** Điểm polygon đặt tại ranh giới rõ ràng (vạch kẻ, mép đường)

## 4. Taxonomy

Bảng đầy đủ ở `03_ontology_and_cvat_setup.md` — hai nơi phải khớp nhau.

| Name | Type | Allowed Values | Default | Mutable | Rationale |
|------|------|----------------|---------|---------|-----------|
| drivable_area | class | drivable | - | No | Class chính cho vùng đường có thể lái |
| occluded | attribute | full / partial / none | none | Yes | Mức độ bị che |
| needs_review | attribute | true / false | false | Yes | Không thể quyết định class |
| state | attribute | clear / ambiguous / __undefined__ | __undefined__ | Yes | Trạng thái minh bạch của vùng |

## 5. Inclusion / exclusion

**Bắt buộc label (LABEL):**
- Vùng đường có **width >= 2m** (ước lượng bằng mắt)
- Vùng đường nhìn thấy **>= 50% diện tích** (kể cả bị che một phần)
- Vùng đường có **vạch kẻ rõ ràng**
- Vùng đường **không bị che 100%**

**Ignore:**
- Vỉa hè, lề đường, bãi đỗ xe
- Vùng **reflection/glare** (phản chiếu, loá sáng)
- Vùng **shadow** (bóng đổ) không phải đường thật
- Vùng **drivable có width < 2m**

## 6. Visibility / occlusion

| Tình huống | Rule | Attribute |
|-----------|------|-----------|
| **Clear** (nhìn thấy toàn bộ) | LABEL | occluded=none |
| **Partial occlusion** (bị che 1-49%) | LABEL | occluded=partial |
| **Partial occlusion** (bị che 50-80%) | LABEL | occluded=partial, needs_review=true |
| **Heavy occlusion** (bị che 81-99%) | LABEL | occluded=partial, needs_review=true |
| **Full occlusion** (bị che 100%) | IGNORE | - |
| **Small-far** (nhỏ/xa, width < 2m) | IGNORE | - |
| **Low visibility** (đêm, mưa, tuyết) | LABEL nếu nhận diện được ranh giới | occluded=none hoặc partial |

## 7. Ambiguity / escalation

| Quyết định | Khi nào | Thể hiện trong CVAT |
|-------------|---------|---------------------|
| **LABEL** | Chắc chắn là drivable area | Polygon + class `drivable` + attribute phù hợp |
| **IGNORE** | Chắc chắn **không** phải drivable area | Không vẽ |
| **UNKNOWN** | Không thể quyết định (thiếu bằng chứng) | Polygon + class `drivable` + attribute `needs_review=true` |
| **ESCALATE** | Ambiguity quan trọng (ví dụ: không phân biệt được làn đường hay vỉa hè) | Polygon + class `drivable` + attribute `state=ambiguous` + **tag `ESCALATE`** |

**Lưu ý:** Mọi quyết định phải nhìn thấy được trong file export CVAT.

## 8. Temporal rule

Không áp dụng — task ảnh tĩnh

## 9. Examples

| sample_id | Thấy gì | Expected output | Rule áp dụng |
|-----------|---------|-----------------|--------------|
| BDD01 | Đường cao tốc ban ngày, 2 làn | 2 polygon `drivable`, `occluded=none` | Mục 5, 6 |
| BDD05 | Đường có vùng nhỏ bị che | 3 polygon `drivable` (kể cả vùng bị che >=50%), `occluded=partial` | Mục 5 (width >= 2m), Mục 6 (partial occlusion) |
| BDD08 | Đường có vùng reflection | 1 polygon `drivable` (bỏ qua reflection) | Mục 5 (reflection -> IGNORE) |
| BDD26 | Đường đêm, visibility thấp | 1 polygon `drivable`, `occluded=none` | Mục 6 (low visibility) |

## 10. Common mistakes

| Lỗi | Cách tránh | Ví dụ |
|-----|-----------|-------|
| Bỏ sót vùng drivable nhỏ | Luôn check **width >= 2m**, kể cả bị che | BDD05 |
| Đếm reflection/glare là drivable | **IGNORE** reflection (mục 5) | BDD08 |
| Không nhất quán về `needs_review` | Chỉ dùng `true` khi **không thể quyết định class** | BDD05, BDD08 |
| Format attribute không đúng (dùng `false|false`) | Luôn export **giá trị đơn** (false), không pipe | BDD05, BDD08 |
| Đếm thiếu/sai object | Đếm **tất cả** vùng drivable >= 50% diện tích nhìn thấy | BDD05 (tai thiếu 1), BDD08 (nguyen thừa 1) |
