# Edge-case library

Tối thiểu **8 card**, khuyến nghị 10-12. Một edge case tốt là case mà hai annotator hợp lý có thể làm khác nhau nếu
guideline chưa rõ. Tám ảnh dễ có label rõ ràng không được tính là edge-case library.

Cần có đủ độ đa dạng: occlusion / truncation / small-far · ambiguous semantics · conflicting road elements · **một case
critical-risk** · **một case guideline cho phép escalation**.

File này là kho nội bộ của nhóm, **không gửi cho peer**. Card dùng ảnh example/calibration thì chép rule + ví dụ sang
`02_guideline.md` (mục 7 và 9) để peer đọc được. Card về ảnh blind chỉ nằm ở đây, và decision của nó phải có trong
`gold_decisions.csv` trước `make freeze`.

`make status` đếm số dòng `CASE ID:` đã điền (đã thay placeholder). Copy khối dưới cho mỗi case.

---

**Mapping CVAT <-> Guideline:**
Trong CVAT, class được gọi là `road`. Trong guideline, semantic là `drivable_area`.
Tất cả các case dưới đây dùng semantic `drivable_area`, nhưng khi implement trong CVAT, dùng class `road`.

---

CASE ID: EC01
Sample: BDD05
Scene: Đường cao tốc có làn xe đang lưu thông và một vùng mặt đường lớn được kẻ các vạch chéo ở bên phải roadway.
Observation: Vùng kẻ chéo có cùng bề mặt asphalt với roadway và nằm sát làn xe nên có thể khiến annotator coi toàn bộ phần asphalt là drivable area. Tuy nhiên vùng này đóng vai trò là vùng phân tách/gore và không phải phần drivable area dành cho xe di chuyển bình thường.
Decision: IGNORE
Expected: `drivable_area` polygon (CVAT: class `road`) chỉ bao phủ phần drivable area dành cho xe di chuyển bình thường; vùng gore/hatched được loại khỏi polygon.
Rationale: Downstream cần phân biệt không gian thực sự dành cho xe di chuyển với phần asphalt dùng để phân tách dòng xe. Gán vùng gore thành drivable area sẽ làm mở rộng sai vùng không gian xe có thể sử dụng.
Common mistake: Gán toàn bộ vùng asphalt vào `drivable_area` (CVAT: `road`) hoặc tạo polygon bao phủ cả vùng gạch chéo.
Diversity: ambiguity / conflict / critical

---

CASE ID: EC02
Sample: BDD03
Scene: Đường cao tốc nhiều làn, có một phần paved shoulder nằm ngoài làn xe và được phân cách bởi đường biên màu vàng.
Observation: Shoulder có cùng màu và vật liệu với drivable area nên có thể bị nhầm. Ranh giới active drivable area và shoulder thể hiện bằng cấu trúc làn đường và đường biên.
Decision: IGNORE
Expected: `drivable_area` polygon (CVAT: class `road`) chỉ bao phủ các làn drivable area dành cho xe di chuyển; paved shoulder nằm ngoài drivable area được loại.
Rationale: Nếu label cả shoulder, diện tích drivable area bị mở rộng sang vùng ngoài active roadway và làm sai không gian chuyển động của xe.
Common mistake: Gán toàn bộ mặt asphalt từ mép đường đến barrier thành một polygon duy nhất.
Diversity: ambiguity / conflict

---

CASE ID: EC03
Sample: BDD10
Scene: Đường phố đô thị có nhiều xe đỗ sát hai bên đường và có các vùng sát curb dễ bị nhầm giữa drivable area và khu vực parking.
Observation: Khu vực đỗ xe nằm sát phần drivable area và có thể cùng bề mặt asphalt. Chỉ nhìn vào việc xe có thể physically đi vào khu vực đó là không đủ để kết luận đó là drivable area.
Decision: IGNORE
Expected: `drivable_area` polygon (CVAT: class `road`) bao phủ phần drivable area dành cho xe di chuyển; vùng chỉ dành cho đỗ xe được loại khỏi polygon.
Rationale: Downstream cần vùng drivable area thực sự phục vụ xe di chuyển bình thường, không phải mọi vùng mà xe có thể dừng hoặc đỗ.
Common mistake: Gán cả phần parking sát curb vào `drivable_area` (CVAT: `road`) chỉ vì xe có thể đi vào đó.
Diversity: ambiguity / conflict / critical

---

CASE ID: EC04
Sample: BDD02
Scene: Giao lộ đô thị có nhiều vạch crosswalk màu trắng phủ ngang mặt đường.
Observation: Các vạch crosswalk tạo thành các vùng sáng rõ trên mặt đường và có thể khiến annotator hiểu nhầm rằng các phần này không còn là drivable area.
Decision: LABEL
Expected: Tạo `drivable_area` polygon (CVAT: class `road`) liên tục trên phần drivable area và đi qua vùng crosswalk; không tạo lỗ theo các vạch crosswalk.
Rationale: Crosswalk là marking nằm trên drivable area. Việc loại crosswalk khỏi polygon sẽ làm sai hình học của mặt đường.
Common mistake: Cắt từng dải crosswalk ra khỏi polygon hoặc chia drivable area thành nhiều polygon chỉ vì có các vạch trắng.
Diversity: ambiguity / conflict

---

CASE ID: EC05
Sample: BDD13
Scene: Đường đô thị có nhiều phương tiện ở phía trước, trong đó xe tải và xe bán tải che khuất một phần mặt đường.
Observation: Một phần drivable area nằm phía sau hoặc giữa các phương tiện không nhìn thấy rõ. Annotator có thể có xu hướng kéo polygon qua vùng bị che dựa trên suy đoán về phần đường tiếp tục phía sau xe.
Decision: LABEL
Expected: Chỉ annotate phần `drivable_area` (CVAT: class `road`) được hỗ trợ bởi geometry nhìn thấy được. Không tự suy đoán hoặc kéo polygon xuyên qua vùng bị che hoàn toàn.
Rationale: Downstream sử dụng geometry của vùng nhìn thấy được. Suy đoán phần đường bị che có thể làm sai boundary và tạo ra annotation không có bằng chứng trực tiếp.
Common mistake: Vẽ polygon xuyên qua xe và tái tạo toàn bộ mặt đường phía sau xe dù phần đó không nhìn thấy.
Diversity: occlusion

---

CASE ID: EC06
Sample: BDD22
Scene: Đường cao tốc rộng, các làn đường tiến dần về phía điểm biến mất ở xa và phần drivable area ở xa có kích thước rất nhỏ trong ảnh.
Observation: Drivable area ở xa vẫn có thể nhận diện được nhưng rất hẹp về mặt pixel. Annotator có thể bỏ qua hoặc vẽ lệch đáng kể vì khó đặt các điểm polygon.
Decision: LABEL
Expected: Vẫn annotate phần drivable area (CVAT: class `road`) nhìn thấy được ở xa nếu boundary còn đủ rõ; polygon phải tiếp tục theo drivable area đến vùng có thể quan sát được.
Rationale: Downstream cần duy trì đầy đủ vùng drivable area nhìn thấy được, kể cả khi vùng đó nhỏ do phối cảnh.
Common mistake: Bỏ hoàn toàn phần drivable area ở xa hoặc tự mở rộng polygon vì cho rằng vùng nhỏ không cần annotate.
Diversity: small_far

---

CASE ID: EC07
Sample: BDD21
Scene: Đường nhiều làn có guardrail chạy dọc bên phải và một vùng không gian khác nằm phía sau guardrail.
Observation: Phần phía bên kia guardrail có thể vẫn trông giống bề mặt giao thông hoặc không gian có thể tiếp cận bằng xe, nhưng guardrail tạo ranh giới vật lý rõ ràng với drivable area đang được quan sát.
Decision: IGNORE
Expected: `drivable_area` polygon (CVAT: class `road`) kết thúc tại boundary drivable area phía trong guardrail; không mở rộng sang vùng phía sau barrier nếu vùng đó không phải drivable area đang sử dụng.
Rationale: Barrier là bằng chứng hình học mạnh cho thấy hai vùng không gian bị tách biệt. Gán cả vùng phía sau vào drivable area sẽ làm sai phạm vi drivable area.
Common mistake: Kéo polygon qua guardrail chỉ vì vùng phía sau vẫn có bề mặt đi lại được.
Diversity: conflict / ambiguous semantics

---

CASE ID: EC08
Sample: BDD23
Scene: Đường khu dân cư có tuyết và các xe đỗ dọc hai bên; ranh giới giữa drivable area và vùng sát curb bị tuyết che một phần.
Observation: Tuyết làm boundary giữa drivable area, curb và vùng đỗ xe khó xác định chính xác. Có thể xác định phần drivable area chính nhưng không chắc vị trí boundary ở một số đoạn.
Decision: ESCALATE
Expected: Tạo `drivable_area` (CVAT: class `road`) cho phần drivable area nhìn thấy được và đặt `needs_review=true` ở polygon bị ảnh hưởng nếu uncertainty chỉ ở cục bộ. Không tự suy đoán boundary bị tuyết che.
Rationale: Ép annotator chọn một boundary chính xác khi ảnh không cung cấp đủ bằng chứng sẽ tạo disagreement giữa các annotator. Downstream cần uncertainty được thể hiện rõ để reviewer xử lý.
Common mistake: Tự dựng curb/drivable area boundary hoàn toàn dựa trên suy đoán về vị trí của tuyết hoặc vị trí xe đỗ.
Diversity: ambiguity / escalation / occlusion

---

CASE ID: EC09
Sample: BDD24
Scene: Đường phố đô thị có tuyết chất thành dải lớn ở hai bên drivable area; một số phương tiện lớn ở phía phải che một phần vùng sát curb.
Observation: Snowbank làm drivable area hẹp lại về mặt nhìn thấy và che một phần boundary thật; đồng thời vật thể ở bên phải làm visibility giảm thêm. Annotator có thể khác nhau ở vị trí boundary giữa drivable area và vùng tuyết.
Decision: ESCALATE
Expected: Gán `drivable_area` (CVAT: class `road`) cho phần drivable area nhìn thấy được; nếu boundary drivable area/snowbank không xác định chắc chắn ở một đoạn, đặt `needs_review=true`. Không kéo polygon vào vùng tuyết chỉ vì phỏng đoán drivable area tiếp tục ở đó.
Rationale: Sai boundary trong case này có thể làm thay đổi đáng kể diện tích drivable area và downstream geometry.
Common mistake: Coi toàn bộ vùng có màu tối/ướt phía trong snowbank là drivable area hoặc tự suy đoán phần đường bị tuyết che.
Diversity: ambiguity / escalation / occlusion / critical

---

CASE ID: EC10
Sample: BDD18
Scene: Đường phố ban đêm, nhiều xe đỗ hai bên, phần drivable area phía trước có ánh sáng yếu và boundary một số đoạn không rõ.
Observation: Drivable area vẫn nhận diện được ở phần chính của ảnh nhưng các vùng tối ở xa có thể khiến annotator kéo polygon khác nhau.
Decision: LABEL
Expected: Annotate phần drivable area có đủ bằng chứng nhìn thấy; không mở rộng polygon vào vùng tối nơi boundary không được quan sát rõ. Nếu một đoạn boundary cụ thể không xác định được, dùng `needs_review=true`.
Rationale: Bóng tối không làm drivable area trở thành non-drivable. Tuy nhiên geometry không được suy đoán ngoài bằng chứng ảnh.
Common mistake: Kéo polygon vào toàn bộ vùng tối phía trước chỉ vì drivable area có khả năng tiếp tục ở đó.
Diversity: low_visibility / ambiguity / escalation

---

CASE ID: EC11
Sample: BDD25
Scene: Đường phố vào lúc chạng vạng/tối, mặt đường ướt và phản xạ mạnh ánh sáng từ đèn đường, đèn xe và các biển hiệu.
Observation: Reflection làm thay đổi màu và độ sáng của mặt đường; các vùng phản sáng có thể khiến annotator hiểu sai boundary hoặc thu nhỏ polygon.
Decision: LABEL
Expected: `drivable_area` (CVAT: class `road`) phải được xác định theo cấu trúc drivable area và boundary vật lý nhìn thấy; không loại các vùng drivable area chỉ vì có reflection. Nếu reflection che mất boundary, dùng `needs_review=true`.
Rationale: Wet road vẫn là drivable area. Downstream cần semantic drivable area boundary chứ không phải phân loại bề mặt dựa trên độ sáng hoặc màu phản xạ.
Common mistake: Thu nhỏ polygon để tránh vùng phản sáng hoặc coi reflection là boundary mới của drivable area.
Diversity: ambiguity / low_visibility

---

CASE ID: EC12
Sample: BDD04
Scene: Đường phố khu dân cư có curb rõ, xe đỗ dọc hai bên và các khoảng không gian sát mép đường.
Observation: Ranh giới giữa active drivable area, phần sát curb và vùng dùng cho đỗ xe nhìn khá gần nhau. Annotator có thể đặt boundary ở các vị trí khác nhau nếu chỉ nhìn vào vị trí của xe đỗ.
Decision: LABEL
Expected: Polygon bám theo boundary vật lý của drivable area; phần parking/roadside nằm ngoài drivable area không được đưa vào polygon.
Rationale: Đây là tình huống cần nhất quán về semantic của drivable area. Việc xe đỗ dọc đường không có nghĩa toàn bộ vùng sát xe đều là drivable area.
Common mistake: Lấy hàng xe đỗ làm boundary chính mà không kiểm tra curb và cấu trúc drivable area.
Diversity: ambiguity / conflict

---
