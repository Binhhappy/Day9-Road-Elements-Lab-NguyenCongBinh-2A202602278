# Annotation guideline — Drivable area BDD100K

**Version:** v1

## 1. Objective + scope

- Gắn polygon vùng xe ego đang đi (`drivable_direct`) và mọi làn xe cơ giới cùng phần đường mà xe có thể đi tới qua các vạch cho phép (`drivable_alternative`), không chỉ làn kề sát.
- Chỉ gắn mặt đường dành cho xe cơ giới; không gắn vỉa hè, làn xe đạp, lề đường, làn/ô đỗ xe, đảo hoặc vùng kẻ sọc phân luồng.
- Đây là task ảnh tĩnh BDD100K; không suy ra ý định, quỹ đạo hay hành vi tương lai của xe.

## 2. Annotation unit

- Đơn vị annotation là một ảnh; dùng Shape, không dùng Track.
- Mỗi ảnh có tối đa một polygon `drivable_direct`; mỗi làn `drivable_alternative` liền mạch là một polygon riêng.
- Gắn tag `no_alternative` chỉ khi đã kiểm tra toàn ảnh và chắc chắn không có alternative.

## 3. Geometry rule

- Vẽ phần mặt đường nhìn thấy được; bám theo vạch, curb hoặc mép mặt đường. Không vẽ polygon lên xe hay vật cản.
- Xe phía trước che làn: kết thúc polygon tại đường nối qua điểm tiếp xúc mặt đường của bánh sau; không gắn phần đường sau xe.
- Xe đỗ sát mép làn: đưa mép polygon tới điểm tiếp xúc bánh xe với mặt đường; không gắn phần nằm dưới xe.
- Xe ego che phần dưới ảnh: dừng cạnh polygon tại mép trên nhìn thấy được của capo hoặc taplo, bao gồm vùng bị giá đỡ/thiết bị che; không vẽ lên xe ego.
- Khi không bị xe ego che, kéo polygon tới mép dưới ảnh nếu mặt đường tiếp tục ra ngoài khung. Ở đầu xa, dừng tại điểm cuối mặt đường còn nhìn thấy rõ.
- Dùng tối đa 12 điểm mỗi polygon, đặt tại góc hoặc chỗ đổi hướng; polygon không được tự cắt.
- Khi ranh giới polygon không đủ rõ để vẽ nhất quán, giữ class nếu đã rõ và bật `needs_review`.

## 4. Taxonomy

- Class `drivable_direct`: vùng/làn xe ego đang đi; tối đa một polygon mỗi ảnh.
- Class `drivable_alternative`: mỗi làn xe cơ giới liền mạch mà xe có thể đi tới theo quy tắc ở mục 5 là một polygon.
- Attribute `direction` chỉ áp dụng cho `drivable_alternative`: `same` (cùng chiều), `opposite` (ngược chiều), `unknown` (không đủ bằng chứng xác định hướng).
- `direction` mặc định là `__undefined__`; trước khi export, bắt buộc chọn `same`, `opposite` hoặc `unknown`.
- Không dùng attribute `boundary`; khi hình học cần xem lại, dùng `needs_review`.

## 5. Inclusion / exclusion

- **Có vạch chia làn:** `direct` là làn ego đang đi, nằm giữa hai ranh làn của nó. Gắn mọi làn xe cơ giới trong cùng phần đường mà xe có thể đi tới qua chuỗi vạch cho phép; không chỉ gắn làn kề sát.
- **Vạch đứt trắng:** làn bên kia là `alternative`, `direction=same`.
- **Vạch đứt vàng:** làn bên kia là `alternative`, `direction=opposite`, nếu không có dải phân cách/vật cản cứng.
- **Một vạch trắng liền giữa hai làn xe cùng chiều:** làn bên kia vẫn là `alternative`, `direction=same`.
- **Vạch vàng liền đơn/đôi hoặc dải cỏ, hàng rào, bê tông phân cách:** không gắn làn phía bên kia thành alternative.
- **Gore/vùng kẻ sọc chéo:** không vẽ polygon lên vùng sọc; làn nằm bên kia gore cũng không phải `alternative`, kể cả khi có xe đang đi trên làn đó.
- **Không có vạch chia làn:** nếu không có vạch giữa và không thấy xe lưu thông ngược chiều, xem toàn bộ mặt đường xe chạy được là một chiều: gắn một `drivable_direct` phủ mặt đường đó và tag `no_alternative`.
- Nếu không có vạch nhưng thấy rõ xe lưu thông ngược chiều, `direct` là nửa mặt đường phía ego; nửa đối diện là `drivable_alternative` `direction=opposite`.
- **Làn xe đạp, lề/shoulder cao tốc, làn hoặc dải đỗ xe:** không gắn; áp dụng cả khi dải đỗ xe không có vạch nhưng nhận ra được nhờ curb/xe đỗ.
- **Giao lộ:** `direct` tiếp tục thẳng theo phần nối dài của làn ego; không vẽ đường cắt ngang. Chỉ gắn làn kề làm `alternative` khi vạch làn thể hiện rõ nó nối từ cùng phần đường trước giao lộ; không gắn nhánh đường bên/cắt ngang.
- Vạch qua đường nằm trên mặt đường không tạo class riêng; phần mặt đường trong làn vẫn thuộc polygon tương ứng.

## 6. Visibility / occlusion

- Chỉ gắn phần mặt đường nhìn thấy; không nội suy phần bị xe/vật cản che. Với xe phía trước và xe đỗ sát mép, áp dụng điểm dừng/điểm biên ở mục 3.
- Nếu mép ảnh cắt vùng và không bị capo/taplo che, kéo polygon tới khung ảnh.
- Khi class đã rõ nhưng ranh polygon chưa chắc, gắn polygon và bật `needs_review`.
- Khi polygon chắc chắn là alternative nhưng không rõ hướng, đặt `direction=unknown`.
- Khi không xác định được class của một vùng, không đoán class; gắn `image_escalate` cho ảnh. Vẫn gắn các polygon khác đã xác định rõ.

## 7. Ambiguity / escalation

- `LABEL`: vẽ polygon với class; polygon alternative phải có `direction`.
- `IGNORE`: không vẽ vùng ngoài scope. `no_alternative` chỉ xác nhận đã rà ảnh và không có làn alternative.
- `UNKNOWN`: dùng `direction=unknown` khi polygon chắc chắn là alternative nhưng không thấy rõ hướng; không dùng `image_escalate` chỉ vì chưa rõ hướng.
- `needs_review`: bật cho polygon đã rõ class nhưng hình học/ranh giới cần người review.
- `image_escalate`: gắn khi còn vùng không thể phân loại chắc chắn. Vẫn vẽ và giữ các polygon direct/alternative đã rõ; không tự đoán phần còn mơ hồ.
- Không gắn đồng thời `image_escalate` và `no_alternative`: ảnh còn điểm chưa phân xử thì chưa thể xác nhận chắc chắn rằng không có alternative.

## 8. Temporal rule

- Không áp dụng — task ảnh tĩnh.

## 9. Examples

- Bảy ảnh trong bảng chỉ được đưa vào split `example` hoặc `calibration`; không đưa vào split `blind`.

| sample_id | Thấy gì                                                                                                                                 | Kết quả mong đợi                                                                                                                                  | Luật áp dụng                                                                             |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| BDD03     | Đường gần thẳng; ego ở làn trái sát vạch vàng; bên trái vạch là dải cỏ và hàng rào, phía sau là làn ngược chiều | 1 `drivable_direct`; 1 `drivable_alternative` `direction=same` ở làn kề cùng chiều; không vẽ qua dải cỏ/hàng rào                      | Dải phân cách cứng loại trừ làn phía bên kia                                       |
| BDD10     | Đường phố có xe đỗ, làn xe đạp bên phải và vạch vàng đứt ở giữa đường                                               | 1 `drivable_direct`; mọi làn xe cơ giới bên kia vạch vàng đứt là `drivable_alternative` `direction=opposite`; không vẽ làn xe đạp | Vạch vàng đứt; loại trừ làn xe đạp                                                 |
| BDD16     | Đường dưới cầu vượt, đủ sáng; có vùng kẻ sọc vàng bên trái và một làn có xe trắng bên kia vùng sọc              | 1 `drivable_direct`; gắn các làn cùng phần đường theo vạch; không gắn làn bên kia gore; không vẽ vùng sọc                           | Gore không phải lối chuyển làn; làn bên kia gore không là alternative              |
| BDD17     | Trời mưa; taplo và giá đỡ điện thoại che phần dưới ảnh                                                                       | Vẽ polygon trên phần đường nhìn thấy; dừng cạnh dưới tại mép trên taplo, không phủ taplo/giá đỡ                                   | Mép capo/taplo của xe ego che vùng phía dưới                                          |
| BDD18     | Đường phố ban đêm có vạch qua đường                                                                                            | Vẽ polygon theo làn/mặt đường; không tạo polygon riêng cho vạch qua đường                                                                | Vạch qua đường không làm thay đổi class vùng mặt đường                         |
| BDD23     | Đường khu dân cư hẹp, không vạch giữa, xe đỗ hai bên, không thấy xe lưu thông ngược chiều                              | 1 `drivable_direct` phủ mặt đường xe chạy được; gắn tag `no_alternative`                                                                 | Đường không vạch, không có dấu hiệu hai chiều: coi là một chiều cho annotation |
| BDD24     | Đường phố có tuyết che khuất ranh mặt đường/làn đến mức không thể xác định class vùng                                | Gắn tag `image_escalate`; vẫn vẽ các polygon khác nếu class của chúng rõ                                                                    | Không đoán class khi bằng chứng hình ảnh không đủ                                 |

## 10. Common mistakes

- Gắn vùng qua dải cỏ/rào/bê tông thành alternative: dừng tại dải phân cách.
- Gắn gore, làn xe đạp, shoulder hoặc làn đỗ xe: các vùng này nằm ngoài scope.
- Vẽ polygon xuyên qua xe phía trước, xe đỗ hoặc capo/taplo: dừng theo quy tắc occlusion ở mục 3.
- Dùng `no_alternative` khi ảnh còn vùng chưa phân xử hoặc gắn đồng thời tag này với `image_escalate`.
- Quên `direction` cho alternative hoặc vượt quá 12 điểm/polygon; kiểm tra attribute và số điểm trước khi export.
