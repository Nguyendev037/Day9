# Problem statement + downstream contract

## Bài toán

Xác định và gán nhãn **drivable area** (vùng mặt đường có thể lái xe) trên ảnh BDD100K, đặc biệt trong các tình huống ranh giới giữa **roadway với vỉa hè, bãi đỗ xe, lề đường, dải phân cách hoặc vùng gạch chéo** khó phân biệt.

**Mapping CVAT:** Trong CVAT, class được đặt tên là `road`, nhưng semantic là `drivable_area` theo định nghĩa của guideline. Peer cần ghi nhớ: **CVAT `road` = guideline `drivable_area`**.

## Downstream contract

1. **Downstream task / model / user là ai?**
   Annotation được sử dụng cho hệ thống **nhận thức cảnh giao thông đường bộ**, cần biết chính xác vùng không gian mặt đường mà xe có thể sử dụng để di chuyển.

2. **Output annotation nào thực sự cần?**

   * **Geometry:** polygon.
   * **Class:** `road` (trong CVAT) - **semantic: drivable_area** (theo guideline).
   * **Attribute:** `needs_review = false/true`, `state = __undefined__/clear/ambiguous`.
   * **Image-level tag:** `image_escalate` khi ambiguity ảnh hưởng đến toàn ảnh.
     Chỉ sử dụng hình học nhìn thấy được, không suy đoán phần drivable area bị che hoàn toàn.

3. **Failure nào gây hậu quả lớn nhất?**
   Lỗi nghiêm trọng nhất là **gán một vùng không drivable area thành drivable area hoặc bỏ sót một vùng drivable area lớn**, vì điều này làm sai không gian mà hệ thống nhận thức cho rằng xe có thể di chuyển.

4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?**

   * Ambiguity cục bộ: gán `needs_review=true` cho polygon bị ảnh hưởng.
   * Ambiguity ở mức toàn ảnh: gán tag `image_escalate`.
     Các trường hợp này được reviewer kiểm tra lại trong quy trình QA.

## Scope

* **Trong scope (bắt buộc label):** làn đường xe cơ giới, làn rẽ, vùng nhập/tách làn, phần roadway tại giao lộ, mặt đường có lane marking hoặc crosswalk và phần roadway nhìn thấy xung quanh xe đang đỗ.

* **Ngoài scope (ignore):** vỉa hè, lối đi bộ, cỏ/thảm thực vật, dải phân cách, barrier/guardrail, đất/cát, khu vực chỉ dành cho parking, shoulder khẩn cấp được phân biệt rõ và vùng gore/hatched không dành cho xe di chuyển bình thường.

* **Geometry tolerance:** polygon phải bám sát ranh giới vật lý nhìn thấy của drivable area; chấp nhận sai lệch nhỏ ở mức khoảng **2-3 px tại đường biên** do thao tác đặt điểm, nhưng không chấp nhận polygon ăn đáng kể sang vùng non-drivable area.

## Output chấm được

Blind test có thể chấm các quyết định:

* **LABEL:** class `road` (CVAT) = semantic `drivable_area` (guideline);
* **IGNORE:** không tạo polygon cho vùng ngoài scope;
* **UNKNOWN / ambiguity:** thể hiện bằng `needs_review=true`;
* **ESCALATE:** thể hiện bằng tag `image_escalate`;
* **GEOMETRY:** kiểm tra polygon có bám đúng ranh giới drivable area hay không.

Mọi quyết định phải được thể hiện trực tiếp trong file export CVAT; quyết định chỉ giải thích bằng lời nhưng không xuất hiện trong annotation sẽ không được chấm.

**Ghi chú:** tất cả reference đến `drivable_area` trong guideline tương đương với class `road` trong CVAT.

## Dữ liệu và giới hạn

Nguồn ảnh sử dụng là **BDD100K** trong `data/bdd100k/` của repo. Bộ này có **26 ảnh** đường phố và cao tốc với nhiều điều kiện như ban ngày, ban đêm, mưa, tuyết và các cảnh đô thị/highway. Nhóm dự kiến chọn một phần ảnh để làm `example`, `calibration` và `blind` theo `sample_pack.csv`. Không sử dụng dữ liệu ngoài repo.
