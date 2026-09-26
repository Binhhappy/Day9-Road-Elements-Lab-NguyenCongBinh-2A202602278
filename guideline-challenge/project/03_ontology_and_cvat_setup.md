# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây. Thay mọi
placeholder mới là xong (gate G2).

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `drivable_direct` | polygon | class | — | — | — | Vùng/làn xe ego đang đi. Downstream lập đường đi cần vùng này tách riêng; tối đa 1 polygon mỗi ảnh |
| `drivable_alternative` | polygon | class | — | — | — | Làn xe có thể chuyển sang. Downstream quyết định chuyển làn cần nó tách khỏi direct; mỗi làn liền mạch 1 polygon |
| `direction` | — | attribute của `drivable_alternative` | `__undefined__`, `same`, `opposite`, `unknown` | `__undefined__` | true (khớp JSON; không có tác dụng vì dùng Shape) | Cùng một loại vùng nhưng khác hướng lưu thông; chuyển sang làn `opposite` rủi ro cao hơn nhiều |
| `needs_review` | — | attribute (checkbox) của cả 2 class | `false` / `true` | `false` | true (khớp JSON; không có tác dụng vì dùng Shape) | Đánh dấu polygon đã rõ class nhưng ranh giới cần reviewer xem lại |
| `no_alternative` | tag (cả ảnh) | class (tag) | — | — | — | Xác nhận đã rà cả ảnh và không có làn alternative, để phân biệt với "quên vẽ" |
| `image_escalate` | tag (cả ảnh) | class (tag) | — | — | — | Ảnh còn vùng không phân loại được; chuyển cho người phụ trách guideline phân xử |

## Class hay attribute

- `drivable_direct` và `drivable_alternative` là **class** vì downstream dùng trực tiếp hai loại vùng khác nhau (lập
  đường đi và chuyển làn), và rule hình học khác nhau: direct tối đa 1 polygon, alternative mỗi làn 1 polygon.
- `direction` là **attribute** của alternative vì nó là thuộc tính của cùng một loại vùng. Tách thành class
  `alternative_same` / `alternative_opposite` sẽ nhân đôi số class mà không đổi cách vẽ.
- `needs_review` là **attribute** vì nó là trạng thái review, không phải loại vùng.
- `no_alternative` và `image_escalate` là **tag** vì chúng là quyết định cho cả ảnh, không có hình học. Nhờ vậy các
  quyết định IGNORE và ESCALATE nhìn thấy được trong file export.
- **Default có thể gây bias:** `direction` để `__undefined__` đứng đầu danh sách, buộc annotator phải chọn. Nếu để
  mặc định `same`, annotator quên đổi sẽ biến làn ngược chiều thành làn cùng chiều "im lặng", đúng loại lỗi critical.
  `needs_review` mặc định `false` là an toàn vì quên bật chỉ làm mất một lượt review, không đổi nhãn.
- Không dùng attribute `boundary` (có trong bản nháp đầu): hai người chọn "cạnh chính" khác nhau nên gây lệch
  calibration mà downstream không cần.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): TODO — bật CVAT rồi chạy `python lab9.py cvat` và chép số phiên bản vào đây
- **Tên task calibration** (có version guideline, ví dụ `team07-calib-v1`): `soopichanfanclub-calib-v1-<tên người>` (mỗi thành viên một task)
- **Guide của task đã dán `02_guideline.md`?** TODO (có / chưa — kiểm trên task của từng người)
- **Nhóm dùng Track hay Shape, vì sao:** Shape. Đây là task ảnh tĩnh BDD100K, mỗi ảnh độc lập, không có object cần
  theo dõi qua frame nên Track không mang thêm thông tin.

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào
escalate. Ghi lại ai test và chỗ họ vấp:

TODO — người test (không phải Dư Văn Sang hay Nguyễn Tiến Sỹ, hai bạn đã dựng CVAT) mở task, trả lời 4 câu trên và ghi
lại chỗ vấp.
