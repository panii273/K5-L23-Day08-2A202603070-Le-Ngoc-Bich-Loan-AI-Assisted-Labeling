# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603070 - Lê Ngọc Bích Loan

> Chỉ phần thay đổi so với v1.0. Mỗi dòng khớp một `case_id` trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày gần camera: từng xe hay `ignore_region`? | **Mỗi box chỉ 1 xe.** Xe tách được (thấy yên + ít nhất 1 bánh hoặc đầu xe) → gán từng xe. Phần ≥ 5 xe chồng nhau không tách được → **một** `ignore_region`. Không bao giờ dùng 1 box `motorbike` cho cả hàng xe. | img_006 (#96), img_028, img_029 | CAL-03 |
| 3 | Hàng nhô ra: box xe có gồm hàng không? | Giữ ngưỡng v1.0: hàng nhô ra > ~20% chiều dài xe **không** tính vào box xe; giỏ / rổ gắn cố định thì tính. | img_031 | CAL-04 |
| 1, 4, 5, 6, 7 | Xe đạp điện; người trong xe có mui; kính xe; box chồng ban đêm; conf tối thiểu | **Chưa chốt**: lô 24 ảnh không có đủ ca để quyết (không gặp xe đạp điện; mask chỉ làm 3 vật). Giữ nguyên v1.0. | — | — |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, nhãn của người duyệt)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_002, img_011 | Xe bán tải / quầy hàng / mép xe bị gọi tuk-tuk | YOLOE-26s (prompt `three_wheeler`, ngưỡng 0.15) gán `three_wheeler`; 26 box bị xoá / 9 giữ | Chỉ `three_wheeler` khi thấy 3 bánh hoặc khung tuk-tuk / xích lô / ba gác | Mục 2 + 4.3: phải đếm được bánh hoặc thấy khung đặc trưng; không chắc → `can_review` (CAL-01) |
| img_032 | Xích lô trống đỗ bên đường | YOLO26m gọi `bicycle` | `three_wheeler` | Mục 5: xích lô trống / đang đỗ vẫn là `three_wheeler` (CAL-02) |
| img_027 | Một xe rất nhỏ ở xa (cao < 20 px) | — (người duyệt vẽ `ignore_region` cho 1 vật) | Không vẽ gì | Mục 4.1.6: `ignore_region` **chỉ** cho cụm ≥ 5 vật, không cho 1 vật (CAL-05) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở 2, 3; thêm quy tắc đếm bánh cho `three_wheeler`, xích lô trống, `ignore_region` chỉ cho cụm | CAL-01 → CAL-05 (CAL-06 là lỗi quy trình, không đổi guideline) |
