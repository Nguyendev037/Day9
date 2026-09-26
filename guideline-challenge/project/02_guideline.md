# Annotation guideline - Drivable Area Segmentation (BDD100K Standard)

**Version:** v2

<!--
v0 = chưa có bản nháp. Đổi dòng Version ở trên thành v1 khi xong bản nháp đầu, v2 sau calibration, v3 sau blind handoff; mỗi lần tăng version ghi một dòng vào 08_revision_log.md. `make freeze` đòi v2 trở lên.

File này là thứ nhóm peer nhận nguyên văn trong blind pack và là Guide dán vào CVAT. Peer KHÔNG nhận edge_case_cards.md, gold_decisions.csv hay sample_pack.csv. Rule nào peer cần biết phải nằm ở đây.
No hidden rules: rule chỉ giải thích bằng miệng thì coi như không tồn tại.

IMPORTANT MAPPING:
- CVAT class name: `road`
- Guideline semantic: `drivable_area`
- BDD100K Standard Core Attributes:
  + `area_type = direct | alternative`
  + `state = clear | ambiguous`
  + `needs_review = false | true`
-->

## 1. Objective + scope

Label **drivable area** (vùng mặt đường có thể lái xe) trên ảnh bộ dữ liệu **BDD100K** phục vụ **hệ thống nhận thức xe tự hành (Autonomous Vehicle Perception System)**.

- **Trong scope (bắt buộc label):** Làn đường xe ego đang vận hành (`direct`) và các làn đường hợp pháp cùng chiều có thể chuyển sang (`alternative`), bao gồm mặt đường tại ngã tư, vạch kẻ crosswalk, làn xe buýt chuyên dụng và mặt đường nhìn thấy quanh xe đỗ.
- **Ngoài scope (IGNORE):** Vỉa hè, lối đi bộ, dải phân cách, rào hộ lan (guardrail), lề đường (paved shoulder) ngoài vạch sơn liền, vùng vạch kẻ chéo phân tách dòng xe (gore/chevron area), vùng đỗ xe chuyên dụng, đất cát/thảm cỏ và các vật thể che khuất hoàn toàn.

---

## 2. Annotation unit & Taxonomy

- **Đơn vị:** Từng ảnh (Image-level).
- **Object type:** Region (Polygon khép kín).
- **Instance rule:** Mỗi làn đường hoặc khu vực mặt đường độc lập là 1 polygon riêng biệt:
  - Làn xe ego đang chạy gán `area_type=direct`.
  - Làn cùng chiều kế bên gán `area_type=alternative`.
- **CVAT Class:** Dùng class **`road`** (tương đương với semantic `drivable_area`).

### Bảng Taxonomy chuẩn

| Name               | Type      | Allowed Values                  | Default  | Bắt buộc | Rationale & Ý nghĩa                                                                                                                          |
| :----------------- | :-------- | :------------------------------ | :------- | :------- | :------------------------------------------------------------------------------------------------------------------------------------------- |
| **`road`**         | Class     | _(Mapped: `drivable_area`)_     | -        | Có       | Lớp đối tượng duy nhất cho mặt đường di chuyển được.                                                                                         |
| **`area_type`**    | Attribute | **`direct`**, **`alternative`** | `direct` | **Có**   | **Chuẩn BDD100K:**<br>• `direct`: Làn đường ego-car đang di chuyển trực tiếp.<br>• `alternative`: Làn cùng chiều hợp pháp có thể chuyển vào. |
| **`state`**        | Attribute | **`clear`**, **`ambiguous`**    | `clear`  | Có       | Thể hiện mức độ rõ ràng của ranh giới làn đường.                                                                                             |
| **`needs_review`** | Attribute | **`false`**, **`true`**         | `false`  | Có       | Đánh dấu vùng cần QA kiểm tra lại do tuyết phủ hoặc bị che khuất.                                                                            |

---

## 3. Geometry rules & Boundary Placement

- **Shape:** Polygon khép kín.
- **Type:** **Visible-only** (chỉ gán nhãn phần bề mặt đường thực sự nhìn thấy được qua camera; không suy đoán hoặc kéo polygon xuyên qua thân ô tô, xe tải hay vật cản).
- **Tolerance:** Polygon phải ôm sát ranh giới vật lý hoặc mép vạch kẻ đường, sai lệch cho phép `<= 2px` mỗi cạnh.
- **Phân tách làn (`direct` vs `alternative`):** Ranh giới giữa 2 polygon bám dọc theo tim vạch kẻ sơn chia làn; không được để hở khe trống hoặc chồng lấn (overlap) giữa các polygon.
- **Giao lộ / Crosswalk:** Polygon kéo dài liên tục qua các cụm vạch kẻ người đi bộ (crosswalk); không khoét rỗng theo các nan vạch sơn trắng/vàng.

---

## 4. Inclusion / Exclusion Matrix

| Tình huống / Đối tượng           | Quyết định | Thuộc tính áp dụng       | Hành vi gán nhãn                                     |
| :------------------------------- | :--------- | :----------------------- | :--------------------------------------------------- |
| Làn xe ego đang chạy             | **LABEL**  | `area_type=direct`       | Vẽ liên tục từ đáy ảnh hướng về điểm tụ xa nhất      |
| Làn kề cận cùng chiều            | **LABEL**  | `area_type=alternative`  | Tách riêng polygon theo từng làn xe                  |
| Làn xe buýt sơn đỏ               | **LABEL**  | `direct` / `alternative` | Vẽ trùm qua toàn bộ lớp sơn đỏ, không khoét chữ      |
| Vạch crosswalk ngã tư            | **LABEL**  | Theo làn di chuyển       | Vẽ phủ kín qua vạch người đi bộ                      |
| Mảng bê tông vá đường            | **LABEL**  | Theo làn di chuyển       | Vẽ trùm qua mảng vật liệu khác biệt                  |
| Vệt nước / phản quang đèn        | **LABEL**  | Theo làn di chuyển       | Không cắt xẻ hay đục lỗ polygon                      |
| Vùng vạch chéo gore/chevron      | **IGNORE** | -                        | Loại bỏ hoàn toàn khỏi polygon                       |
| Lề đường (paved shoulder)        | **IGNORE** | -                        | Không vẽ vượt ra ngoài vạch kẻ liền màu trắng        |
| Hộ lan (guardrail)               | **IGNORE** | -                        | Chặn ranh giới tại chân hộ lan, bỏ vùng phía sau     |
| Dải đỗ xe (parking bay) sát curb | **IGNORE** | -                        | Chặn ranh giới tại vạch sơn phân cách hoặc mép xe đỗ |
| Làn ngược chiều có vạch đôi vàng | **IGNORE** | -                        | Cấm lấn làn, không gán nhãn `alternative`            |

---

## 5. Visibility, Occlusion & Escalation

| Hiện trạng quan sát                    | Quyết định            | Attributes                                 | Image Tag                            |
| :------------------------------------- | :-------------------- | :----------------------------------------- | :----------------------------------- |
| Rõ ràng, vạch kẻ sắc nét               | LABEL                 | `state=clear`, `needs_review=false`        | Không                                |
| Bị xe che khuất một phần (1-80%)       | LABEL (visible-only)  | Bo sát mép vỏ xe, gầm xe nhìn thấy         | Không                                |
| Che khuất 100%                         | IGNORE                | Bỏ qua hoàn toàn, không phỏng đoán         | Không                                |
| Bị tuyết phủ bẩn / nước mưa làm mờ mép | LABEL (phần thấy rõ)  | `state=ambiguous`, `needs_review=true`     | Không                                |
| Đống tuyết đùn cao co hẹp lòng đường   | LABEL (phần nhựa đen) | `state=ambiguous`, `needs_review=true`     | `image_escalate` (nếu mất tim đường) |
| Mất hoàn toàn ranh giới toàn ảnh       | ESCALATE              | Gán polygon khả nghi + `needs_review=true` | **`image_escalate`**                 |

---

## 6. Examples (Minh họa quyết định chuẩn)

| Sample ID | Tình huống thực tế                                   | Expected Output                                                                          | Quy tắc áp dụng                             |
| :-------- | :--------------------------------------------------- | :--------------------------------------------------------------------------------------- | :------------------------------------------ |
| **BDD02** | Ngã tư đô thị có taxi vàng và vạch kẻ crosswalk lớn  | Polygon `direct` & `alternative` phủ liên tục qua crosswalk; cắt vòng quanh đuôi xe taxi | Mục 3 (crosswalk liên tục), Mục 4           |
| **BDD04** | Đường dốc 2 chiều vạch đôi vàng, xe đỗ hai bên       | 1 Polygon `area_type=direct` bên phải; IGNORE dải đỗ xe và làn ngược chiều               | Mục 4 (vạch đôi vàng cấm lấn làn)           |
| **BDD05** | Cao tốc có vùng vạch chéo gore area bên phải         | Polygon `area_type=direct` bên trong làn; loại bỏ (IGNORE) toàn bộ vùng vạch chéo        | Mục 4 (loại trừ gore area)                  |
| **BDD10** | Tuyến phố có dải vạch trắng đỗ xe (parking bay)      | Polygon `area_type=direct` bám vạch sơn làn; loại bỏ hoàn toàn các hốc đỗ xe             | Mục 4 (loại trừ bãi đỗ sát curb)            |
| **BDD13** | Đường phố có xe bán tải trắng che khuất tầm nhìn     | Polygon ôm sát bánh và đuôi xe bán tải; phủ trùm qua vạch crosswalk vàng                 | Mục 3 & 5 (visible-only, không vẽ xuyên xe) |
| **BDD14** | Cao tốc có paved shoulder rộng ngoài vạch liền trắng | Polygon bám mép trong vạch liền trắng; loại bỏ toàn bộ phần paved shoulder               | Mục 4 (loại trừ shoulder khẩn cấp)          |
| **BDD18** | Đêm đô thị có làn xe buýt sơn màu đỏ ở giữa          | Polygon phủ trùm toàn bộ làn sơn đỏ của xe buýt; không đục lỗ chữ                        | Mục 4 (bus lane là drivable area)           |
| **BDD21** | Đường có dải hộ lan tôn sóng (guardrail) bên phải    | Polygon kết thúc tại chân hộ lan; không mở rộng ra hành lang phía sau                    | Mục 4 (hộ lan là biên cứng)                 |
| **BDD22** | Cao tốc rộng hoàng hôn thu hẹp dần về điểm tụ xa     | Kéo dài polygon dọc các làn xe đến điểm tụ xa nhất còn phân biệt được                    | Mục 3 (nhận diện tầm xa)                    |
| **BDD24** | Phố hẹp có đống tuyết đùn cao co hẹp lòng đường      | Vẽ sát phần mặt đường lộ ra; gán `needs_review=true` và tag `image_escalate`             | Mục 5 (escalation do tuyết đùn)             |
| **BDD25** | Đêm mưa ướt, ánh đèn neon phản chiếu chói chang      | Polygon phủ qua các vệt sáng phản quang trên mặt đường ướt                               | Mục 4 (không né vùng phản chiếu)            |

---

## 7. Common mistakes & Quality checklist

| Lỗi thường gặp                                | Cách khắc phục                                                       | Mẫu minh họa     |
| :-------------------------------------------- | :------------------------------------------------------------------- | :--------------- |
| **Vẽ trùm lên vùng vạch chéo / gore area**    | Cắt bỏ vùng phân tách dòng xe ngoài vạch sơn biên                    | **BDD05**        |
| **Gộp cả lề đường khẩn cấp (paved shoulder)** | Chặn đường biên tại vạch kẻ liền màu trắng sát mép phải              | **BDD14**        |
| **Kéo polygon vào các ô đỗ xe sát vỉa hè**    | Giữ ranh giới thẳng theo vạch phân định làn đường                    | **BDD10**        |
| **Cắt vụn polygon tại vạch kẻ người đi bộ**   | Phủ kín qua vạch crosswalk (dù là vạch trắng hay vàng)               | **BDD02, BDD13** |
| **Vẽ xuyên qua thân xe phía trước**           | Bo sát mép cản sau, lốp và gầm xe nhìn thấy (visible-only)           | **BDD13**        |
| **Dừng polygon quá sớm trên cao tốc thẳng**   | Kéo dài liên tục theo phối cảnh tới điểm biến mất ở xa               | **BDD22**        |
| **Vượt qua dải hộ lan guardrail**             | Dừng điểm đặt polygon tại mép trong chân hộ lan                      | **BDD21**        |
| **Khoét rỗng làn xe buýt sơn đỏ**             | Phủ kín bề mặt làn bus sơn đỏ vì vẫn là không gian xe chạy           | **BDD18**        |
| **Cắt xẻ polygon để tránh vệt phản quang**    | Vẽ trùm qua vệt đèn phản chiếu trên mặt đường nhựa ướt               | **BDD25**        |
| **Gán làn ngược chiều thành alternative**     | Chỉ gán làn cùng chiều; làn ngược chiều có vạch đôi vàng phải IGNORE | **BDD04**        |
