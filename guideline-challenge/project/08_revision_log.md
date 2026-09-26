# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Bản nháp đầu đủ 10 mục: 2 class polygon `drivable_direct` / `drivable_alternative`, attribute `direction`, `needs_review`, 2 tag `no_alternative` / `image_escalate`; bảng vạch kẻ → class; luật dừng polygon ở bánh sau xe phía trước và mép capo/taplo | Chuyển bài toán thành rule nhìn thấy được trong ảnh, mỗi quyết định LABEL / IGNORE / UNKNOWN / ESCALATE đều thể hiện trong export CVAT | So bản nháp đầu: bỏ attribute `boundary` (hai người chọn "cạnh chính" khác nhau), bỏ chữ "hợp lệ" không kiểm tra được, thêm luật gore (BDD16), capo/taplo (BDD17), đường không vạch (BDD23) |
