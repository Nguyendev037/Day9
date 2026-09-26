# Annotation guideline — Drivable Road Area

**Version:** v2

<!--

v2 = bản guideline sau calibration.

Mỗi lần tăng version phải ghi một dòng tương ứng vào:
project/08_revision_log.md

File này là tài liệu nhóm peer nhận nguyên văn trong blind pack và là nội dung
dán vào Guide của CVAT.

Peer KHÔNG nhận:
- edge_case_cards.md
- gold_decisions.csv
- sample_pack.csv

No hidden rules:
Mọi quy tắc mà annotator cần biết đều phải được viết trong guideline.
Quy tắc chỉ giải thích bằng miệng nhưng không được ghi ở đây được xem như không tồn tại.

Các ví dụ trong guideline chỉ được sử dụng ảnh thuộc split example hoặc calibration.
Không được sử dụng ảnh thuộc split blind làm ví dụ.

-->

## 1. Objective + scope

### Mục tiêu

Mục tiêu của bài annotation là xác định **vùng mặt đường nhìn thấy được dành cho
xe di chuyển bình thường** trong ảnh giao thông đường bộ.

`drivable_area` là phần mặt đường nhìn thấy được mà theo cấu trúc và chức năng
của cảnh giao thông, xe cơ giới thông thường được sử dụng để di chuyển.

Không được hiểu rằng mọi bề mặt được trải nhựa đều là `drivable_area`.

Đặc biệt, phải phân biệt giữa:

- roadway dành cho xe di chuyển;
- shoulder/lề đường;
- parking area;
- gore hoặc vùng gạch chéo;
- sidewalk;
- median;
- các vùng mặt đường khác nhưng không dành cho xe di chuyển bình thường.

### Phạm vi cần gán nhãn

Bắt buộc gán nhãn cho:

- làn đường xe cơ giới thông thường;
- nhiều làn xe thuộc cùng một roadway liên tục;
- làn rẽ;
- vùng nhập làn và tách làn thuộc roadway;
- phần roadway tại giao lộ;
- phần roadway có lane marking;
- phần roadway có crosswalk;
- phần roadway nhìn thấy xung quanh xe đang đỗ.

### Ngoài phạm vi

Không gán nhãn cho:

- vỉa hè;
- lối đi chỉ dành cho người đi bộ;
- cỏ, cây hoặc thảm thực vật;
- median/dải phân cách;
- barrier, guardrail hoặc vật chắn;
- đất, cát hoặc bề mặt không thuộc roadway;
- khu vực chỉ dành cho đỗ xe;
- shoulder/lề đường được nhận diện rõ là vùng ngoài roadway dành cho
  dừng khẩn cấp hoặc chức năng roadside;
- gore hoặc vùng gạch chéo không dành cho xe di chuyển bình thường.

### Nguyên tắc chính

**Bề mặt asphalt không đồng nghĩa với drivable area.**

Quyết định phải dựa trên vai trò của vùng đó trong cảnh giao thông, kết hợp
với các ranh giới vật lý và dấu hiệu giao thông nhìn thấy được.

Không quyết định chỉ dựa vào:

- màu sắc;
- texture;
- vật liệu;
- độ sáng;
- việc một chiếc xe có thể về mặt vật lý đi vào vùng đó.

---

## 2. Annotation unit

Đơn vị annotation là:

**một vùng drivable liên tục nhìn thấy được trong một ảnh tĩnh.**

Mỗi vùng drivable liên tục được annotate bằng:

`drivable_area`

và hình học:

`polygon`

### Tạo polygon mới khi

Tạo một polygon mới khi:

- xuất hiện một vùng drivable tách biệt về mặt không gian;
- hai vùng roadway bị ngăn cách bởi median;
- barrier hoặc guardrail;
- vùng gore hoặc vùng non-drivable rõ ràng;
- một vùng roadway khác không còn nối trực tiếp với vùng đang annotate.

### Không tạo polygon mới chỉ vì

Không tách polygon chỉ vì xuất hiện:

- vạch chia làn;
- mũi tên chỉ hướng;
- crosswalk;
- ký hiệu giao thông sơn trên mặt đường;
- thay đổi màu sơn.

Nếu các vùng đó vẫn là cùng một mặt đường drivable liên tục thì giữ trong
cùng một polygon.

### Task

Đây là task ảnh tĩnh.

Không sử dụng:

- tracking;
- track ID;
- temporal identity;
- mutable attribute theo thời gian.

---

## 3. Geometry rule

### 3.1. Loại hình học

Sử dụng:

**Polygon**

Không sử dụng:

- rectangle;
- bounding box;
- polyline;
- tag thay cho polygon.

### 3.2. Nguyên tắc biên

Polygon phải bám theo **ranh giới vật lý nhìn thấy được của roadway**.

Các ranh giới ưu tiên gồm:

- mép curb/bó vỉa;
- mép roadway;
- mép median;
- mép barrier hoặc guardrail;
- ranh giới rõ ràng giữa roadway và một bề mặt không drivable.

### 3.3. Visible geometry only

Chỉ annotate phần geometry được hỗ trợ bởi bằng chứng nhìn thấy trong ảnh.

Không được:

- kéo polygon xuyên qua vật thể che hoàn toàn;
- tự suy đoán hình dạng road phía sau xe;
- tự kéo polygon vào vùng tối không có đủ bằng chứng;
- tự dựng lại curb/road edge bị che hoàn toàn.

### 3.4. Lane marking

Vạch lane marking không tự động chia roadway thành hai vùng.

Nếu cả hai phía của vạch đều thuộc cùng roadway:

- giữ trong cùng polygon;
- không tạo lỗ;
- không cắt polygon theo đường sơn.

### 3.5. Crosswalk

Crosswalk là phần sơn nằm trên roadway.

Nếu mặt đường bên dưới là roadway:

- giữ crosswalk trong polygon;
- không tạo lỗ;
- không tách polygon chỉ vì có các vạch crosswalk.

### 3.6. Gore / hatched separator

Vùng gore hoặc vùng gạch chéo được dùng để tách hoặc phân luồng xe nhưng
không dành cho xe di chuyển bình thường phải được loại khỏi polygon.

Ví dụ các vùng gạch chéo nhìn thấy trong:

- `BDD01`;
- `BDD05`;
- `BDD16`.

Không được label toàn bộ vùng asphalt chỉ vì vùng đó nằm cạnh roadway.

### 3.7. Shoulder / lề đường

Một paved shoulder nằm ngoài active roadway phải được loại khỏi polygon khi
có đủ bằng chứng cho thấy đó là vùng shoulder.

Một dấu hiệu mạnh là:

- shoulder nằm ngoài mép roadway rõ ràng;
- có đường biên/solid edge line tách khỏi active lane;
- có cấu trúc roadside khác với traffic lane.

Ví dụ các cảnh highway như:

- `BDD03`;
- `BDD08`;
- `BDD09`;
- `BDD14`;
- `BDD19`.

Không mặc định rằng mọi phần asphalt ngoài lane đều là drivable.

Nếu không thể xác định chắc chắn roadway hay shoulder thì dùng escalation
ở Mục 7.

### 3.8. Geometry tolerance

Polygon phải bám sát ranh giới roadway.

Cho phép sai lệch nhỏ do thao tác đặt điểm, **không vượt quá khoảng 5 px theo
phương vuông góc với ranh giới tại vùng biên**, với điều kiện sai lệch không
làm polygon chuyển sang một vùng semantic khác.

Ví dụ:

- lệch 2–5 px trên curb nhưng vẫn ở phía roadway → có thể chấp nhận;
- ăn rõ vào sidewalk/parking/grass → không chấp nhận;
- bỏ sót đáng kể một phần roadway → không chấp nhận.

Nếu ranh giới thực tế không thể xác định trong ảnh, không cố gắng đạt tolerance
bằng cách đoán; sử dụng `needs_review`.

---

## 4. Taxonomy

Task sử dụng một class chính:

`drivable_area`

### Định nghĩa class

`drivable_area`:

> Vùng mặt đường nhìn thấy được và thuộc roadway dành cho xe cơ giới di chuyển
> bình thường.

### Attribute của polygon

`needs_review`

Giá trị:

- `false`
- `true`

Default:

`false`

Dùng:

`needs_review=true`

khi vùng đó vẫn có thể là roadway nhưng boundary hoặc semantic chưa thể xác
định chắc chắn từ bằng chứng trong ảnh.

### Image-level tag

`image_escalate`

Dùng khi uncertainty ảnh hưởng đến phần chính của ảnh hoặc toàn ảnh, khiến
không thể xây dựng annotation đáng tin cậy.

### Không tạo class phụ

Không tạo class riêng cho:

- sidewalk;
- parking;
- shoulder;
- median;
- crosswalk;
- lane marking;
- gore.

Các vùng này được xử lý bằng inclusion/exclusion và escalation.

### Không tạo class `unknown`

Không dùng một class riêng tên `unknown`.

Thay vào đó:

- ambiguity cục bộ → `needs_review=true`;
- ambiguity mức toàn ảnh → `image_escalate`.

Ontology chi tiết phải khớp với:

`03_ontology_and_cvat_setup.md`

---

## 5. Inclusion / exclusion

### 5.1. Bắt buộc label

Label:

- traffic lane;
- turn lane;
- merge lane;
- diverging roadway;
- intersection roadway;
- roadway có lane marking;
- roadway có crosswalk;
- roadway nhìn thấy xung quanh xe đang đỗ.

### 5.2. Sidewalk

Không label sidewalk.

Nếu curb nhìn thấy rõ:

- polygon dừng tại mép roadway;
- không ăn sang sidewalk.

### 5.3. Parking area

Không label một vùng chỉ dành cho parking.

Không sử dụng tiêu chí:

> “xe có thể chạy vào đây”

để quyết định.

Cần xem vùng đó có phải là phần roadway dành cho xe di chuyển bình thường
hay chỉ là vùng đỗ xe sát curb.

Các cảnh urban như `BDD04`, `BDD10`, `BDD12` và `BDD20` phải đặc biệt chú ý
đến ranh giới này.

Nếu parking-only area và roadway phân biệt rõ:

- roadway → LABEL;
- parking-only area → IGNORE.

Nếu ranh giới không rõ:

`needs_review=true`

### 5.4. Shoulder

Nếu shoulder được nhận diện rõ là phần ngoài roadway:

`IGNORE`

Không label toàn bộ asphalt từ curb/guardrail đến lane center chỉ vì nó cùng
màu hoặc cùng vật liệu.

Nếu không chắc roadway hay shoulder:

`needs_review=true`

### 5.5. Median

Median không phải drivable roadway.

Nếu median nằm giữa hai roadway:

- annotate roadway ở phía cần annotate;
- không annotate median.

### 5.6. Barrier / guardrail

Không annotate bản thân barrier hoặc vùng nằm phía ngoài barrier nếu vùng đó
không phải roadway.

Trong cảnh như `BDD21`, cần phân biệt roadway với vùng ở phía bên kia
guardrail.

### 5.7. Gore / hatched area

Gore hoặc vùng gạch chéo không dành cho xe di chuyển bình thường:

`IGNORE`

Các vạch trắng/vàng trong vùng này không biến vùng đó thành roadway.

### 5.8. Crosswalk

Crosswalk trên roadway:

`LABEL`

Không tạo lỗ trong polygon.

Các ảnh như `BDD02`, `BDD11`, `BDD12`, `BDD13` và `BDD18` là các ví dụ tốt
cho quy tắc này.

### 5.9. Xe đang đỗ

Xe đang đỗ không tự động biến roadway thành non-drivable.

Annotate phần roadway nhìn thấy được xung quanh xe.

Không annotate phần đường bị xe che hoàn toàn.

---

## 6. Visibility / occlusion

### 6.1. Bị che một phần

Nếu vẫn đủ bằng chứng để xác định roadway boundary:

- annotate phần visible;
- theo boundary nhìn thấy;
- không mở rộng polygon vào phần không chắc chắn.

### 6.2. Bị che hoàn toàn

Nếu phần roadway phía sau vật thể không thể xác định được:

- không tự vẽ phần bị che;
- chỉ annotate phần visible.

Nếu việc che khuất gây uncertainty đáng kể:

`needs_review=true`

### 6.3. Bị cắt bởi mép ảnh

Nếu roadway đi ra khỏi mép ảnh:

- annotate phần visible;
- polygon kết thúc tại image boundary;
- không đoán hình dạng ngoài ảnh.

### 6.4. Vùng xa / nhỏ

Nếu roadway nhỏ hoặc xa nhưng vẫn nhận diện rõ:

→ vẫn annotate.

Nếu quá nhỏ hoặc quá mờ để xác định boundary:

→ `needs_review=true`

### 6.5. Phản chiếu

Reflection không phải là một drivable road region mới.

Không tạo polygon dựa trên reflection.

### 6.6. Mặt đường ướt

Mặt đường ướt vẫn là roadway nếu cấu trúc đường xác định được.

Không thu nhỏ hoặc loại vùng chỉ vì:

- mặt đường có phản sáng;
- nước tạo reflection;
- màu mặt đường thay đổi.

`BDD17` và `BDD25` là các tình huống cần chú ý.

Nếu reflection/glare làm mất boundary:

`needs_review=true`

### 6.7. Tuyết

Tuyết không tự động làm roadway thành non-drivable.

Nếu ranh giới roadway vẫn nhìn thấy:

→ annotate.

Nếu tuyết che hoặc làm nhập ranh giới roadway với sidewalk/parking:

→ `needs_review=true`

Nếu ambiguity quá lớn để xác định vùng roadway chính:

→ `image_escalate`.

`BDD23` và `BDD24` là các edge case chính cho quy tắc này.

### 6.8. Ban đêm

Ban đêm không tự động làm ảnh thành invalid.

Nếu roadway vẫn nhận diện được:

→ annotate phần visible.

Không kéo polygon vào vùng tối chỉ vì dự đoán roadway có thể tiếp tục ở đó.

Nếu không xác định được boundary:

→ `needs_review=true`

Nếu uncertainty ảnh hưởng đến phần chính của ảnh:

→ `image_escalate`.

Các ảnh `BDD18`, `BDD25`, `BDD26` là các tình huống cần chú ý.

---

## 7. Ambiguity / escalation

Mọi ambiguity phải được thể hiện trực tiếp trong CVAT.

### LABEL

Dùng:

`drivable_area`

khi có đủ bằng chứng cho thấy vùng đó là roadway dành cho xe di chuyển
bình thường.

### IGNORE

Không tạo polygon khi vùng đó rõ ràng ngoài scope.

Ví dụ:

- sidewalk;
- grass;
- median;
- parking-only;
- clearly separated shoulder;
- gore/hatched separator;
- vùng bên ngoài guardrail không phải roadway.

### UNKNOWN

Không tạo class `unknown`.

Nếu không đủ bằng chứng cho một vùng cụ thể:

`needs_review=true`

### ESCALATE

#### 7.1. Ambiguity cục bộ

Dùng:

`drivable_area` + `needs_review=true`

khi:

- road và shoulder không phân biệt được;
- road và parking không phân biệt được;
- snow che curb;
- glare che road boundary;
- vật thể che mất boundary;
- vùng quá nhỏ hoặc quá mờ.

#### 7.2. Ambiguity toàn ảnh

Dùng:

`image_escalate`

khi:

- phần chính của roadway không thể xác định đáng tin cậy;
- phần lớn road boundary bị che;
- điều kiện ánh sáng/thời tiết làm mất thông tin cần thiết trên phạm vi lớn.

### Không được ép đoán

Khi evidence không đủ:

**Không đoán geometry chỉ để hoàn thành polygon.**

### Thứ tự quyết định

Annotator phải suy nghĩ theo thứ tự:

1. Vùng này có thuộc roadway không?
2. Nếu có, vùng visible nào thực sự drivable?
3. Boundary visible nằm ở đâu?
4. Có phần nào không chắc chắn không?
5. Nếu có, uncertainty là cục bộ hay toàn ảnh?

### Cách thể hiện trong CVAT

| Quyết định | Cách thể hiện |
|---|---|
| LABEL | Polygon `drivable_area` |
| IGNORE | Không tạo polygon |
| UNKNOWN cục bộ | `needs_review=true` |
| ESCALATE toàn ảnh | Tag `image_escalate` |
| Geometry | Boundary của polygon |

Quyết định chỉ giải thích bằng lời nhưng không xuất hiện trong CVAT export
được xem là không hợp lệ để chấm.

---

## 8. Temporal rule

**Không áp dụng — task ảnh tĩnh.**

Mỗi ảnh được annotate độc lập.

Không tạo track.

Không có temporal identity.

Không có mutable temporal attribute.

---

## 9. Examples

Các ví dụ bên dưới chỉ sử dụng các ảnh thuộc split `example` hoặc `calibration`.
Không được đưa các ảnh này vào split `blind` nếu vẫn giữ chúng làm ví dụ trong
guideline.

| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| BDD01 | Highway có vùng gạch chéo bên phải roadway | Polygon chỉ bao phủ roadway; loại vùng gore/hatched | Gore/hatched separator |
| BDD03 | Highway có active lanes và paved shoulder ở bên cạnh | Chỉ label phần active roadway; loại shoulder nếu boundary rõ | Roadway vs shoulder |
| BDD05 | Highway có vùng gạch chéo lớn cạnh làn xe | Không label vùng hatch/gore | Không phải mọi asphalt đều drivable |
| BDD10 | Đường đô thị có nhiều xe đỗ sát hai bên đường | Label roadway; không label vùng parking-only | Roadway vs parking |
| BDD11 | Giao lộ có crosswalk | Polygon roadway tiếp tục qua crosswalk | Crosswalk vẫn nằm trên roadway |
| BDD17 | Đường đô thị đang mưa, mặt đường phản sáng | Label roadway theo boundary nhìn thấy; không loại vì reflection | Wet road / reflection |
| BDD21 | Roadway có guardrail ngăn cách vùng bên cạnh | Chỉ label roadway ở phía xe di chuyển | Barrier / guardrail |
| BDD23 | Đường khu dân cư có tuyết ở hai bên, boundary khó nhìn | Label phần visible; dùng `needs_review=true` nếu boundary không chắc | Snow ambiguity |
| BDD24 | Đường đô thị có snowbank làm thu hẹp và che ranh giới | Label roadway nhìn thấy; không suy đoán phần bị tuyết che | Snow + occlusion |
| BDD18 | Đường đô thị ban đêm, nhiều vùng tối | Label visible roadway; không kéo polygon vào vùng không có bằng chứng | Night / low visibility |

### Positive example

Một highway có các lane rõ ràng và boundary roadway nhìn thấy được:

→ tạo `drivable_area` polygon.

### Negative example

Một sidewalk hoặc vùng grass cạnh roadway:

→ không tạo polygon.

### Edge case: shoulder

Nếu có active lane và paved shoulder được tách rõ:

→ label active roadway;

→ ignore shoulder.

### Edge case: parking

Nếu vùng sát curb là parking-only và được phân biệt rõ:

→ label roadway;

→ ignore parking area.

### Edge case: gore

Nếu vùng asphalt có các vạch chéo và đóng vai trò separator/gore:

→ ignore vùng đó.

### Edge case: crosswalk

Nếu crosswalk nằm trên roadway:

→ giữ toàn bộ roadway trong polygon;

→ không cắt crosswalk ra khỏi polygon.

### Edge case: snow

Nếu snow che road boundary:

→ annotate visible roadway;

→ `needs_review=true` nếu boundary không thể xác định chắc chắn.

### Edge case: night/glare

Nếu roadway còn nhìn thấy nhưng một phần boundary không rõ:

→ annotate phần có bằng chứng;

→ `needs_review=true` cho ambiguity cục bộ.

---

## 10. Common mistakes

### 1. Nghĩ rằng mọi asphalt đều là drivable

Sai vì shoulder, gore, parking hoặc vùng khác cũng có thể là asphalt.

Cách tránh:
xác định vai trò của vùng đó trong roadway.

### 2. Label cả shoulder

Sai khi phần ngoài active roadway đã được phân biệt rõ là shoulder.

Cách tránh:
tìm edge line, curb, barrier và cấu trúc roadside.

### 3. Label parking-only area

Sai khi vùng sát curb chỉ phục vụ parking.

Cách tránh:
phân biệt roadway với parking dựa trên cấu trúc và chức năng thể hiện
trong ảnh.

### 4. Cắt polygon theo lane marking

Sai vì lane marking không tự động chia roadway.

Cách tránh:
giữ trong cùng polygon nếu bề mặt roadway vẫn liên tục.

### 5. Cắt crosswalk ra khỏi roadway

Sai vì crosswalk vẫn nằm trên mặt đường.

Cách tránh:
giữ roadway liên tục qua crosswalk.

### 6. Label gore / vùng gạch chéo

Sai vì không phải mọi phần paved đều dành cho xe chạy.

Cách tránh:
loại vùng gore/hatched separator khi có bằng chứng rõ.

### 7. Vẽ xuyên qua xe

Sai vì phần roadway bị xe che hoàn toàn không có đủ evidence về geometry.

Cách tránh:
chỉ dùng visible geometry.

### 8. Đoán đường trong vùng tối

Sai vì roadway có thể tiếp tục, nhưng ảnh không cung cấp đủ geometry.

Cách tránh:
không suy đoán; dùng `needs_review` hoặc `image_escalate`.

### 9. Bỏ qua tuyết

Sai vì snow có thể che boundary road/sidewalk/parking.

Cách tránh:
dùng visible evidence và escalation khi cần.

### 10. Thấy reflection rồi thay đổi boundary

Sai vì mặt đường ướt vẫn có thể là roadway.

Cách tránh:
phân biệt reflection với ranh giới vật lý của roadway.

### 11. Dùng `image_escalate` cho mọi ambiguity

Sai vì ambiguity cục bộ và ambiguity toàn ảnh là hai trường hợp khác nhau.

Cách tránh:
- cục bộ → `needs_review=true`;
- toàn ảnh → `image_escalate`.

### 12. Ép annotator đoán khi thiếu evidence

Sai vì điều này làm tăng semantic disagreement.

Cách tránh:
dùng escalation thay vì tự suy đoán.

### 13. Dùng màu sắc làm tiêu chí chính

Sai vì asphalt, concrete và vùng có reflection có thể có màu gần nhau.

Cách tránh:
ưu tiên cấu trúc roadway, curb, barrier, lane arrangement và các dấu hiệu
hình học nhìn thấy được.

### 14. Cho rằng xe có thể đi vào thì vùng đó là drivable

Sai vì parking, shoulder hoặc một số paved roadside areas có thể vẫn không
thuộc roadway dành cho xe di chuyển bình thường.

Cách tránh:
xác định semantic role của vùng trong cảnh.

### 15. Rule trong đầu nhưng không ghi vào guideline

Sai vì peer không thể biết rule đó và rule không được xem là một phần của
specification.

Cách tránh:
mọi rule mà annotator cần để ra quyết định phải được ghi trong file này.