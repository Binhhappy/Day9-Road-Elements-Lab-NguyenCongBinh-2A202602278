# QA plan + quality gates

Không được viết "reviewer kiểm tra lại". Phải có sampling, metric, threshold và action khi fail. Thay mọi placeholder
mới là xong (gate G6).

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate. Ghi cụ thể cho project của nhóm:

- **Ai review, review bao nhiêu:** Nguyen Cong Binh (QA) review. Mỗi batch production review **100% ảnh có tag
  `image_escalate`, polygon có `needs_review=true`, hoặc ảnh có alternative `opposite`**, cộng thêm **20% ngẫu nhiên**
  trong số ảnh còn lại (tối thiểu 5 ảnh mỗi batch). Annotator mới trong 2 batch đầu được review 50%.
- **Chọn sample theo rule nào:** theo rủi ro trước, ngẫu nhiên sau. Ưu tiên ảnh có tag rủi ro trong `sample_pack.csv`
  (`critical`, `conflict`, `low_visibility`, `ambiguity`) và cảnh đường phố hai chiều, vì đây là nơi lỗi critical
  (gắn làn ngược chiều bên kia vạch vàng liền) dễ xảy ra nhất. Phần ngẫu nhiên để ước lượng tỉ lệ lỗi chung.
- **Issue được ghi ở đâu, đóng thế nào:** Reviewer dùng Issue của CVAT, gắn trực tiếp vào polygon/ảnh, ghi mức
  severity ở đầu (`[critical]`, `[major]`, `[minor]`, `[question]`). Annotator sửa và reply; reviewer mở lại ảnh,
  thấy đúng thì resolve. Issue chỉ đóng khi reviewer resolve, annotator không tự đóng.
- **Khi phát hiện guideline gap thì update và version ra sao:** Issue `[question]` lặp lại ≥ 2 lần cho cùng tình huống
  được coi là guideline gap. Lê Đức Huy (spec owner) thêm rule hoặc ví dụ vào `02_guideline.md`, tăng `Version`, ghi
  một dòng vào `08_revision_log.md` kèm sample_id làm bằng chứng, rồi dán lại guideline vào Guide của mọi task đang mở.
  Ảnh đã label theo version cũ ở tình huống đó được đưa vào vòng review lại.

## Defect severity

Nhóm được đổi mapping nếu downstream contract khác, nhưng phải giải thích và chốt trước khi QA.

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| Critical | Lỗi làm downstream cho xe đi vào vùng không được đi, hoặc nhầm vùng đang đi | Gắn làn ngược chiều bên kia vạch vàng liền/dải phân cách thành alternative; nhầm direct với alternative; direct phủ vỉa hè/lề | Rework ngay ảnh đó; review lại 100% batch của annotator đó cho cùng loại lỗi |
| Major | Sai class, sai số polygon, sai attribute, hoặc thiếu tag, mà không đưa xe vào vùng nguy hiểm | Thiếu một làn alternative cùng chiều; `direction` sai giữa `same` và `unknown`; quên `no_alternative`; gắn làn xe đạp/dải đỗ | Rework ảnh đó |
| Minor | Class và attribute đúng, chỉ hình học lệch rule | Cạnh dưới không dừng ở mép capo; đầu xa kéo quá xa; vượt 12 điểm/polygon | Sửa nếu batch còn thời gian; không chặn gate |
| Question | Annotator không chắc, rule chưa trả lời được | Không biết dải giữa hai vạch trắng là làn hay lề | Chuyển spec owner phân xử; có thể thành guideline gap |

## Metrics

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| Image accuracy | Số ảnh review không có lỗi critical hoặc major / tổng số ảnh review | Downstream dùng cả ảnh; một ảnh có lỗi class là một mẫu train sai |
| Critical defect escape rate | Số ảnh có lỗi critical trong mẫu ngẫu nhiên / số ảnh ngẫu nhiên đã review | Ước lượng lỗi critical còn lọt ra production; chỉ tính phần ngẫu nhiên để không bị lệch do chọn theo rủi ro |
| Polygon count agreement | Số ảnh mà số polygon direct/alternative khớp với reviewer / số ảnh review | Đo được tự động từ export CVAT (giống lệnh `calib`), bắt lỗi thiếu hoặc thừa làn |
| Escalation rate | Số ảnh có `image_escalate` / tổng số ảnh | Quá cao thì guideline thiếu rule; bằng 0 ở ảnh mơ hồ thì annotator đang đoán |

Metric high-risk tách riêng: **critical defect escape rate** được báo cáo riêng và có ngưỡng riêng, không gộp vào image
accuracy, vì một lỗi critical ảnh hưởng an toàn nặng hơn nhiều lỗi hình học.

## Quality gate

Threshold là đề xuất của nhóm, không phải chuẩn ngành. Giải thích trade-off cost/risk.

```text
PASS if:
  critical defect escape rate = 0%
  AND image accuracy >= 90%
  AND polygon count agreement >= 85%
REWORK if: không có lỗi critical nhưng image accuracy 75–90% hoặc count agreement < 85%
           → annotator sửa các ảnh lỗi, reviewer review thêm 20% ngẫu nhiên
REJECT / ESCALATE if: có bất kỳ lỗi critical nào trong mẫu ngẫu nhiên, hoặc image accuracy < 75%,
           hoặc escalation rate > 25% → trả cả batch, spec owner xem guideline trước khi label lại
```

Trade-off: ngưỡng critical đặt bằng 0 vì downstream là lập đường đi, một làn ngược chiều bị gắn nhầm có thể dẫn xe vào
làn xe đối diện; chấp nhận chi phí review lại cả batch để đổi lấy rủi ro thấp. Ngưỡng image accuracy 90% (không phải
99%) vì với polygon, lỗi hình học nhỏ khá phổ biến và đã được xếp minor, không chặn gate; đòi cao hơn sẽ làm chi phí
review tăng mạnh mà downstream ít hưởng lợi. Escalation rate > 25% coi là dấu hiệu guideline chưa đủ, sửa guideline rẻ
hơn review từng ảnh.
