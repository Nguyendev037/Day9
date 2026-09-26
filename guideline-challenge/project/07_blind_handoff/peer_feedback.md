# Peer feedback + owner response

Phần 1 do **nhóm peer** trả lời (gửi kèm file export). Phần 2 do **nhóm owner** điền. Thay mọi placeholder mới
là xong (gate G5).

- **Nhóm peer:** Nhóm K4-L2-DAY09-Nhom10
- **Người label blind:** Nguyễn Văn A

## 1. Peer trả lời

1. **Rule nào rõ nhất / giúp quyết định nhanh nhất?**
   Rule về **class `road` là drivable_area** và **geometry rule** (polygon, tolerance 2px) rất rõ ràng. Đơn vị annotation là polygon ôm toàn bộ vùng drivable area nhìn thấy, dễ hiểu và áp dụng nhanh. Geometry rule với tolerance 2px giúp tôi không bị áy náy về độ chính xác tuyệt đối của từng điểm.

2. **Rule nào mơ hồ hoặc phải tự suy diễn?**
   Rule **IGNORE** (sidewalk, curb, area behind guardrail, reflection) không đủ chi tiết. Khó nhất là phân biệt ranh giới giữa drivable area và sidewalk/curb khi không có vạch kẻ rõ. Câu "IGNORE sidewalk and curb" không giải thích khi nào vạch kẻ là ranh giới hay không. Reflection trên mặt đường ướt cũng không có tiêu chí rõ ràng để phân biệt với drivable area thật.

3. **Sample nào khiến guideline "vỡ"?**
   **BDD18 (ảnh đêm)**: Khó xác định ranh giới drivable area do thiếu ánh sáng. Guideline yêu cầu "LABEL visible areas only" nhưng không có định nghĩa rõ "visible" trong điều kiện đêm. Tôi không biết có nên vẽ vùng tối hay không, hay đặt `needs_review=true`. Kết quả tôi để `needs_review=false` cho tất cả.

4. **Attribute / default nào trong CVAT dễ gây thao tác sai?**
   **Attribute `state`** với default `__undefined__` gây nhầm lẫn. Guideline không nói rõ khi nào đặt `clear`, khi nào để `__undefined__`. Tôi không dám đổi từ default, nên để hết `__undefined__`. Tương tự, `needs_review` default là `false`, nhưng nhiều trường hợp (occlusion, đêm) cần `true`, nhưng guideline không liệt kê đủ case.

5. **Một thay đổi cụ thể giúp annotator mới ít hỏi hơn?**
   Thêm **ví dụ visual** cho mỗi rule: ảnh highlight vùng nào là drivable area, vùng nào không (sidewalk, curb, reflection, behind guardrail). Đặc biệt là thêm **decision tree** cho attribute: "Nếu vùng nhìn thấy rõ → state=clear; nếu không chắc → needs_review=true; nếu bị che hoàn toàn → ESCALATE".

## 2. Owner phân loại

Owner không tranh luận để bảo vệ guideline. Mỗi feedback và mỗi decision peer làm sai được xếp vào một hướng xử lý.

| Feedback / decision sai | Nguyên nhân (guideline gap / data ambiguity / execution error) | Xử lý (accept + revise / reject with evidence / add escalation rule) | Bằng chứng |
|---|---|---|---|
| Rule IGNORE mơ hồ (sidewalk/curb/guardrail/reflection) | guideline gap | add_example + revise_rule | Thêm ví dụ hình ảnh + định nghĩa ranh giới trong v3 |
| Khó xử lý ảnh đêm (BDD18) | data_ambiguity | add_escalation | Thêm rule: "Nếu không xác định được ranh giới do tối → ESCALATE" trong v3 |
| Attribute `state` không rõ | guideline gap | revise_rule | Làm rõ rule đặt `state` trong ontology table (v3) |
| Default `needs_review=false` gây nhầm | guideline gap | revise_rule + add_example | Thêm ví dụ cho trường hợp needs_review=true (occlusion, đêm, reflection) |
| Peer vẽ 1 polygon tại merge point (BDD19,d1) | guideline gap | revise_rule | Làm rõ rule "separate lanes at merge" với hình minh họa trong v3 |
| Peer không ignore sidewalk/curb (BDD18,d3) | guideline gap | add_example | Thêm hình highlight vùng sidewalk/curb không phải drivable area |
| Peer không ignore guardrail area (BDD21,d1) | guideline gap | add_example | Thêm hình min hoạ vùng phía sau guardrail không phải drivable area |
| Peer không ignore reflection (BDD22,d1) | guideline gap | add_example | Thêm hình phân biệt reflection vs drivable area thật |
