# Edge-case library

Tối thiểu **8 card**, khuyến nghị 10–12. Một edge case tốt là case mà hai annotator hợp lý có thể làm khác nhau nếu
guideline chưa rõ. Tám ảnh dễ có label rõ ràng không được tính là edge-case library.

Cần có đủ độ đa dạng: occlusion / truncation / small-far · ambiguous semantics · conflicting road elements · **một case
critical-risk** · **một case guideline cho phép escalation**.

File này là kho nội bộ của nhóm, **không gửi cho peer**. Card dùng ảnh example/calibration thì chép rule + ví dụ sang
`02_guideline.md` (mục 7 và 9) để peer đọc được. Card về ảnh blind chỉ nằm ở đây, và decision của nó phải có trong
`gold_decisions.csv` trước `make freeze`.

`make status` đếm số dòng `CASE ID:` đã điền (đã thay placeholder). Copy khối dưới cho mỗi case.

---

CASE ID: EC01
Sample: BDD03
Scene: Cao tốc, xe ego ở làn trái sát vạch vàng mép; bên trái là dải cỏ + hàng rào, sau đó là đường ngược chiều
Observation: Làn ngược chiều nhìn thấy rõ và khá gần, nhưng ngăn cách bằng dải cỏ và hàng rào
Decision: LABEL (direct + alternative cùng chiều) / IGNORE (phần bên kia dải cỏ)
Expected: 1 `drivable_direct`; 1 `drivable_alternative` `direction=same` ở làn bên phải; không polygon nào bên trái vạch vàng
Rationale: Downstream chuyển làn không được coi đường bên kia dải phân cách cứng là làn có thể đi (failure nặng nhất trong 01)
Common mistake: Gắn đường ngược chiều thành alternative `opposite` vì thấy xe chạy ngược
Diversity: conflict / critical

---

CASE ID: EC02
Sample: BDD16
Scene: Đường dưới cầu vượt; vùng kẻ sọc vàng (gore) bên trái, bên kia gore có một làn đang có xe trắng
Observation: Làn bên kia gore có xe chạy thật, nhìn như làn liền kề
Decision: IGNORE vùng gore và làn bên kia gore
Expected: 1 `drivable_direct` giữa mép gore và vạch trắng phải; không polygon nào trên gore hoặc làn bên kia gore
Rationale: Gore không phải lối chuyển làn; gắn làn bên kia làm downstream lên kế hoạch cắt qua vùng cấm
Common mistake: Kéo direct vòng sang trái xe đen tới tận làn bên kia gore (đã gặp trong bản vẽ thử `labday-09`)
Diversity: conflict / ambiguity

---

CASE ID: EC03
Sample: BDD17
Scene: Phố trời mưa, taplo và giá đỡ điện thoại che gần nửa dưới ảnh; bên phải có dải đỗ xe với xe đen đang đỗ
Observation: Mặt đường chỉ nhìn thấy từ khoảng y = 410 trở lên; dải đỗ xe liền với làn xe chạy, không có vạch rõ
Decision: LABEL phần đường thấy được / IGNORE dải đỗ
Expected: Cạnh dưới mọi polygon dừng ở mép trên taplo; alternative bên phải không phủ dải đỗ xe (dừng ở bánh xe đỗ)
Rationale: Vẽ lên taplo tạo vùng drivable giả; dải đỗ không phải làn xe chạy
Common mistake: Alternative bên phải phủ luôn dải đỗ xe (đã gặp trong bản vẽ thử `labday-09`)
Diversity: occlusion / low_visibility

---

CASE ID: EC04
Sample: BDD18
Scene: Đường phố ban đêm, vạch qua đường ngay trước xe, xe đỗ hai bên
Observation: Vạch qua đường trắng nổi bật trên mặt đường tối; capo che phần dưới
Decision: LABEL
Expected: Vạch qua đường nằm trong polygon của làn tương ứng, không tạo polygon riêng; cạnh dưới dừng ở mép capo
Rationale: Vạch qua đường không đổi class của mặt đường; downstream cần vùng liên tục
Common mistake: Dừng polygon ở mép vạch qua đường, hoặc để khoảng trống giữa polygon và capo
Diversity: low_visibility / geometry

---

CASE ID: EC05
Sample: BDD23
Scene: Đường khu dân cư hẹp, không vạch giữa, xe đỗ kín hai bên, không thấy xe ngược chiều
Observation: Không có bằng chứng đường hai chiều
Decision: LABEL + tag
Expected: 1 `drivable_direct` phủ mặt đường giữa hai hàng xe đỗ (mép theo bánh xe); tag `no_alternative`
Rationale: Theo rule đường không vạch; tag giúp phân biệt "không có alternative" với "quên vẽ"
Common mistake: Chỉ vẽ direct mà quên tag `no_alternative` (đã gặp trong bản vẽ thử `labday-09`); hoặc chia đôi đường
Diversity: ambiguity / negative

---

CASE ID: EC06
Sample: BDD24
Scene: Phố có tuyết, xe ego ở bên phải vạch trắng liền; đống tuyết lớn chiếm phần đường bên phải làn ego
Observation: Không rõ phần đường dưới đống tuyết là làn xe chạy hay dải đỗ; làn ego cần xác định bằng vị trí giữa capo
Decision: LABEL phần đã rõ + ESCALATE
Expected: 1 `drivable_direct` ở làn bên phải vạch trắng (làn chứa điểm giữa capo); 1 `drivable_alternative` `same` bên trái vạch; tag `image_escalate`
Rationale: Không đoán vùng bị tuyết chiếm; vẫn giữ polygon đã chắc để downstream dùng được
Common mistake: Đảo direct và alternative (đã gặp trong bản vẽ thử `labday-09`); hoặc phủ polygon lên đống tuyết
Diversity: escalation / occlusion

---

CASE ID: EC07
Sample: BDD07
Scene: Đường phố hai chiều có vạch vàng liền đôi ở giữa, dải đỗ xe bên phải, giao lộ phía trước
Observation: Làn ngược chiều ngay cạnh, không có vật cản cứng, chỉ có vạch vàng liền đôi
Decision: LABEL direct + IGNORE làn ngược chiều + tag
Expected: 1 `drivable_direct` giữa vạch vàng đôi và vạch dải đỗ; 0 `drivable_alternative`; tag `no_alternative`; direct đi thẳng qua giao lộ
Rationale: Vạch vàng liền đôi cấm vượt; gắn alternative `opposite` khiến downstream lên kế hoạch lấn làn ngược chiều
Common mistake: Coi vạch vàng như vạch đứt và gắn làn trái là `opposite`
Diversity: critical / conflict

---

CASE ID: EC08
Sample: BDD20
Scene: Khu dân cư, vạch vàng liền đôi ở giữa; bên phải có hai vạch trắng liền tạo một dải trống trước hàng xe đỗ; giá đỡ camera che giữa ảnh
Observation: Dải giữa hai vạch trắng rộng gần bằng một làn nhưng không có xe chạy; có thể là làn xe đạp hoặc lề
Decision: LABEL direct / IGNORE làn ngược chiều / dải bên phải: không chấm trong gold
Expected: direct dừng ở vạch trắng thứ nhất bên phải; không alternative bên trái vạch vàng đôi; polygon không phủ giá đỡ camera
Rationale: Hai annotator hợp lý có thể coi dải bên phải là làn hoặc lề; guideline v1 chưa có rule "dải trống giữa hai vạch trắng liền"
Common mistake: Kéo direct qua vạch trắng thứ nhất để gộp dải trống vào làn ego
Diversity: occlusion / ambiguity / critical

---

CASE ID: EC09
Sample: BDD05
Scene: Đường có vạch vàng liền mép trái, vạch trắng liền bên phải, phía phải là vùng rộng kẻ nhiều vạch trắng song song
Observation: Vùng kẻ vạch bên phải trông rộng như một làn, nhưng là vùng kẻ sọc phân luồng
Decision: LABEL direct + IGNORE vùng kẻ sọc + tag
Expected: 1 `drivable_direct` giữa vạch vàng và vạch trắng liền; không polygon trên vùng kẻ sọc; tag `no_alternative`
Rationale: Rule "vạch trắng liền giữa hai làn cùng chiều → alternative" có thể bị áp nhầm; vùng kẻ sọc không phải làn
Common mistake: Gắn vùng kẻ sọc thành alternative `same`
Diversity: conflict / ambiguity

---

CASE ID: EC10
Sample: BDD14
Scene: Cao tốc 5 làn cùng chiều, xe bạc ngay phía trước trong làn ego, rào chắn bên trái
Observation: Vạch làn xa nhất bên trái mờ dần gần rào chắn
Decision: LABEL
Expected: 1 `drivable_direct` dừng ở bánh sau xe bạc; ít nhất 3 `drivable_alternative` `same` (3 trái, 1 phải); không có `no_alternative`
Rationale: Rule "gắn mọi làn, không chỉ làn kề sát"; downstream chuyển làn cần biết đủ số làn
Common mistake: Chỉ gắn làn kề sát hai bên; hoặc vẽ direct xuyên qua xe phía trước
Diversity: small_far / normal
