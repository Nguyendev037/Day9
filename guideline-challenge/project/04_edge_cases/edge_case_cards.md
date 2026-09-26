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

- CVAT class: `road` (semantic: `drivable_area`).
- BDD100K attribute: `area_type` (`direct` | `alternative`).
- Phân định rõ ràng: `direct` (làn xe ego đang chạy) và `alternative` (làn cùng chiều hợp lệ kế cận/chuyển làn được).
- Không tự suy đoán phần bị che khuất hoàn toàn (visible-only).

---

CASE ID: EC01
Sample: BDD05
Scene: Đường cao tốc ngoại ô ban ngày, phía bên phải có vùng nhập/tách làn với đảo tam giác sơn vạch xương cá/vạch chéo màu trắng (gore/chevron area), tiếp giáp với hàng rào và công trình thép.
Observation: Vùng vạch chéo có cùng cốt cao độ và cùng chất liệu bê tông nhựa với mặt đường chính. Annotator rất dễ nhầm lẫn coi toàn bộ thảm nhựa là drivable area và vẽ polygon trùm qua cả vùng vạch chéo.
Decision: IGNORE
Expected: Polygon `drivable_area` (CVAT: `road`, `area_type=direct`) chỉ bao phủ làn đường xe đang chạy; vùng vạch chéo phân dòng (gore area) và dải phân cách mềm bên phải bị loại bỏ hoàn toàn (IGNORE).
Rationale: Theo luật giao thông và chuẩn BDD100K, vùng gạch chéo là vùng đệm an toàn cấm xe đè vạch/lưu thông. Gán vùng này là drivable area sẽ khiến bộ lập quỹ đạo điều khiển xe đâm vào vùng xung đột tách/nhập làn.
Common mistake: Vẽ một polygon duy nhất bao trùm cả làn đường và toàn bộ vùng vạch gạch chéo bên phải.
Diversity: conflict / critical / ambiguous semantics

---

CASE ID: EC02
Sample: BDD14
Scene: Đường cao tốc nhiều làn ban ngày, lưu lượng xe đông đúc; phía bên phải ngoài vạch sơn liền màu trắng là phần lề đường trải nhựa (paved shoulder) rất rộng tiếp tiếp giáp taluy cỏ.
Observation: Paved shoulder có bề mặt nhựa đường tương đồng với làn xe chính. Do không có gờ bê tông ngăn cách, người gán nhãn dễ nhầm shoulder là một làn xe bổ sung (`alternative`).
Decision: IGNORE
Expected: Polygon `road` (`area_type=direct`) bao phủ làn xe ego đang chạy; các làn bên trái gán `area_type=alternative`. Phần paved shoulder ngoài vạch liền trắng bên phải bị LOẠI BỎ (IGNORE).
Rationale: Lề đường khẩn cấp không phục vụ lưu thông xe bình thường. Bao gồm cả shoulder sẽ làm sai lệch không gian di chuyển hợp pháp của hệ thống tự hành.
Common mistake: Kéo polygon vượt qua vạch sơn liền trắng sang tận chân taluy cỏ bên phải.
Diversity: ambiguity / conflict

---

CASE ID: EC03
Sample: BDD10
Scene: Tuyến phố đô thị 2 chiều với vạch tim đường màu vàng đứt đoạn; hai bên lề đường có vạch sơn trắng phân định dải đỗ xe (parking bay) với nhiều ô tô đang đỗ sát curb.
Observation: Khu vực đỗ xe cùng bề mặt asphalt với lòng đường lưu thông. Khoảng trống giữa các xe đỗ hoặc phía sau đuôi xe có thể khiến annotator phân vân có nên khoét vào sát mép vỉa hè hay không.
Decision: IGNORE
Expected: `drivable_area` (CVAT: `road`, `area_type=direct`) giới hạn chuẩn xác bên trong vạch sơn trắng phân định làn xe chạy và vạch vàng tim đường; toàn bộ dải đỗ xe và các xe đỗ hai bên bị loại khỏi polygon. Làn đối diện bên trái vạch vàng không gán alternative (IGNORE).
Rationale: Downstream perception cần không gian hành lang giao thông thực tế. Việc mở rộng polygon vào các hốc đỗ xe sát vỉa hè gây nguy cơ xe tự hành lách sai làn và va chạm với xe đang đỗ.
Common mistake: Kéo polygon lượn lách vào các khoảng trống giữa các xe đỗ sát vỉa hè.
Diversity: ambiguity / conflict / critical

---

CASE ID: EC04
Sample: BDD02
Scene: Giao lộ đô thị lớn ban ngày, xe taxi vàng phía trước, mặt đường có cụm vạch đi bộ qua đường (crosswalk) màu trắng cỡ lớn nằm trước vạch dừng chờ đèn đỏ.
Observation: Các vạch kẻ crosswalk song song tạo các mảng tương phản trắng - đen xen kẽ, có thể khiến annotator nhầm lẫn cắt vụn polygon hoặc coi vạch sơn là vật cản không thể lái xe qua.
Decision: LABEL
Expected: Tạo polygon `road` (`area_type=direct` cho làn ego và `area_type=alternative` cho các làn cùng chiều kề bên) phủ liên tục xuyên suốt qua vạch dừng và toàn bộ cụm vạch kẻ crosswalk; cắt vòng quanh đuôi xe taxi vàng theo nguyên tắc visible-only.
Rationale: Vạch kẻ crosswalk là tín hiệu giao thông trên mặt đường, toàn bộ bề mặt này hoàn toàn cho phép xe lăn bánh qua.
Common mistake: Cắt đứt polygon trước crosswalk hoặc khoét lỗ rỗng theo các nan vạch kẻ vôi trắng.
Diversity: conflict / visible-only

---

CASE ID: EC05
Sample: BDD13
Scene: Đường đô thị một chiều nhiều làn, phía trước có xe bán tải trắng (pickup) và xe sedan bạc che khuất tầm nhìn, trên mặt đường có vạch sơn kẻ người đi bộ màu vàng đậm.
Observation: Thân xe bán tải và xe con che khuất hoàn toàn một khoảng mặt đường lớn ngay phía trước mũi xe ego. Annotator thường có xu hướng vẽ phỏng đoán phần đường tiếp nối phía trước mũi xe tải.
Decision: LABEL
Expected: Gán nhãn `area_type=direct` cho làn hiện tại và `area_type=alternative` cho làn bên cạnh; ranh giới polygon phải bo sát mép lốp và gầm xe nhìn thấy được (visible boundary). Tuyệt đối KHÔNG vẽ đè xuyên qua thân xe hoặc phỏng đoán vùng khuất. Vạch vàng crosswalk được phủ kín bình thường.
Rationale: Chuẩn annotation thị giác máy tính chỉ học trên điểm ảnh nhìn thấy (visible pixels). Phỏng đoán hình học che khuất sẽ làm sai lệch ground truth của mô hình phân đoạn.
Common mistake: Kéo polygon xuyên qua gầm và thân xe bán tải để nối liền dải đường phía xa.
Diversity: occlusion / truncation

---

CASE ID: EC06
Sample: BDD22
Scene: Đường cao tốc ngoại ô lúc hoàng hôn, đường thẳng tắp và thu hẹp dần về phía điểm tụ (vanishing point) ở chân trời xa xôi, có biển báo chỉ dẫn màu xanh bên phải.
Observation: Càng về xa, làn đường càng thu hẹp chỉ còn vài pixel bề rộng. Annotator dễ nản hoặc tự ý dừng polygon ở cự ly trung bình do khó đặt điểm chính xác.
Decision: LABEL
Expected: Tạo polygon `area_type=direct` (làn giữa/phải xe đang đi) và `area_type=alternative` bám theo các vạch sơn đứt đoạn, kéo dài liên tục về phía trước đến điểm tụ xa nhất mà mắt thường còn phân biệt được ranh giới mặt đường.
Rationale: Hệ thống tự hành cần nhận diện tầm xa (long-range perception) để giữ làn trên cao tốc vận tốc lớn. Cắt cụt polygon sớm sẽ làm mô hình mất khả năng dự đoán điểm tụ.
Common mistake: Dừng polygon đột ngột ở khoảng cách 30-50m trước xe vì cho rằng vùng xa quá hẹp.
Diversity: small_far / long_range

---

CASE ID: EC07
Sample: BDD21
Scene: Đường đô thị nhiều làn có dải hộ lan tôn sóng (guardrail) chạy dài bên phải ngăn cách với hành lang cây xanh và hàng rào lưới thép bên ngoài; phía trước có biển báo rẽ gấp màu vàng.
Observation: Phần hành lang phía sau guardrail có cốt nền bê tông/gạch trông bằng phẳng và có vẻ đi lại được. Ranh giới hộ lan có chân cột và bóng đổ phức tạp.
Decision: IGNORE
Expected: Polygon `road` (`area_type=direct` cho làn đang chạy, `area_type=alternative` cho các làn bên trái) kết thúc dứt khoát tại chân đế dải hộ lan guardrail. Toàn bộ khu vực phía sau hộ lan bị LOẠI BỎ (IGNORE).
Rationale: Hộ lan là rào cản vật lý tuyệt đối (hard physical barrier). Gán vùng phía sau vào drivable area là lỗi an toàn nghiêm trọng có thể dẫn đến va chạm chết người.
Common mistake: Thả điểm polygon vượt qua mép trên hộ lan hoặc bao trùm cả phần hành lang kỹ thuật bên ngoài.
Diversity: conflict / physical_barrier / critical

---

CASE ID: EC08
Sample: BDD23
Scene: Tuyến phố khu dân cư trời âm u sau mưa tuyết, kính lái đọng giọt nước mờ ảo, tuyết bẩn tích tụ thành các mảng sát mép vỉa hè (curb), hai bên có xe đỗ ken dày.
Observation: Nước đọng trên kính làm mờ hình ảnh; tuyết lẫn bùn đất che lấp hoàn toàn mép bó vỉa hè bê tông ở một số đoạn khiến không thể xác định ranh giới vật lý bằng mắt thường.
Decision: ESCALATE
Expected: Gán polygon `road` (`area_type=direct`) cho phần lòng đường nhựa ướt nhìn rõ; tại các đoạn ranh giới bị tuyết vùi lấp hoặc mép xe đỗ không chắc chắn, annotator bật thuộc tính `needs_review=true` và `state=ambiguous`.
Rationale: Khi hình ảnh thiếu bằng chứng thị giác trực tiếp do thời tiết, annotator không được đoán mò. Việc gắn cờ escalation giúp reviewer hiệu chuẩn lại ranh giới trong khâu QA.
Common mistake: Tự phỏng đoán vẽ đường thẳng xuyên qua các tảng tuyết bẩn sát lề đường.
Diversity: ambiguity / escalation / weather_adverse

---

CASE ID: EC09
Sample: BDD24
Scene: Đại lộ đô thị hẹp giữa các tòa nhà cao tầng, tuyết được ủi dồn lại thành dải đùn cao (snowbanks) chiếm dụng lòng đường; bên phải có xe bán đồ ăn (diner cart) và người đi bộ đông đúc.
Observation: Dải tuyết đùn làm co hẹp bề rộng làn xe di chuyển, ranh giới giữa mặt đường ướt và chân đống tuyết bị bùn lầy nhem nhuốc. Xe cộ đang xếp hàng di chuyển chậm.
Decision: ESCALATE
Expected: Polygon `area_type=direct` chỉ vẽ bám sát bề mặt đường nhựa đen nhìn thấy; tuyệt đối KHÔNG lấn vào đống tuyết đùn. Đặt `needs_review=true` cho polygon và gắn tag ảnh `image_escalate` nếu toàn bộ làn đường bị tuyết xâm lấn không rõ tim đường.
Rationale: Đống tuyết đóng băng là chướng ngại vật thực tế làm giảm không gian lưu thông. Gán tuyết đùn vào drivable area sẽ khiến xe tự hành lao vào đống tuyết gây kẹt hoặc lật xe.
Common mistake: Coi đống tuyết đùn là mặt đường di chuyển được chỉ vì nó nằm dưới lòng đường.
Diversity: ambiguity / escalation / obstacle_intrusion / critical

---

CASE ID: EC10
Sample: BDD18
Scene: Tuyến phố đô thị ban đêm có đèn đường, ở giữa đường có một làn xe buýt chuyên dụng sơn phủ màu đỏ gạch nổi bật kèm chữ sơn trắng lớn; hai bên đường có hàng loạt ô tô đang đỗ.
Observation: Làn xe buýt sơn đỏ có màu sắc tương phản dị biệt so với mặt đường nhựa xám đen xung quanh, dễ khiến người gán băn khoăn liệu đây có phải làn cấm (non-drivable) hay không.
Decision: LABEL
Expected: Gán nhãn polygon `road` phủ kín toàn bộ làn xe buýt sơn đỏ (`area_type=direct` nếu ego đang chạy trên đó hoặc `area_type=alternative` nếu ego ở làn bên cạnh). Không khoét lỗ hay tách vụn polygon theo chữ sơn trắng. Hàng xe đỗ hai bên bị loại bỏ.
Rationale: Làn xe buýt (bus lane) về mặt hình học và kết cấu hoàn toàn là bề mặt lưu thông cơ giới hợp pháp (drivable roadway), màu sơn chỉ quy định quyền ưu tiên làn xe.
Common mistake: Loại bỏ hoặc khoét rỗng phần đường sơn màu đỏ vì tưởng là khu vực cấm xe chạy.
Diversity: ambiguity / low_visibility / lane_marking_anomaly

---

CASE ID: EC11
Sample: BDD25
Scene: Đại lộ đô thị sầm uất lúc chập tối/đêm, mặt đường nhựa ướt sũng phản chiếu chói chang ánh đèn neon từ các tòa nhà, cửa hàng và đèn đuôi xe cộ.
Observation: Hiện tượng phản quang (reflection/glare) làm mặt đường loang lổ các vệt sáng vàng, đỏ, trắng, che khuất một phần các vạch sơn phân làn bên dưới.
Decision: LABEL
Expected: Polygon `road` (`area_type=direct` cho làn xe đang di chuyển, `area_type=alternative` cho các làn kề bên) phải phủ trùm liên tục qua các vệt phản chiếu ánh sáng trên mặt đường ướt; không cắt xẻ polygon theo bóng sáng phản quang. Nếu đoạn nào vạch kẻ bị chói mất dấu, đặt `state=ambiguous`.
Rationale: Mặt đường ướt đẫm nước vẫn là không gian di chuyển drivable area hợp chuẩn. Phản quang chỉ là hiệu ứng quang học, không phải vật cản vật lý.
Common mistake: Cắt đục lỗ polygon để né các vệt sáng đèn phản chiếu trên mặt đường nhựa ướt.
Diversity: low_visibility / reflection / glare

---

CASE ID: EC12
Sample: BDD04
Scene: Tuyến đường dốc khu dân cư ban ngày có vạch đôi màu vàng ở giữa tim đường; hai bên lề đường có nhiều xe ô tô đỗ dọc sát mép vỉa hè (curb), phía xa có xe buýt đang lên dốc.
Observation: Ranh giới mép đường vật lý (curb) rõ ràng nhưng bị gián đoạn bởi các thân xe đỗ; vạch đôi màu vàng chia tách 2 chiều lưu thông rõ rệt.
Decision: LABEL
Expected: Polygon `road` với `area_type=direct` phủ trọn vẹn làn đường bên phải vạch đôi vàng cho đến mép bánh xe đỗ hoặc curb hở; làn ngược chiều bên trái vạch đôi vàng KHÔNG gán alternative (loại bỏ hoặc gán opposite nếu có schema, ở chuẩn 2 nhãn thì IGNORE làn ngược chiều có vạch đôi liền).
Rationale: Vạch đôi vàng liền là vạch cấm lấn làn theo luật giao thông. Xe tự hành không thể coi làn đối diện là làn chuyển đổi hợp pháp (`alternative`).
Common mistake: Gán trùm toàn bộ mặt đường cả 2 chiều xe chạy thành một polygon duy nhất hoặc gán làn ngược chiều là `alternative`.
Diversity: conflict / lane_boundary / traffic_rule
