# Annotation guideline — Drivable Road Area

**Version:** v1

<!--

v0 = chưa có bản nháp. Đổi Version thành v2 sau calibration và v3 sau blind handoff.
Mỗi lần tăng version phải ghi một dòng vào 08_revision_log.md.

File này là tài liệu nhóm peer nhận nguyên văn trong blind pack và là nội dung
dán vào Guide của CVAT.

Peer KHÔNG nhận:
- edge_case_cards.md
- gold_decisions.csv
- sample_pack.csv

No hidden rules:
Mọi quy tắc mà người annotator cần biết phải được viết trong guideline.
Quy tắc chỉ giải thích bằng miệng nhưng không được ghi trong guideline
được xem như không tồn tại.

Ví dụ trong guideline chỉ được sử dụng ảnh thuộc split example hoặc calibration.
Không sử dụng ảnh thuộc split blind làm ví dụ.

-->

## 1. Objective + scope

### Mục tiêu

Mục tiêu của bài annotation là xác định **vùng mặt đường có thể sử dụng cho xe
di chuyển bình thường** trong ảnh giao thông đường bộ.

`drivable_area` là phần **mặt đường nhìn thấy được** và có chức năng là phần
đường dành cho xe cơ giới di chuyển bình thường trong bối cảnh của ảnh.

Không được hiểu đơn giản rằng:

> mọi bề mặt được trải nhựa hoặc mọi nơi xe có thể đi vào đều là drivable.

Việc quyết định phải dựa trên **chức năng của bề mặt trong cảnh giao thông**.

### Phạm vi cần gán nhãn

Bắt buộc gán nhãn cho:

* làn đường xe cơ giới thông thường;
* nhiều làn đường cùng thuộc một mặt đường liên tục;
* làn rẽ;
* vùng nhập làn và tách làn;
* phần mặt đường tại giao lộ;
* mặt đường có vạch kẻ làn;
* mặt đường có vạch qua đường dành cho người đi bộ (crosswalk);
* phần mặt đường nhìn thấy xung quanh xe đang đỗ.

### Ngoài phạm vi

Không gán nhãn cho:

* vỉa hè;
* đường hoặc lối đi chỉ dành cho người đi bộ;
* cỏ, cây hoặc thảm thực vật;
* dải phân cách (median);
* lan can, hộ lan hoặc vật chắn;
* đất, cát hoặc bề mặt không thuộc mặt đường;
* khu vực chỉ dành cho đỗ xe;
* phần lề đường rõ ràng chỉ dành cho dừng khẩn cấp;
* vùng gore hoặc vùng gạch chéo (hatched area) không dành cho xe di chuyển bình thường.

### Nguyên tắc chính

**Mặt đường được trải nhựa không đồng nghĩa với vùng drivable.**

Khi quyết định một vùng có phải `drivable_area` hay không, ưu tiên vai trò của
vùng đó trong giao thông thay vì chỉ dựa vào màu sắc, vật liệu hoặc việc
một chiếc xe về mặt vật lý có thể đi vào đó hay không.

---

## 2. Annotation unit

Đơn vị annotation là **một vùng mặt đường drivable liên thông và nhìn thấy được
trong một ảnh tĩnh**.

Mỗi vùng drivable liên thông nhìn thấy được được gán bằng **một polygon**
mang class:

`drivable_area`

Tạo polygon mới khi:

* xuất hiện một vùng drivable tách biệt về mặt không gian;
* hai vùng đường bị ngăn cách bởi dải phân cách, vật chắn hoặc một vùng không
  drivable rõ ràng.

Không tạo polygon mới chỉ vì có:

* vạch chia làn;
* mũi tên trên đường;
* vạch crosswalk;
* màu sơn khác nhau;
* nhiều làn xe nhưng vẫn thuộc cùng một mặt đường liên tục.

Đây là **bài ảnh tĩnh**.

Không sử dụng tracking.

Không tạo track giữa các ảnh.

Không duy trì identity của một vùng đường qua nhiều frame.

---

## 3. Geometry rule

### Loại hình học

Sử dụng:

**Polygon**

Không sử dụng:

* rectangle;
* bounding box;
* polyline;
* tag thay cho polygon.

### Quy tắc đặt biên polygon

Biên polygon phải bám theo **ranh giới vật lý nhìn thấy được của mặt đường
drivable**.

Khi nhìn thấy rõ, ưu tiên các ranh giới như:

* mép bó vỉa;
* mép đường;
* mép dải phân cách;
* mép lan can hoặc hộ lan;
* ranh giới vật lý giữa mặt đường và vùng rõ ràng không drivable.

### Chỉ sử dụng hình học nhìn thấy được

Chỉ được sử dụng bằng chứng có trong ảnh.

Không được tự suy đoán hoặc vẽ thêm phần mặt đường bị che hoàn toàn bởi:

* xe;
* công trình;
* cây cối;
* tuyết;
* bóng tối;
* vật thể khác.

Nếu phần đường phía sau vật cản không thể xác định được chính xác, chỉ
annotate phần nhìn thấy và có bằng chứng đủ rõ.

### Vạch kẻ làn

Vạch kẻ làn **không mặc nhiên chia mặt đường thành các vùng drivable khác nhau**.

Nếu hai bên vạch đều thuộc cùng một mặt đường drivable liên tục:

* giữ cả hai phía trong cùng một polygon;
* không tạo lỗ theo vạch;
* không tách polygon chỉ vì có vạch chia làn.

### Crosswalk

Vạch crosswalk nằm trên mặt đường.

Không loại bỏ vùng crosswalk khỏi polygon chỉ vì nó có các vạch trắng hoặc
màu khác.

Nếu bề mặt bên dưới vẫn là mặt đường drivable thì polygon phải tiếp tục qua
vùng crosswalk.

### Vùng gore / gạch chéo

Vùng gore hoặc vùng gạch chéo được đánh dấu rõ và không dành cho xe di chuyển
bình thường phải được **loại khỏi polygon**.

Không được coi:

> "có trải nhựa"

là lý do đủ để gán nhãn.

### Chất lượng polygon

Các điểm của polygon phải được đặt gần ranh giới nhìn thấy được của mặt đường.

Tránh:

* polygon rộng hơn mặt đường;
* ăn sang vỉa hè;
* ăn sang cỏ;
* ăn sang bãi đỗ xe;
* các đoạn biên gấp khúc vô lý do đặt quá nhiều điểm;
* kéo polygon vào vùng không có bằng chứng.

Mục tiêu là tạo ra ranh giới nhất quán và bám sát **ranh giới vật lý nhìn thấy
được** của mặt đường.

---

## 4. Taxonomy

Bài toán chỉ sử dụng **một class chính**:

### Class

`drivable_area`

Ý nghĩa:

> Một vùng mặt đường liên thông, nhìn thấy được và dành cho xe cơ giới di chuyển
> bình thường.

### Attribute của polygon

`needs_review`

Giá trị cho phép:

* `false`
* `true`

Giá trị mặc định:

`false`

Đặt:

`needs_review=true`

khi ranh giới hoặc ý nghĩa drivable của vùng đó **không thể xác định chắc
chắn từ bằng chứng nhìn thấy trong ảnh**.

### Tag ở mức ảnh

`image_escalate`

Sử dụng khi sự không chắc chắn ảnh hưởng đến toàn bộ ảnh hoặc ảnh hưởng đến
một phần chính của cảnh khiến không thể đưa ra annotation đáng tin cậy.

### Unknown

Không tạo class riêng tên `unknown`.

Đối với vùng không chắc chắn cục bộ:

`needs_review=true`

Đối với trường hợp không chắc chắn ở mức toàn ảnh:

`image_escalate`

### Không tạo thêm class

Không tạo các class riêng cho:

* sidewalk;
* parking;
* shoulder;
* lane;
* median;
* crosswalk;
* road marking.

Các trường hợp trên được xử lý bằng quy tắc inclusion/exclusion và escalation.

Ontology đầy đủ phải được ghi thống nhất trong:

`03_ontology_and_cvat_setup.md`

Hai file phải khớp nhau.

---

## 5. Inclusion / exclusion

### Những trường hợp bắt buộc label

Gán nhãn cho:

1. Làn đường xe cơ giới thông thường.
2. Nhiều làn đường thuộc cùng một mặt đường liên tục.
3. Làn rẽ.
4. Vùng nhập làn hoặc tách làn thuộc mặt đường.
5. Phần mặt đường trong giao lộ.
6. Mặt đường có vạch kẻ làn.
7. Mặt đường có crosswalk.
8. Phần mặt đường nhìn thấy xung quanh xe đang đỗ.

### Những trường hợp phải ignore

Không gán nhãn cho:

1. Vỉa hè.
2. Lối đi chỉ dành cho người đi bộ.
3. Cỏ, cây hoặc thảm thực vật.
4. Dải phân cách.
5. Lan can, hộ lan hoặc vật chắn.
6. Đất hoặc cát ngoài mặt đường.
7. Khu vực chỉ dành cho đỗ xe.
8. Lề đường rõ ràng chỉ dành cho xe dừng khẩn cấp.
9. Vùng gore hoặc vùng gạch chéo không dành cho xe di chuyển bình thường.

### Khu vực đỗ xe

Không được coi một vùng là drivable chỉ vì xe **có thể về mặt vật lý đi vào hoặc
đỗ tại đó**.

Một khu vực **chỉ dành cho đỗ xe** được loại khỏi `drivable_area` khi cảnh cho
thấy đó là khu vực parking chứ không phải phần roadway dành cho xe di chuyển
bình thường.

### Lề đường / shoulder

Nếu lề đường được phân biệt rõ với roadway và có chức năng là khu vực dừng khẩn
cấp hoặc khu vực roadside riêng biệt:

`IGNORE`

Nếu không thể xác định chắc chắn đó là roadway hay shoulder:

áp dụng quy tắc escalation tại Mục 7.

### Crosswalk

Crosswalk không làm cho phần mặt đường bên dưới trở thành non-drivable.

Nếu crosswalk nằm trên roadway:

* vẫn giữ nó trong polygon;
* không tạo lỗ trong polygon;
* không chia polygon chỉ vì crosswalk.

### Xe đang đỗ

Xe đang đỗ không tự động biến mặt đường xung quanh nó thành non-drivable.

Gán nhãn phần mặt đường nhìn thấy được xung quanh xe.

Không suy đoán hình học chính xác của mặt đường bị xe che hoàn toàn.

---

## 6. Visibility / occlusion

### Bị che một phần

Nếu một phần mặt đường bị che nhưng vẫn có đủ bằng chứng để xác định vùng:

* annotate phần nhìn thấy được;
* bám theo ranh giới nhìn thấy;
* không tự vẽ phần bị che mà không có bằng chứng.

### Bị che hoàn toàn

Nếu phần mặt đường tiếp tục phía sau vật thể nhưng không thể xác định được
hình học chính xác:

* không kéo polygon xuyên qua vùng bị che;
* chỉ annotate phần nhìn thấy;
* dùng `needs_review=true` nếu phần bị che tạo ra sự không chắc chắn đáng kể
  đối với ranh giới.

### Bị cắt bởi mép ảnh

Nếu mặt đường tiếp tục ra ngoài mép ảnh:

* annotate phần nhìn thấy đến mép ảnh;
* không suy đoán hình dạng chính xác của phần nằm ngoài ảnh.

### Vùng nhỏ hoặc xa

Nếu vùng roadway vẫn xác định rõ dù ở xa:

* vẫn annotate;
* sử dụng đúng bằng chứng nhìn thấy.

Không được phóng đại hoặc tự thay đổi geometry chỉ vì vùng đó nhỏ.

Nếu vùng quá nhỏ hoặc quá mờ khiến không thể xác định ranh giới:

`needs_review=true`

### Phản chiếu

Không gán nhãn reflection như một vùng drivable riêng.

Chỉ gán nhãn mặt đường thật được hỗ trợ bởi cấu trúc của cảnh.

### Loá

Loá không tự động biến vùng đường thành non-drivable.

Nếu vẫn xác định được mặt đường:

→ annotate bình thường.

Nếu loá làm mất khả năng xác định ranh giới:

→ `needs_review=true`

hoặc `image_escalate` nếu ảnh bị ảnh hưởng ở mức toàn cảnh.

### Bóng tối / thiếu sáng

Không kéo polygon vào vùng tối mà không có bằng chứng.

Bóng tối không tự động làm mặt đường trở thành non-drivable.

Chỉ annotate phần có đủ bằng chứng nhìn thấy.

Nếu không xác định được ranh giới:

→ sử dụng escalation.

### Tuyết, mưa và điều kiện thời tiết

Tuyết, mưa, mặt đường ướt, sương mù hoặc điều kiện thời tiết khác không tự
động thay đổi semantic.

Dùng bằng chứng nhìn thấy trong ảnh.

Nếu điều kiện thời tiết che mất một ranh giới quan trọng:

`needs_review=true`

Nếu sự không chắc chắn ảnh hưởng đến phần chính của toàn ảnh:

`image_escalate`

---

## 7. Ambiguity / escalation

Mọi trường hợp không chắc chắn phải có **một biểu diễn nhìn thấy được trong CVAT**.

### LABEL

Sử dụng `drivable_area` khi bằng chứng trong ảnh cho thấy vùng đó là mặt đường
dành cho xe di chuyển bình thường.

### IGNORE

Không tạo annotation khi vùng đó rõ ràng nằm ngoài phạm vi drivable.

Ví dụ:

* vỉa hè;
* cỏ;
* dải phân cách;
* bãi đỗ xe chỉ dành cho parking;
* lối đi bộ;
* shoulder khẩn cấp được phân biệt rõ.

### UNKNOWN

Không được ép annotator phải đoán khi bằng chứng không đủ.

Không tạo class `unknown`.

Thay vào đó:

* ambiguity cục bộ → `needs_review=true`;
* ambiguity toàn ảnh → `image_escalate`.

### ESCALATE

Sử dụng escalation khi không thể đưa ra quyết định đáng tin cậy từ ảnh.

#### Ambiguity cục bộ

Dùng:

`drivable_area` + `needs_review=true`

Ví dụ:

* không rõ ranh giới road và shoulder;
* bó vỉa bị che;
* tuyết che mép đường;
* không rõ khu vực là roadway hay parking;
* loá làm mất ranh giới đường.

#### Ambiguity toàn ảnh

Dùng:

`image_escalate`

khi sự không chắc chắn ảnh hưởng đến phần chính của cảnh hoặc toàn ảnh,
khiến không thể xác định đáng tin cậy drivable road area.

### Quy tắc bằng chứng

Không được quyết định chỉ dựa vào:

* màu sắc;
* texture;
* độ sáng;
* giả định thông thường về đường;
* suy đoán về hướng camera.

Phải sử dụng cấu trúc và bằng chứng nhìn thấy được trong ảnh.

### Cách thể hiện trong CVAT

| Quyết định               | Cách biểu diễn trong CVAT     |
| ------------------------ | ----------------------------- |
| Vùng drivable            | Polygon class `drivable_area` |
| Không chắc chắn cục bộ   | Attribute `needs_review=true` |
| Không cần label          | Không tạo polygon             |
| Không chắc chắn toàn ảnh | Tag `image_escalate`          |

Mọi quyết định trên phải có thể nhìn thấy trong file export.

Không chấp nhận việc chỉ giải thích bằng miệng mà không thể hiện trong annotation.

---

## 8. Temporal rule

**Không áp dụng — task ảnh tĩnh.**

Mỗi ảnh được annotate độc lập.

Không sử dụng track.

Không tạo temporal identity giữa các ảnh.

Không có mutable temporal attribute.

---

## 9. Examples

Các ví dụ trong guideline chỉ được sử dụng ảnh thuộc split `example` hoặc
`calibration` trong `sample_pack.csv`.

**Không sử dụng ảnh thuộc split `blind`.**

| sample_id | Thấy gì                                               | Expected output                                                                   | Rule áp dụng                                 |
| --------- | ----------------------------------------------------- | --------------------------------------------------------------------------------- | -------------------------------------------- |
| BDD01     | Đường cao tốc, ranh giới roadway nhìn rõ              | Tạo polygon `drivable_area` bao phủ phần mặt đường nhìn thấy                      | Mặt đường rõ ràng thuộc scope                |
| BDD02     | Giao lộ đô thị có crosswalk                           | Tạo polygon đi qua vùng crosswalk                                                 | Crosswalk không loại mặt đường khỏi drivable |
| BDD05     | Vùng nhập/tách làn có phần gạch tách                  | Gán polygon cho roadway, loại vùng gore/gạch tách không dành cho xe               | Loại trừ gore/hatched separator              |
| BDD10     | Đường đô thị có xe đỗ và khu vực parking bên cạnh     | Gán polygon cho roadway, không gán khu vực parking-only                           | Phân biệt roadway và parking                 |
| BDD16     | Roadway có vùng phân cách được đánh dấu               | Gán polygon cho roadway, loại vùng phân cách                                      | Không phải mọi bề mặt asphalt đều drivable   |
| BDD20     | Đường khu dân cư, ranh giới road/parking khó xác định | Gán polygon theo ranh giới quan sát được; dùng `needs_review=true` nếu không chắc | Ambiguity road/parking                       |
| BDD21     | Roadway và một vùng bên cạnh được tách bởi guardrail  | Chỉ gán polygon cho roadway                                                       | Vùng bị ngăn cách không thuộc roadway        |
| BDD23     | Tuyết che một phần ranh giới đường                    | Gán phần nhìn thấy và dùng `needs_review=true` nếu ranh giới không chắc           | Visibility / weather ambiguity               |

### Ví dụ positive

Một mặt đường xe cơ giới nhìn rõ và có ranh giới vật lý xác định được:

→ tạo `drivable_area` polygon.

### Ví dụ negative

Một vỉa hè nằm cạnh roadway:

→ không tạo `drivable_area` polygon trên vỉa hè.

### Ví dụ edge case

Tuyết hoặc loá che mất ranh giới quan trọng của đường:

→ chỉ annotate phần có bằng chứng;

→ `needs_review=true` nếu ambiguity là cục bộ;

→ `image_escalate` nếu ambiguity ảnh hưởng ở mức toàn ảnh.

---

## 10. Common mistakes

### 1. Gán nhãn mọi bề mặt được trải nhựa

**Sai:** Gán cả parking, shoulder hoặc gore chỉ vì chúng đều là asphalt.

**Cách tránh:** Xác định vai trò của vùng đó trong giao thông.

### 2. Dùng rectangle thay cho polygon

**Sai:** Khoanh cả mặt đường bằng một bounding box.

**Cách tránh:** Luôn sử dụng polygon.

### 3. Cắt polygon theo vạch chia làn

**Sai:** Tạo các polygon riêng hoặc lỗ chỉ vì có vạch lane marking.

**Cách tránh:** Nếu hai phía vẫn thuộc cùng mặt đường drivable liên tục,
giữ trong cùng polygon.

### 4. Loại crosswalk khỏi drivable area

**Sai:** Cắt các vạch crosswalk ra khỏi polygon.

**Cách tránh:** Crosswalk vẫn nằm trên mặt đường; giữ toàn bộ roadway liên tục.

### 5. Gán nhãn sidewalk

**Sai:** Polygon ăn sang vỉa hè.

**Cách tránh:** Bám theo mép bó vỉa hoặc ranh giới vật lý giữa road và sidewalk.

### 6. Gán nhãn khu vực parking-only

**Sai:** Cho rằng parking là drivable vì xe có thể đi vào đó.

**Cách tránh:** Phân biệt khu vực dành cho parking với roadway dành cho xe di chuyển.

### 7. Vẽ xuyên qua vùng bị che

**Sai:** Tự kéo polygon phía sau một chiếc xe lớn hoặc vật cản.

**Cách tránh:** Chỉ sử dụng hình học nhìn thấy được.

### 8. Đoán ranh giới trong bóng tối hoặc tuyết

**Sai:** Vẽ ranh giới dù ảnh không đủ bằng chứng.

**Cách tránh:** Dùng `needs_review=true` hoặc `image_escalate`.

### 9. Không sử dụng escalation

**Sai:** Ép mọi ảnh phải có polygon hoàn toàn chính xác dù bằng chứng không đủ.

**Cách tránh:** Đánh dấu uncertainty bằng cơ chế CVAT được quy định ở Mục 7.

### 10. Tạo quá nhiều class

**Sai:** Tạo riêng class cho lane, sidewalk, parking, shoulder, median,
crosswalk, road marking.

**Cách tránh:** Chỉ sử dụng class `drivable_area`; các trường hợp khác xử lý
bằng inclusion/exclusion.

### 11. Áp dụng quy tắc không được ghi trong guideline

**Sai:** Thành viên trong nhóm biết một rule nhưng peer không thể tìm thấy rule
đó trong tài liệu.

**Cách tránh:** Nếu peer cần biết một rule để annotation, rule đó phải được
viết trực tiếp vào guideline.

### 12. Không nhất quán vị trí đường biên polygon

**Sai:** Một người bám theo curb, người khác lại lấy cả parking/sidewalk.

**Cách tránh:** Luôn bám theo ranh giới vật lý nhìn thấy được và dùng
`needs_review` khi bằng chứng không đủ.
