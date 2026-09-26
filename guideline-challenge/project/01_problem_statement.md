# Problem statement + downstream contract

## Bài toán

Xác định và gán nhãn phân đoạn **drivable area** (vùng mặt đường xe có thể di chuyển) trên ảnh thuộc tập dữ liệu chuẩn **BDD100K**, phân định rõ ràng giữa làn di chuyển trực tiếp của xe tự chủ (**direct**) và các làn đường thay thế hợp lệ (**alternative**), đồng thời bóc tách chuẩn xác ranh giới giữa mặt đường di chuyển với **vỉa hè, lề đường (curb/shoulder), dải phân cách, bãi đỗ xe hoặc vùng vạch kẻ mắt võng/chữ V (gore/hatched area)**.

**Mapping CVAT:**
Trong CVAT, nhằm tối ưu quy trình gán nhãn và tránh tạo nhiều class phức tạp, ta chuẩn hóa theo thuộc tính phân loại:

- Class CVAT: `area/driveable`.
- Semantic Type (bắt buộc qua attribute `area_type`):
  - **`direct`**: Khu vực làn xe hiện tại mà xe tự chủ (ego-vehicle) đang di chuyển trực tiếp bên trong (có quyền ưu tiên quỹ đạo cao nhất).
  - **`alternative`**: Khu vực các làn đường khác cùng chiều xe chạy mà xe tự chủ có thể chuyển làn sang hợp pháp mà không vi phạm luật giao thông.

## Downstream contract

1. **Downstream task / model / user là ai?**
   Dữ liệu được dùng để huấn luyện mô hình **Perception (Drivable Area Segmentation & Free-space Detection)** và cung cấp trực tiếp đầu vào cho module **Behavioral Planning & Trajectory Generation** (Lập kế hoạch quỹ đạo di chuyển và chuyển làn an toàn).

2. **Output annotation nào thực sự cần?**
   - **Geometry:** Polygon (hoặc multi-polygon khép kín).
   - **Class:** `area/driveable` (trong CVAT) - **Semantic:** `drivable_area`.
   - **Attributes cốt lõi:**
     - `area_type = direct | alternative` (Bắt buộc phân định rõ theo chuẩn BDD100K).
     - `state = clear | ambiguous`.
     - `needs_review = false | true`.
   - **Image-level tag:** `image_escalate` khi toàn ảnh bị suy giảm tầm nhìn nghiêm trọng (sương mù, chói lóa, ban đêm hoàn toàn không rõ làn) hoặc không thể xác định vị trí làn xe ego.
   - **Nguyên tắc hình học:** Chỉ gán nhãn trên phần bề mặt đường nhìn thấy được (**visible-only**), không phỏng đoán các vùng bị che khuất hoàn toàn bởi xe cộ hoặc vật cản tĩnh.

3. **Failure nào gây hậu quả lớn nhất?**
   - **False Positive (Vùng nguy hiểm nhất):** Gán vùng không thể đi được (vỉa hè, dải phân cách, làn ngược chiều có rào chắn, vùng công trường) thành `direct` hoặc `alternative`, gây nguy cơ va chạm nghiêm trọng cho xe tự hành.
   - **Misclassification Direct vs. Alternative:** Đánh nhầm làn xe đang đi (`direct`) thành `alternative` hoặc ngược lại, làm gián đoạn bộ lập kế hoạch kiểm soát làn hiện tại.
   - **False Negative:** Bỏ sót diện tích lớn mặt đường di chuyển hợp lệ làm hạn chế không gian tránh chướng ngại vật khẩn cấp.

4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?**
   - Ambiguity cục bộ (ví dụ: vạch phân cách giữa làn direct và alternative bị mòn/mất dấu): Gán `needs_review=true` và đặt `state=ambiguous`.
   - Ambiguity toàn ảnh (ví dụ: đường ngập nước, tuyết phủ trắng xóa không thấy tim đường): Gán tag `image_escalate`.
   - Toàn bộ các trường hợp nghi vấn sẽ được chuyển lên Lead Reviewer trong các phiên QA/QC.

## Scope

- **Trong scope (Bắt buộc label):**
  - **`direct`**: Làn đường hiện tại của ego vehicle, giới hạn bởi 2 vạch kẻ làn hai bên (lane markings) hoặc mép đường vật lý.
  - **`alternative`**: Các làn đường hợp pháp kề cận (cùng chiều), làn rẽ mở rộng, vùng nhập làn/tách làn cao tốc, mặt đường ngã tư trong phạm vi quỹ đạo hợp lệ.
  - Phần mặt đường nhìn thấy xung quanh các phương tiện đang lưu thông hoặc đang đỗ tạm thời.

- **Ngoài scope (Ignore / Không label):**
  - Vỉa hè (sidewalk), lối đi bộ riêng biệt.
  - Làn đường ngược chiều phân cách cứng hoặc vạch liền cấm lấn (trừ phi thiết kế cho phép chạy 2 chiều linh hoạt).
  - Vùng đỗ xe chuyên dụng nằm ngoài luồng giao thông (parking bay/slots), trạm xăng, đường cụt tư nhân.
  - Dải phân cách, rào chắn (guardrail), bồn cây, thảm cỏ, rãnh thoát nước.
  - Đảo giao thông vẽ bằng sơn kẻ gạch chéo (gore area / chevron markings).

- **Geometry tolerance:**
  - Đường biên polygon phải bám sát ranh giới vạch kẻ hoặc mép mép đường vật lý; sai lệch chấp nhận tối đa **<= 2-3 px**.
  - Tuyệt đối không để polygon ăn lấn sang vùng chướng ngại vật tĩnh hoặc vỉa hè.

## Output chấm được

Blind test sẽ đánh giá trực tiếp dựa trên:

- **CLASS & ATTRIBUTE:** Phân loại đúng `area/driveable` với `area_type` (`direct` vs `alternative`).
- **IGNORE:** Loại bỏ đúng các vùng ngoài scope.
- **AMBIGUITY FLAG:** Gán đúng `needs_review=true` và `state=ambiguous` tại các vùng tranh chấp/vạch mờ.
- **ESCALATE:** Đặt tag `image_escalate` chính xác khi có điều kiện thời tiết khắc nghiệt.
- **mIoU / BOUNDARY ACCURACY:** Đo lường độ trùng khớp hình học theo tiêu chuẩn IoU phân đoạn BDD100K.

## Dữ liệu và giới hạn

Sử dụng bộ dữ liệu **BDD100K** phân phối tại thư mục `data/bdd100k/` (tập con 26 ảnh đại diện cho các điều kiện ban ngày, ban đêm, mưa, ngược sáng, cao tốc và nội đô). Không nạp hoặc sử dụng thêm dữ liệu ngoài repository.
