# Annotation guideline - Drivable Area Segmentation (BDD100K Standard)

**Version:** v2

<!--
IMPORTANT MAPPING:
- CVAT class name: `road`
- Guideline semantic: `drivable_area`
- Mandatory Attribute: `area_type` phân tách giữa `direct` và `alternative`.
- Peer annotator bắt buộc chọn thuộc tính `area_type` cho từng polygon vẽ ra.
-->

## 1. Objective + scope

Hướng dẫn gán nhãn phân đoạn **drivable area** theo chuẩn bộ dữ liệu **BDD100K** trên ảnh thị giác máy tính xe tự hành. Nhiệm vụ chính là phân định ranh giới không gian di chuyển thành 2 loại làn đường:

- **`direct` (Làn điều khiển trực tiếp):** Làn đường mà xe tự chủ (ego-vehicle) đang vận hành bên trong.
- **`alternative` (Làn đường thay thế):** Các làn đường cùng chiều khác mà xe có thể chuyển làn sang hợp pháp.
- **Ngoài scope (IGNORE):** Vỉa hè, lề cỏ, dải phân cách, vạch mắt võng cấm đi, vùng đỗ xe chuyên dụng, vật thể che khuất hoàn toàn.

---

## 2. Annotation unit & Taxonomy

- **Đơn vị:** Từng ảnh (Image-level).
- **Object type:** Region (Polygon).
- **Instance rule:**
  - Vùng `direct` thường là 1 dải polygon liên tục dọc theo tầm nhìn của làn xe hiện tại.
  - Vùng `alternative` được vẽ riêng biệt cho từng làn kề bên hoặc gộp các dải liên tục cùng loại (ngăn cách bởi vạch phân làn).

### Bảng Taxonomy chi tiết

| Name               | Type      | Allowed Values                  | Default  | Bắt buộc | Rationale & Ý nghĩa                                                                                                           |
| :----------------- | :-------- | :------------------------------ | :------- | :------- | :---------------------------------------------------------------------------------------------------------------------------- |
| **`road`**         | Class     | _Mapped to `drivable_area`_     | -        | Có       | Lớp đối tượng tổng cho mặt đường di chuyển.                                                                                   |
| **`area_type`**    | Attribute | **`direct`**, **`alternative`** | `direct` | **Có**   | **Cốt lõi BDD100K:**<br>• `direct`: Làn ego car đang chạy.<br>• `alternative`: Làn kề cận cùng chiều được phép đi/chuyển vào. |
| **`state`**        | Attribute | **`clear`**, **`ambiguous`**    | `clear`  | Có       | Độ rõ ràng của vạch kẻ và ranh giới mép đường.                                                                                |
| **`needs_review`** | Attribute | **`false`**, **`true`**         | `false`  | Có       | Đánh dấu khi nghi ngờ ranh giới do bị che/mất vạch kẻ.                                                                        |

---

## 3. Geometry Rules & Ranh giới Direct vs. Alternative

1. **Vùng `direct`:**
   - Được giới hạn bởi vạch kẻ làn bên trái và bên phải của xe ego.
   - Bắt đầu từ phía dưới cùng của khung hình (mui xe/vị trí camera) kéo dài về phía trước điểm tụ (vanishing point).
   - Nếu làn rẽ mở rộng ngay trên làn hiện tại, giữ nguyên nhãn `direct`.

2. **Vùng `alternative`:**
   - Làn xe bên trái hoặc bên phải cùng chiều lưu thông.
   - Vùng nhập làn (merging lanes) hoặc tách làn (exit ramps) trước khi phân nhánh hoàn toàn.
   - Đường tránh, làn vượt cùng chiều hợp lệ.

3. **Quy tắc ranh giới hình học:**
   - **Visible-only:** Chỉ vẽ phần mặt đường nhìn thấy. Xe cộ, người đi bộ, rào chắn cắt ngang mặt đường -> Vẽ vòng qua mép ngoài của vật cản (trừ gầm xe nhìn xuyên thấy mặt đường rõ rệt).
   - **Độ lệch ranh giới:** Dung sai cho phép `<= 2px` so với mép vạch sơn hoặc mép gờ đường (curb).
   - **Vạch phân cách làn:** Ranh giới giữa polygon `direct` và polygon `alternative` bám dọc theo tim vạch sơn chia làn. Không để chồng lấn (overlap) giữa 2 polygon.

---

## 4. Inclusion / Exclusion Matrix

| Tình huống / Đối tượng            | Quyết định | Giá trị gán nhãn                   | Ghi chú                                                                |
| :-------------------------------- | :--------- | :--------------------------------- | :--------------------------------------------------------------------- |
| Làn đường xe ego đang chạy        | **LABEL**  | `area_type=direct`                 | Kéo dài tối đa đến khi tầm nhìn bị che khuất                           |
| Làn kề bên cùng chiều             | **LABEL**  | `area_type=alternative`            | Phân cách bởi vạch đứt hoặc vạch liền cho phép                         |
| Ngã tư / Giao lộ mở rộng          | **LABEL**  | `area_type=direct` / `alternative` | Khu vực xe chuẩn bị đi thẳng là `direct`, nhánh rẽ mở là `alternative` |
| Vạch mắt võng / Vùng đảo chevron  | **IGNORE** | -                                  | Không đi vào được theo luật                                            |
| Làn ngược chiều có dải phân cách  | **IGNORE** | -                                  | Không phải làn hợp lệ cho ego-vehicle                                  |
| Vỉa hè, thảm cỏ, rào hộ lan       | **IGNORE** | -                                  | Chướng ngại vật tĩnh ngoại vi                                          |
| Bãi đỗ xe bên đường (parking lot) | **IGNORE** | -                                  | Trừ khi là làn lưu thông chính xuyên qua bãi                           |
| Mặt đường có vũng nước/bóng râm   | **LABEL**  | Theo làn (`direct`/`alternative`)  | Vẫn là mặt đường di chuyển được                                        |

---

## 5. Visibility, Occlusion & Ambiguity Handling

| Hiện trạng quan sát                            | Hành động                         | Thuộc tính áp dụng                                  |
| :--------------------------------------------- | :-------------------------------- | :-------------------------------------------------- |
| Vạch kẻ rõ, tầm nhìn quang đãng                | Vẽ polygon chuẩn                  | `state=clear`, `needs_review=false`                 |
| Bị che khuất nhẹ bởi ô tô (1 - 49%)            | Vẽ bao quanh phần đường thấy được | `state=clear`, `needs_review=false`                 |
| Bị che khuất nặng (50 - 80%) / vạch mờ         | Vẽ phần thấy được                 | `state=ambiguous`, `needs_review=true`              |
| Bị che khuất hoàn toàn (100%)                  | **Bỏ qua (IGNORE)**               | Không phỏng đoán hình học                           |
| Tầm nhìn cực thấp (mưa bão, ban đêm chói sáng) | Không phân định nổi làn           | Gán polygon khả nghi + đặt tag **`image_escalate`** |

---

## 6. Examples (Minh họa quyết định)

| Sample ID    | Tình huống                                  | Expected Output                                                                                          | Ghi chú áp dụng                                          |
| :----------- | :------------------------------------------ | :------------------------------------------------------------------------------------------------------- | :------------------------------------------------------- |
| **BDD_EX01** | Cao tốc ban ngày 3 làn, xe đang ở làn giữa  | • 1 Polygon `area_type=direct` (làn giữa).<br>• 2 Polygon `area_type=alternative` (làn trái & làn phải). | Mục 2 & 3: Tách rõ 3 instance riêng biệt, không gộp làn. |
| **BDD_EX02** | Đường đô thị 1 làn mỗi hướng, xe đi thẳng   | • 1 Polygon `area_type=direct`.<br>• Làn đối diện bỏ qua (IGNORE) nếu có vạch đôi vàng liền.             | Không gán alternative cho làn ngược chiều cấm lấn.       |
| **BDD_EX03** | Có xe tải phía trước che khuất một phần làn | • Vẽ polygon `direct` bọc sát phần bánh/gầm xe tải nhìn thấy.                                            | Rule Visible-only (Mục 3).                               |
| **BDD_EX04** | Đoạn đường đang thi công có cọc tiêu        | • Phần làn bị rào chắn -> IGNORE.<br>• Phần hở còn lại xe chạy -> `direct` hoặc `alternative`.           | `state=ambiguous` nếu ranh giới tạm bợ.                  |

---

## 7. Common Mistakes & Quality Checklist

1. **Nhầm lẫn giữa `direct` và `alternative`:**
   - _Lỗi:_ Gán toàn bộ mặt đường thành 1 polygon `direct` duy nhất.
   - _Khắc phục:_ Phải tách biệt: Chỉ làn mà bánh xe ego-vehicle đang hướng chạy trên đó mới là `direct`. Các làn còn lại cùng chiều là `alternative`.
2. **Vẽ đè lên chướng ngại vật:**
   - _Lỗi:_ Kéo polygon xuyên qua thân xe khác đang dừng đèn đỏ.
   - _Khắc phục:_ Bắt buộc cắt khoét (cut-out) chướng ngại vật theo nguyên tắc _visible-only_.
3. **Lấn sang vỉa hè / curb:**
   - _Lỗi:_ Thả điểm polygon quá mép bó vỉa hè > 3px.
   - _Khắc phục:_ Zoom in 200-300% tại các góc bo cua để bo sát mép đường nhựa/bê tông.
4. **Quên cập nhật attribute `area_type`:**
   - _Lỗi:_ Để toàn bộ mặc định mà không kiểm tra xem làn kề bên có phải `alternative` hay không.
