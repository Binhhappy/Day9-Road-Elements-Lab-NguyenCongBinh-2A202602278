# Problem statement + downstream contract

## Bài toán

Trên ảnh tĩnh BDD100K, phân biệt vùng xe ego đang đi (`drivable_direct`) với làn có thể chuyển sang (`drivable_alternative`).
Tập trung vào ca dễ nhầm: làn ngược chiều, lề/bãi đỗ, lối rẽ, vạch mờ hoặc bị tuyết che.
Mục tiêu là tạo nhãn polygon nhất quán cho mô hình phân vùng đường phục vụ lập đường đi và quyết định chuyển làn.

## Downstream contract

1. **Downstream task / model / user là ai?** Mô hình phân vùng đường cho xe tự lái, hỗ trợ lập đường đi và quyết định chuyển làn.
2. **Output annotation nào thực sự cần?** Polygon class `drivable_direct` hoặc `drivable_alternative`; alternative có attribute `direction`.
3. **Failure nào gây hậu quả lớn nhất?** Gán direct sang làn ngược chiều, vỉa hè/lề; hoặc coi làn bên kia dải phân cách cứng là alternative.
4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?** Annotator gắn `needs_review` hoặc tag `image_escalate` trong CVAT; người phụ trách guideline phân xử và cập nhật quy tắc nếu cần.

## Scope

- **Trong scope (bắt buộc label):** Vùng direct (tối đa một polygon/ảnh) và mọi làn alternative xe có thể đi tới qua các vạch cho phép; mỗi làn liền mạch là một polygon.
- **Đường không vạch:** Nếu không có vạch giữa và không thấy xe lưu thông ngược chiều, toàn mặt đường xe chạy được là direct và gắn `no_alternative`; nếu thấy xe ngược chiều, chia theo đường giữa ước tính.
- **Ngoài scope (ignore):** Vỉa hè, lề không lưu thông, làn phía bên kia dải phân cách cứng, và phần bị che/ngoài khung không nhìn thấy.
- **Geometry tolerance:** Bám theo vạch, curb hoặc mép mặt đường nhìn thấy; khi ranh polygon chưa chắc, gắn `needs_review`. Không đặt ngưỡng pixel.

## Output chấm được

CVAT export phải có polygon và class `drivable_direct`/`drivable_alternative`; alternative có `direction`.
Ca cần xem lại thể hiện bằng `needs_review`; ảnh không thể kết luận thể hiện bằng tag `image_escalate`, còn ảnh đã kiểm tra không có alternative bằng `no_alternative`.
Các giá trị và tag được đối chiếu trong export; bất đồng hình học phải mở polygon trong CVAT để review theo gold.

## Dữ liệu và giới hạn

Nguồn: 26 ảnh tĩnh BDD100K trong `data/bdd100k` (13 cao tốc, 11 đường phố, 2 khu dân cư).
Tập có 2 ảnh đêm, 2 ảnh chạng vạng, 2 ảnh tuyết và 1 ảnh mưa; chỉ đại diện cho sample pack đã chọn, không đại diện toàn bộ BDD100K.
