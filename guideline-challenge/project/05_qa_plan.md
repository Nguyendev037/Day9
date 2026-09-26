# QA Plan + Quality Gates

**Project:** Drivable Area Segmentation
**CVAT class:** `area/driveable`
**Guideline semantic:** `drivable_area`
**Guideline version:** v2 → v3 sau blind handoff
**Calibration status:** Hoàn thành với 5 samples, 4 annotators

QA được thiết kế để kiểm tra hai loại chất lượng chính:

1. **Semantic correctness:** annotator có quyết định đúng vùng `drivable_area` hay không.
2. **Geometry correctness:** polygon có bám đúng boundary của vùng drivable area nhìn thấy hay không.

QA ưu tiên các lỗi có thể làm downstream autonomous-vehicle perception system học sai vùng xe có thể di chuyển.

---

## 1. QA Flow

```text
Guideline v2
    ↓
Calibration
    ↓
Calibration review
    ↓
Freeze gold
    ↓
Blind handoff
    ↓
Peer annotation
    ↓
Score / GTS
    ↓
Root-cause analysis
    ↓
Guideline v3
    ↓
Final QA gate
```

### Nguyên tắc

* Annotator phải tự kiểm tra output trước khi export.
* QA không chỉ kiểm tra hình dạng polygon mà phải kiểm tra **semantic decision**.
* Critical / high-risk cases phải được kiểm tra trước random samples.
* Không tự suy đoán đối với case thiếu bằng chứng; sử dụng `needs_review` và `image_escalate` theo guideline.
* Mọi lỗi phải được phân loại thành:

  * `guideline_gap`
  * `data_ambiguity`
  * `execution_error`
* Nếu lỗi xuất phát từ guideline, phải cập nhật guideline và ghi vào `08_revision_log.md`.

---

# 2. Scope của QA

QA kiểm tra các thành phần sau:

| Thành phần     | Nội dung kiểm tra                                                                 |
| -------------- | --------------------------------------------------------------------------------- |
| Class          | Chỉ sử dụng CVAT class `area/driveable`                                          |
| Semantic       | `area/driveable` phải đại diện cho `drivable_area`                              |
| Geometry       | Polygon bao phủ vùng drivable area nhìn thấy                                      |
| Instance       | Mỗi vùng drivable area liên tục là một polygon                                    |
| Inclusion      | Vùng đủ bằng chứng và width ≥ 2m phải được label                                  |
| Exclusion      | Sidewalk, parking, shoulder, grass, empty land, reflection/glare phải được ignore |
| Occlusion      | Áp dụng đúng rule 1–49%, 50–80%, 81–99%, 100%                                     |
| `needs_review` | Chỉ dùng khi không thể quyết định chắc chắn                                       |
| `state`        | Không tự gán giá trị nếu chưa có bằng chứng; default là `__undefined__`           |
| Escalation     | Case ambiguity quan trọng phải dùng `image_escalate`                              |
| Export         | Annotation phải được lưu và export đúng từ CVAT                                   |

---

# 3. Self-QC của Annotator

Trước khi submit/export, annotator kiểm tra **100% annotation của chính mình**.

### Checklist

#### 3.1 Class

* [ ] Tất cả polygon sử dụng class `area/driveable`.
* [ ] Không tạo class khác ngoài ontology.
* [ ] Không nhầm `area/driveable` của CVAT với semantic khác.

> Trong project này: `CVAT area/driveable = guideline drivable_area`.

#### 3.2 Geometry

* [ ] Polygon bao phủ toàn bộ vùng drivable area nhìn thấy.
* [ ] Boundary bám theo curb, mép đường hoặc boundary quan sát được.
* [ ] Không kéo polygon vào vùng không có bằng chứng.
* [ ] Không mở rộng polygon xuyên qua vùng bị che hoàn toàn.
* [ ] Sai lệch boundary nằm trong tolerance của guideline khi có thể đánh giá.

#### 3.3 Semantic

* [ ] Không label sidewalk.
* [ ] Không label parking-only area.
* [ ] Không label road shoulder ngoài scope.
* [ ] Không label grass / empty land.
* [ ] Không label reflection/glare.
* [ ] Không label vùng có width < 2m.
* [ ] Crosswalk vẫn thuộc drivable area nếu nằm trong vùng đường.

#### 3.4 Occlusion / visibility

* [ ] Partial occlusion 1–49%: label.
* [ ] Partial occlusion 50–80%: label + `needs_review=true`.
* [ ] Heavy occlusion 81–99%: label + `needs_review=true`.
* [ ] Full occlusion 100%: ignore.
* [ ] Low visibility: chỉ label phần có đủ bằng chứng.

#### 3.5 Escalation

* [ ] Case không chắc chắn không được tự đoán.
* [ ] Dùng `needs_review=true` khi uncertainty thuộc một region.
* [ ] Dùng `image_escalate` khi ambiguity ảnh hưởng ở mức toàn ảnh / không thể tạo annotation đáng tin cậy.

---

# 4. Risk-based QA Sampling

QA production được chia thành hai lớp.

## 4.1 High-risk / critical samples

Các sample có một hoặc nhiều tình huống sau phải được review:

* reflection / glare
* parking area
* sidewalk / road boundary
* road shoulder
* occlusion
* low visibility
* snow / weather làm che boundary
* intersection hoặc boundary khó xác định
* case đã từng gây disagreement trong calibration
* case có gold decision trong `gold_decisions.csv`

Các sample này được ưu tiên review **100%**.

Trong project hiện tại, calibration evidence (06_calibration_report.csv) đã cho thấy:

* **BDD05**: disagreement về geometry (13-31 điểm) - ranh giới gore area chưa rõ
* **BDD10**: disagreement về geometry - ranh giới parking bay chưa nhất quán
* **BDD13**: disagreement về geometry - xử lý occlusion xe bán tải khác nhau
* **BDD19**: ranh giới mảng bê tông vá (surface transition) chưa có rule rõ
* **BDD21**: disagreement về geometry - ranh giới chân hộ lan guardrail
* **BDD18**: low visibility (blind set - chưa calibration)
* **BDD24**: boundary bị snow che (blind set - chưa calibration)
* **BDD25**: wet road / reflection (blind set - chưa calibration)

Các pattern này được coi là risk patterns cần tiếp tục theo dõi.

---

## 4.2 Random samples

Ngoài các high-risk samples, chọn **30% random samples** trong production output để kiểm tra.

Mục đích:

* phát hiện lỗi ở các ảnh nhìn bình thường;
* tránh việc annotator chỉ được QA ở những case khó;
* kiểm tra consistency của execution.

Random sampling phải được thực hiện độc lập với annotator khi có thể.

---

# 5. Calibration QA

Calibration được dùng để phát hiện disagreement trước khi freeze gold.

 Trong evidence hiện tại (06_calibration_measure.csv):

* `BDD05`, `BDD10`, `BDD13`, `BDD19`, `BDD21` có agreement 100% về count và attributes
* **BUT** có disagreement về geometry (tọa độ polygon khác nhau) do guideline chưa đủ rõ

### Rule xử lý

Nếu calibration disagreement xuất hiện:

```text
Disagreement
    ↓
Kiểm tra guideline
    ↓
Có rule rõ?
 ┌───┴────┐
Yes      No
 ↓        ↓
Execution  Guideline gap
error      ↓
 ↓       Revise guideline
Coaching   ↓
          Re-calibration
```

Không được chỉ sửa annotation mà bỏ qua nguyên nhân specification nếu disagreement xuất phát từ guideline.

---

# 6. Defect Classification

Mỗi defect phải được phân loại theo nguyên nhân.

| Loại              | Ý nghĩa                                          | Ví dụ                                           |
| ----------------- | ------------------------------------------------ | ----------------------------------------------- |
| `guideline_gap`   | Rule chưa đủ rõ hoặc thiếu case                  | BDD05 gây disagreement về vùng nhỏ bị che       |
| `data_ambiguity`  | Ảnh không đủ evidence để quyết định              | Boundary bị che hoàn toàn                       |
| `execution_error` | Guideline đã rõ nhưng annotator thao tác sai     | Gán sai class, sai attribute                    |
| `format_error`    | Annotation đúng semantic nhưng export/schema sai | Giá trị attribute dùng pipe thay vì giá trị đơn |

### Evidence hiện tại

`06_calibration_report.csv` đã ghi nhận:

* **BDD05** geometry disagreement (gore area boundary) → `guideline_gap`
* **BDD10** geometry disagreement (parking bay boundary) → `guideline_gap`
* **BDD13** geometry disagreement (occlusion handling) → `guideline_gap`
* **BDD19** semantic ambiguity (surface transition) → `guideline_gap`
* **BDD21** geometry disagreement (guardrail boundary) → `guideline_gap`

Các issue này phải được phản ánh vào guideline/revision log nếu chúng ảnh hưởng đến annotation process.

---

# 7. Defect Severity

## Critical

Lỗi làm thay đổi đáng kể semantic của drivable area.

Ví dụ:

* Label sidewalk thành `road`.
* Label parking-only area thành `road`.
* Label reflection/glare thành drivable area.
* Label median / clearly non-drivable area thành road.
* Bỏ toàn bộ vùng drivable area chính.
* Mở rộng polygon vào vùng rõ ràng không phải drivable area.

**Action:**

```text
Rework ngay
+
Kiểm tra các sample có cùng pattern
```

---

## Major

Lỗi semantic hoặc geometry đáng kể nhưng chưa làm thay đổi toàn bộ quyết định.

Ví dụ:

* Bỏ sót một phần đáng kể của drivable area.
* Polygon mở rộng rõ ràng vào shoulder.
* Xử lý sai partial occlusion.
* Không sử dụng `needs_review=true` trong một ambiguity quan trọng.
* Sai boundary lớn.

**Action:**

```text
Rework
+
Reviewer kiểm tra lại pattern tương tự
```

---

## Minor

Sai lệch nhỏ nhưng semantic decision vẫn đúng.

Ví dụ:

* Boundary lệch nhẹ.
* Một số điểm polygon chưa thật sát boundary.
* Geometry thừa/thiếu nhỏ nhưng không đưa polygon sang semantic region khác.

**Action:**

```text
Correct nếu phát hiện
+
Theo dõi trend
```

---

## Question / Escalation

Case không đủ evidence hoặc guideline chưa đủ rõ để kết luận.

Ví dụ:

* Boundary road/parking bị che hoàn toàn.
* Không thể xác định vùng phía trước có tiếp tục là drivable area.
* Reflection/glare che mất boundary.
* Low visibility làm semantic boundary không thể xác định.

**Action:**

```text
Không tự đoán
↓
needs_review / image_escalate
↓
Ghi evidence
↓
Reviewer quyết định
```

---

# 8. QA Metrics

## 8.1 Decision Accuracy

Đo khả năng áp dụng đúng semantic rule.

```text
Decision Accuracy
= số decision đúng / tổng decision được review
```

Decision bao gồm:

* LABEL
* IGNORE
* `needs_review`
* ESCALATE
* class
* attribute

Đặc biệt tập trung vào các decision có trong `gold_decisions.csv`.

### Target

```text
Decision Accuracy ≥ 95%
```

Threshold này là **project threshold**, không phải industry standard.

---

# 9. Critical Defect Rate

```text
Critical Defect Rate
= số critical defects / tổng samples được review
```

### Gate

```text
Critical Defect Rate = 0
```

Critical defect không được phép tồn tại tại final release.

---

# 10. Major Defect Rate

```text
Major Defect Rate
= số major defects / tổng samples được review
```

Major defect phải được rework hoặc có justification được reviewer chấp nhận.

---

# 11. Geometry Quality

Đối với sample có gold/reference geometry:

```text
IoU = Intersection Area / Union Area
```

Có thể sử dụng IoU để đánh giá polygon giữa peer output và reference/gold.

### Project threshold

```text
PASS: IoU ≥ 0.90

WARN: 0.80 ≤ IoU < 0.90

FAIL: IoU < 0.80
```

Đây là threshold do project đặt ra để kiểm soát geometry; không coi đây là industry standard.

IoU không được dùng thay thế semantic review. Một polygon có IoU khá cao nhưng label nhầm một vùng semantic quan trọng vẫn có thể là Critical/Major defect.

---

# 12. Blind Handoff QA

Blind handoff là bước kiểm tra **transferability** của guideline.

Peer phải:

1. Nhận guideline và CVAT task.
2. Không được xem `gold_decisions.csv`.
3. Không được xem edge-case gold trước khi label.
4. Tự annotation blind samples.
5. Gửi CVAT export.
6. Ghi clarification nếu guideline chưa đủ rõ.

Owner sau đó:

```text
Peer export
    ↓
make score
    ↓
transfer_score.csv
    ↓
make gts
    ↓
GTS / decision accuracy
    ↓
Root-cause analysis
    ↓
peer_feedback.md
    ↓
Guideline v3
```

---

# 13. Blind QA Sampling

Blind set phải bao phủ nhiều loại tình huống thay vì chỉ ảnh bình thường.

Theo `sample_pack.csv`, blind set hiện có:

| Sample | Risk           | Tags | Quyết định gold (từ gold_decisions.csv) |
| ------ | -------------- | ---- | ---------------------------------------- |
| BDD01  | normal         | normal | Baseline kiểm tra độ chính xác cơ bản |
| BDD18  | low visibility | edge;low_visibility | Cảnh ban đêm có làn xe buýt sơn đỏ |
| BDD25  | low visibility | edge;low_visibility | Phản quang mặt đường ướt ban đêm |
| BDD24  | critical;ambiguity | critical;ambiguity | Đống tuyết đùn lấn chiếm lòng đường |
| BDD23  | ambiguity;edge | ambiguity;edge | Mép đường bị tuyết che khuất cần escalate |

Mục đích là kiểm tra peer có thể áp dụng guideline mà **không cần owner giải thích trực tiếp** hay không.

---

# 14. Transferability Metrics

## Decision correctness

GTS phải kiểm tra:

* đúng inclusion/exclusion;
* đúng class;
* đúng attribute;
* đúng ignore;
* đúng escalation.

Không chỉ kiểm tra số lượng polygon.

---

## Critical decision correctness

Các critical decision phải đúng.

Ví dụ:

```text
parking → IGNORE
reflection → IGNORE
clearly non-drivable → IGNORE
drivable area → LABEL
```

Critical semantic error trong blind set phải được điều tra nguyên nhân.

---

## Clarification independence

Theo dõi số lượng câu hỏi peer phải hỏi owner.

```text
Clarification count
= số câu hỏi peer cần hỏi trong blind window
```

Câu hỏi càng nhiều cho cùng một rule cho thấy guideline có thể chưa đủ actionable.

Tuy nhiên, câu hỏi trung thực **không tự động là defect**. Cần phân loại câu hỏi thành:

* guideline gap;
* data ambiguity;
* execution issue.

---

# 15. Guideline Gap Handling

Khi QA phát hiện guideline gap:

### Bước 1

Ghi issue:

```text
sample_id
defect_type
severity
evidence
expected_decision
actual_decision
root_cause
action
status
```

### Bước 2

Phân loại:

```text
guideline_gap
data_ambiguity
execution_error
```

### Bước 3

Nếu là `guideline_gap`:

* sửa rule;
* thêm example nếu cần;
* thêm escalation rule nếu cần;
* tăng guideline version.

### Bước 4

Ghi thay đổi vào:

```text
project/08_revision_log.md
```

### Bước 5

Re-check các sample có cùng pattern.

---

# 16. Revision Gate

Project sử dụng:

```text
v1 = initial guideline
v2 = sau calibration
v3 = sau blind handoff
```

Hiện tại guideline đã ở **v2** (sau calibration) và revision log đã ghi các thay đổi liên quan:

* mapping `CVAT area/driveable = guideline drivable_area`;
* minimum polygon complexity (≥ 10 điểm);
* gore area boundary rule (loại bỏ hoàn toàn vùng vạch chéo);
* parking bay boundary rule (dừng tại vạch sơn phân cách);
* occlusion threshold (51-80% che → needs_review=true);
* guardrail boundary rule (kết thúc TẠI CHÂN hộ lan);
* surface transition rule (mảng bê tông vá ranh giới không rõ → state=ambiguous)

Sau blind handoff, các finding phải được dùng để quyết định nội dung **v3**.

Không tăng version chỉ để hoàn thành checklist; mỗi revision phải có evidence.

---

# 17. Final Quality Gate

Một batch được **PASS** khi đáp ứng tất cả điều kiện:

```text
1. Critical Defect Rate = 0

2. Không còn critical defect chưa xử lý

3. Major defects đã được rework
   hoặc có justification được reviewer chấp nhận

4. Decision Accuracy ≥ 95%

5. Gold/reference geometry đạt IoU ≥ 0.90
   trên các sample được đánh giá geometry
   **Lưu ý:** Polygon phải có ≥ 10 điểm (theo rule mới sau calibration)

6. Không còn guideline gap chưa xử lý
   có khả năng ảnh hưởng production

7. Các critical-risk samples đã được review

8. Blind handoff đã được score

9. Peer feedback đã được phân loại nguyên nhân

10. Guideline revision đã được ghi trong 08_revision_log.md

11. QA evidence và export cần thiết đã được lưu trong repo
```

---

# 18. Rework Gate

Chuyển sang **REWORK** nếu xảy ra một trong các trường hợp:

```text
Critical defect > 0
OR
Decision Accuracy < 95%
OR
Geometry IoU < 0.90 trên gold/reference sample
OR
Major defect chưa được xử lý
OR
Một pattern lỗi lặp lại trên nhiều sample
OR
Guideline gap ảnh hưởng production chưa được cập nhật
```

Sau rework phải kiểm tra lại sample lỗi và các sample cùng pattern.

---

# 19. Escalation Gate

Không ép annotator đưa ra decision khi ảnh không đủ evidence.

Nếu:

```text
Không đủ evidence
+
Guideline không thể quyết định
```

thì:

```text
ESCALATE
↓
needs_review / image_escalate
↓
record evidence
↓
reviewer decision
↓
update guideline nếu cần
```

Đặc biệt áp dụng cho:

* boundary bị che;
* ambiguity road / sidewalk;
* ambiguity road / parking;
* low visibility;
* reflection/glare che boundary.

---

# 20. QA Evidence

Các bằng chứng QA chính của project nằm ở:

| File                                     | Vai trò                               |
| ---------------------------------------- | ------------------------------------- |
| `06_calibration_measure.csv`             | disagreement giữa annotator           |
| `06_calibration_report.csv`              | diagnosis + action sau calibration    |
| `04_edge_cases/gold_decisions.csv`       | expected decisions                    |
| `07_blind_handoff/clarification_log.csv` | câu hỏi của peer                      |
| `07_blind_handoff/peer_feedback.md`      | feedback + root cause                 |
| `08_revision_log.md`                     | lịch sử thay đổi guideline            |
| `transfer_score.csv`                     | kết quả blind handoff                 |
| GTS                                      | kiểm tra transferability              |
| CVAT export                              | evidence geometry + class + attribute |

QA result phải truy xuất được từ annotation → defect → evidence → action.

---

# 21. QA Philosophy

QA của project không cố biến mọi sai lệch polygon nhỏ thành lỗi nghiêm trọng.

Thứ tự ưu tiên là:

```text
1. Semantic correctness
       ↓
2. Critical inclusion / exclusion
       ↓
3. Correct handling of ambiguity
       ↓
4. Geometry correctness
       ↓
5. Small boundary refinement
```

Lý do:

Downstream use case cần phân biệt chính xác:

```text
DRIVABLE AREA
vs
NON-DRIVABLE AREA
```

Do đó:

* Gán sidewalk thành road → Critical.
* Gán parking thành road → Critical.
* Gán reflection thành road → Critical.
* Bỏ sót một phần đáng kể road → Major.
* Boundary lệch nhỏ nhưng semantic vẫn đúng → Minor.
* Không đủ bằng chứng → Escalate thay vì đoán.

Mục tiêu cuối cùng của QA là bảo đảm **một annotator hoặc nhóm peer không trực tiếp tham gia thiết kế vẫn có thể đọc guideline, thao tác CVAT và tạo annotation nhất quán với gold expectation**.
