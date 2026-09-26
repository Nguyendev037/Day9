# QA plan + quality gates

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate.

### Reviewer và phạm vi review

* Mỗi annotator thực hiện **self-QC 100% annotation** trước khi submit.
* Reviewer kiểm tra **100% critical-risk samples** và **30% random samples** của mỗi annotator trong production set.
* Nếu một annotator có defect rate ≥ 10% trong lần review đầu, reviewer mở rộng kiểm tra lên **100% output của annotator đó** cho batch hiện tại.
* Các sample thuộc nhóm `edge`, `ambiguity`, `conflict`, `occlusion` hoặc `low_visibility` được ưu tiên review.
* Calibration và blind-handoff samples được review toàn bộ vì đây là các mẫu dùng để đánh giá transferability của guideline.

### Sampling rule

Sample QA được chọn theo ba lớp:

1. **Risk-based sampling:** 100% các ảnh có tình huống dễ nhầm:

   * sidewalk tiếp giáp road
   * parking area
   * road shoulder
   * intersection
   * unclear road boundary
   * occlusion
   * shadow / low visibility

2. **Random sampling:** 30% annotation còn lại được chọn ngẫu nhiên theo annotator.

3. **Annotator-based sampling:** nếu annotator có defect rate ≥ 10%, chuyển sang 100% review cho batch đó.

Mục tiêu là không chỉ kiểm tra các ảnh dễ mà phải phát hiện cả lỗi trong các ảnh bình thường.

### Issue management

Mỗi defect được ghi vào calibration report, blind feedback hoặc QA review log với:

* `sample_id`
* `defect_type`
* `severity`
* `evidence`
* `expected_decision`
* `actual_decision`
* `root_cause`
* `action`
* `status`

Issue chỉ được đóng khi:

1. Annotation đã được sửa nếu cần.
2. Reviewer xác nhận correction.
3. Nếu nguyên nhân là guideline gap, guideline được cập nhật và tăng version.
4. Nếu issue có thể ảnh hưởng các sample khác, các sample cùng pattern phải được re-check.

### Guideline gap

Khi phát hiện guideline gap:

1. Ghi issue vào review/calibration evidence.
2. Xác định đó là `guideline_gap`, `data_ambiguity` hay `execution_error`.
3. Nếu là `guideline_gap`, cập nhật rule, example hoặc escalation rule.
4. Tăng version guideline:

   * v1: initial guideline
   * v2: sau calibration
   * v3: sau blind handoff
5. Ghi thay đổi và evidence vào `08_revision_log.md`.
6. Re-check các annotation có cùng tình huống.

---

## Defect severity

| Severity     | Định nghĩa cho project này                                                                                                        | Ví dụ                                                                                              | Action mặc định                                |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| **Critical** | Sai quyết định làm thay đổi đáng kể vùng đường mà ego vehicle được phép đi, có nguy cơ làm downstream model học sai drivable area | Label sidewalk/median/parking area thành drivable road; bỏ sót phần lớn vùng road tại intersection | Rework ngay + kiểm tra các sample cùng pattern |
| **Major**    | Annotation sai đáng kể về boundary hoặc semantic class nhưng không làm thay đổi toàn bộ quyết định drivable/undrivable            | Polygon ăn rộng sang sidewalk/shoulder; bỏ sót một phần lớn road; boundary sai rõ rệt              | Rework + reviewer kiểm tra lại                 |
| **Minor**    | Sai lệch nhỏ về geometry nhưng semantic decision vẫn đúng                                                                         | Boundary lệch vài pixel; polygon hơi thừa/thiếu ở vùng không ảnh hưởng quyết định chính            | Correct nếu phát hiện; theo dõi trend          |
| **Question** | Case không đủ bằng chứng hoặc guideline chưa quy định rõ, chưa thể kết luận annotator đúng/sai                                    | Road và parking area không thể phân biệt rõ từ ảnh; boundary bị che hoàn toàn                      | Escalate + xem xét guideline                   |

### Critical-risk examples

Các lỗi sau được coi là critical:

* Gán **sidewalk** thành drivable road.
* Gán **median/island** thành drivable road.
* Gán **parking area** thành drivable road khi guideline xác định khu vực này ngoài scope.
* Bỏ toàn bộ vùng road chính khỏi annotation.
* Boundary làm vùng drivable chuyển sang một khu vực clearly non-drivable.

---

## Metrics

| Metric                   | Cách tính                                                        | Vì sao phù hợp với bài toán                                            |
| ------------------------ | ---------------------------------------------------------------- | ---------------------------------------------------------------------- |
| **Decision Accuracy**    | số quyết định đúng / tổng số quyết định được review              | Đo annotator có áp dụng đúng inclusion/exclusion rule hay không        |
| **Critical Defect Rate** | số critical defects / tổng số samples được review                | Critical error có tác động lớn nhất đến downstream drivable-area model |
| **Major Defect Rate**    | số major defects / tổng số samples được review                   | Theo dõi các lỗi geometry/semantic đáng kể                             |
| **Geometry IoU**         | Intersection over Union giữa annotation và gold/reference region | Đo chất lượng boundary của vùng drivable                               |
| **Defect Escape Rate**   | defects phát hiện sau gate / tổng defects                        | Đo khả năng QA phát hiện lỗi trước khi release                         |
| **Rework Rate**          | số samples phải sửa / tổng samples được review                   | Đo chi phí QA và độ ổn định của annotation                             |
| **Guideline Gap Rate**   | số defects do guideline gap / tổng defects                       | Xác định QA đang phát hiện lỗi annotator hay lỗi specification         |

### High-risk metric

**Critical Defect Escape Rate**

```text
Critical Defect Escape Rate
= Critical defects discovered after QA gate
  / Total critical defects
```

Target:

```text
Critical Defect Escape Rate = 0%
```

Không cho phép critical defect đi qua final quality gate.

### Geometry metric

Đối với các sample có gold/reference geometry:

```text
IoU = Intersection Area / Union Area
```

Target đề xuất:

```text
PASS:   IoU ≥ 0.90
WARN:   0.80 ≤ IoU < 0.90
FAIL:   IoU < 0.80
```

Các threshold này là **project thresholds**, không phải industry standard.

---

## Quality gate

```text
PASS if:

  Critical Defect Rate = 0
  AND Critical Defect Escape Rate = 0
  AND Decision Accuracy ≥ 95%
  AND Geometry IoU ≥ 0.90 on gold/reference samples
  AND all identified major defects are reworked
  AND no unresolved guideline gap affects production labels
```

```text
REWORK if:

  Critical Defect Rate > 0
  OR Decision Accuracy < 95%
  OR Geometry IoU < 0.90
  OR unresolved major defects remain
  OR repeated errors indicate an unclear guideline rule
```

```text
REJECT / ESCALATE if:

  A critical semantic ambiguity cannot be resolved from the image
  AND the current guideline does not define the case.

  → Mark as ESCALATE
  → Record evidence
  → Update guideline or add an explicit exception
  → Re-review affected samples
```

## Release rule

Một batch chỉ được release khi:

1. Không còn **critical defect** chưa xử lý.
2. Major defects đã được rework hoặc có justification được reviewer chấp nhận.
3. Decision Accuracy đạt ≥ 95%.
4. Geometry IoU trên gold/reference samples đạt ≥ 0.90.
5. Các guideline gap có ảnh hưởng đến production đã được cập nhật.
6. Reviewer xác nhận QA evidence đầy đủ.

---

## Trade-off

QA tập trung mạnh vào **critical semantic errors** thay vì yêu cầu mọi boundary đều hoàn hảo đến từng pixel.

Lý do là downstream use case của Driveable Road ưu tiên việc phân biệt chính xác:

**drivable road vs non-drivable region**

hơn là phạt một sai lệch geometry rất nhỏ ở boundary.

Vì vậy:

* Critical semantic error → luôn phải rework.
* Major geometry error → phải rework.
* Minor boundary deviation → có thể chấp nhận nếu không thay đổi semantic decision.
* Ambiguous case → không tự đoán; dùng `ESCALATE` và ghi evidence.

Thiết kế này cân bằng giữa **annotation cost** và **downstream risk**: dành nhiều QA effort nhất cho các lỗi có khả năng làm model học sai vùng xe có thể di chuyển.
