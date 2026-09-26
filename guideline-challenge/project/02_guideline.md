# Annotation guideline - Drivable Area Segmentation

**Version:** v2

<!--
v0 = chưa có bản nháp. Đổi dòng Version ở trên thành v1 khi xong bản nháp đầu, v2 sau calibration, v3 sau blind
handoff; mỗi lần tăng version ghi một dòng vào 08_revision_log.md. `make freeze` đòi v2 trở lên.

File này là thứ nhóm peer nhận nguyên văn trong blind pack và là Guide dán vào CVAT. Peer KHÔNG nhận
edge_case_cards.md, gold_decisions.csv hay sample_pack.csv. Rule nào peer cần biết phải nằm ở đây.
No hidden rules: rule chỉ giải thích bằng miệng thì coi như không tồn tại.
Ví dụ trong guideline chỉ dùng ảnh split example hoặc calibration, không dùng ảnh blind.

IMPORTANT MAPPING:
- CVAT class name: `road`
- Guideline semantic: `drivable_area`
- Peer MUST remember: In CVAT, use class `road`; in this guideline, we call it `drivable_area`.
  They are the same thing. This is to avoid re-labeling cost in CVAT.
-->

## 1. Objective + scope

Label **drivable area** (vùng mặt đường có thể lái xe) trên ảnh BDD100K để phục vụ **autonomous vehicle perception system**.

- **Trong scope (bắt buộc label):** Vùng drivable area (lane, làn đường chính)
- **Ngoài scope (IGNORE):** Vỉa hè, bãi đỗ xe, lề đường, vùng cỏ, vùng đất trống, reflection/glare

## 2. Annotation unit

- **Đơn vị:** Image-level (ảnh tĩnh)
- **Object type:** Region (polygon)
- **Instance rule:** Mỗi vùng drivable area **liên tục** là 1 instance. Không tách theo làn nếu cùng direction.
- **CVAT Class:** Use class **`road`** (this represents drivable_area semantic)

## 3. Geometry rule

- **Shape:** Polygon
- **Type:** Visible-only (chỉ vùng nhìn thấy)
- **Tolerance:** Polygon ôm **toàn bộ vùng drivable area nhìn thấy**, lệch <= 2px mỗi cạnh là chấp nhận
- **Endpoint rule:** Điểm polygon đặt tại ranh giới rõ ràng (vạch kẻ, mép đường)
- **CVAT Note:** All polygons must use class `road`

## 4. Taxonomy

Bảng đầy đủ ở `03_ontology_and_cvat_setup.md` - hai nơi phải khớp nhau.

| Name          | Type      | Allowed Values                  | Default       | Mutable | Rationale                                                                            |
| ------------- | --------- | ------------------------------- | ------------- | ------- | ------------------------------------------------------------------------------------ |
| drivable_area | class     | (mapped from CVAT class `road`) | -             | No      | **Semantic class:** Vùng đường dành cho xe di chuyển. **CVAT:** Sử dụng class `road` |
| state         | attribute | **undefined**, clear, ambiguous | **undefined** | Yes     | Trạng thái minh bạch của vùng                                                        |
| needs_review  | attribute | true / false                    | false         | Yes     | Không thể quyết định class                                                           |

## 5. Inclusion / exclusion

**Bắt buộc label (LABEL):**

- Vùng drivable area có **width >= 2m** (ước lượng bằng mắt)
- Vùng drivable area nhìn thấy **>= 50% diện tích** (kể cả bị che một phần)
- Vùng drivable area có **vạch kẻ rõ ràng**
- Vùng drivable area **không bị che 100%**

**Ignore:**

- Vỉa hè, lề đường, bãi đỗ xe
- Vùng **reflection/glare** (phản chiếu, loá sáng)
- Vùng **shadow** (bóng đổ) không phải drivable area thật
- Vùng **drivable area có width < 2m**

## 6. Visibility / occlusion

| Tình huống                            | Rule                               | Attribute         |
| ------------------------------------- | ---------------------------------- | ----------------- |
| **Clear** (nhìn thấy toàn bộ)         | LABEL                              | -                 |
| **Partial occlusion** (bị che 1-49%)  | LABEL                              | -                 |
| **Partial occlusion** (bị che 50-80%) | LABEL                              | needs_review=true |
| **Heavy occlusion** (bị che 81-99%)   | LABEL                              | needs_review=true |
| **Full occlusion** (bị che 100%)      | IGNORE                             | -                 |
| **Small-far** (nhỏ/xa, width < 2m)    | IGNORE                             | -                 |
| **Low visibility** (đêm, mưa, tuyết)  | LABEL nếu nhận diện được ranh giới | -                 |

## 7. Ambiguity / escalation

| Quyết định   | Khi nào                                                                 | Thể hiện trong CVAT                                                                 |
| ------------ | ----------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| **LABEL**    | Chắc chắn là drivable area                                              | Polygon + class **`road`** (semantic: drivable_area) + attribute phù hợp            |
| **IGNORE**   | Chắc chắn **không** phải drivable area                                  | Không vẽ                                                                            |
| **UNKNOWN**  | Không thể quyết định (thiếu bằng chứng)                                 | Polygon + class **`road`** + attribute `needs_review=true`                          |
| **ESCALATE** | Ambiguity quan trọng (ví dụ: không phân biệt được làn đường hay vỉa hè) | Polygon + class **`road`** + attribute `state=ambiguous` + **tag `image_escalate`** |

**Lưu ý:** Mọi quyết định phải nhìn thấy được trong file export CVAT.

## 8. Temporal rule

Không áp dụng - task ảnh tĩnh

## 9. Examples

| sample_id | Thấy gì                       | Expected output                                                       | Rule áp dụng                                   |
| --------- | ----------------------------- | --------------------------------------------------------------------- | ---------------------------------------------- |
| BDD01     | Đường cao tốc ban ngày, 2 làn | 2 polygon class **`road`** (semantic: drivable_area), không attribute | Mục 5, 6                                       |
| BDD05     | Đường có vùng nhỏ bị che      | 3 polygon class **`road`** (kể cả vùng bị che >=50%)                  | Mục 5 (width >= 2m), Mục 6 (partial occlusion) |
| BDD08     | Đường có vùng reflection      | 1 polygon class **`road`** (bỏ qua reflection)                        | Mục 5 (reflection -> IGNORE)                   |
| BDD26     | Đường đêm, visibility thấp    | 1 polygon class **`road`**, không attribute                           | Mục 6 (low visibility)                         |

**Ghi chú:** Trong tất cả ví dụ, class **`road`** trong CVAT tương đương semantic **`drivable_area`** trong guideline.

## 10. Common mistakes

| Lỗi                                   | Cách tránh                                          | Ví dụ         |
| ------------------------------------- | --------------------------------------------------- | ------------- |
| Bỏ sót vùng drivable area nhỏ         | Luôn check **width >= 2m**, kể cả bị che            | BDD05         |
| Đếm reflection/glare là drivable area | **IGNORE** reflection (mục 5)                       | BDD08         |
| Không nhất quán về `needs_review`     | Chỉ dùng `true` khi **không thể quyết định**        | BDD05, BDD08  |
| Format attribute không đúng           | Luôn export **giá trị đơn** (false), không pipe     | BDD05, BDD08  |
| Sử dụng class sai                     | Chỉ dùng class **`road`** (không có class nào khác) | Tất cả sample |
| Đặt nhầm semantic                     | **`road` in CVAT = `drivable_area` in guideline**   | Tất cả sample |
