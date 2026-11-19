# Báo cáo Day 08 — 2A202603070 - Lê Ngọc Bích Loan

> Một file cho cả bài. Mọi con số kèm **mẫu số** và nguồn (file nào).

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | 0.3 chung (`--conf 0.3`) | r04 (0.25) bắt được vật xa; r05 (0.5) sót nhiều người / xe ở xa. `threshold_sweep` không tìm được ngưỡng ≤ 0.5 đạt recall 90% cho các lớp này (person tối đa 71%/34 vật, motorbike 75%/12, car 88%/8) nên giữ mặc định; bus (1 vật), truck (0 vật) quá ít để chọn |
| three_wheeler | YOLOE-26s, prompt `three_wheeler` | 0.15 | r03, r06: YOLOE bắt tuk-tuk / xích lô mà YOLO26 gọi car / bicycle (r04, r07). Sweep: recall 7/7 = 100%, precision 78% trên gold |

- Lệnh `merge_prelabels.py` (theo GUIDE + chuỗi ngưỡng từ `threshold_sweep.md`): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM:** SAM 2.1-b (`sam2.1_b.pt`). So với MobileSAM trên cùng 4 click (r12) mask gần như giống nhau (13.3% so với 13.6% ảnh); chọn 2.1-b vì là bản mặc định của bài và đủ nhanh trên CPU (~11 s/ảnh gồm nạp model).
- **Lỗi không sửa được bằng đổi ngưỡng:** YOLO26 (COCO) không có lớp xe ba bánh nên tuk-tuk thành `car` (img_004, img_013: 0.34–0.42), xích lô thành `bicycle` (img_032). Sửa bằng model thứ hai (YOLOE prompt `three_wheeler`) + merge, và quy tắc đếm bánh trong guideline v1.1 (CAL-01, CAL-02).
- **Điểm cao mà sai / thấp mà đúng:** YOLOE cho `three_wheeler` 0.37 trên xe bán tải img_002 (sai), trong khi xích lô đúng ở img_032 chỉ 0.27. Ngưỡng 0.15 chọn trên gold nên giữ quá nhiều box sai: 26 box `three_wheeler` bị xoá so với 9 giữ nguyên (mục 2).
- **Giới hạn:** ngưỡng chọn trên chính 4 ảnh gold và chấm lại trên đó, nên precision / recall của máy trên gold là lạc quan; lớp ít vật thì không chọn được ngưỡng.

## 2. AI error profile (từ `qc/review_stats.md`)

Đã loại 27 đối tượng quanh bẫy khỏi bảng (lỗi nhân tạo).

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 396 | 76.3% | 519 box máy |
| edited | 45 | 8.7% | 519 box máy |
| deleted | 72 | 13.9% | 519 box máy |
| added | 120 | 21.3% | 564 đối tượng sau duyệt |

- 13 trong số deleted + added là **đổi lớp**; vật máy sót thật ≈ 120 − 13 = 107.
- **Lớp tệ nhất: `three_wheeler`**: 26 deleted / 41 box máy (9 accepted, 6 edited). Nguyên nhân: ngưỡng 0.15 quá thấp, YOLOE gọi xe bán tải / quầy hàng là tuk-tuk. Tiếp theo là `truck` (4 deleted / 5 box máy) và `motorbike` (47 added: xe đỗ dày và xe xa bị sót, img_005, img_018, img_024).
- **So với gold:** precision 45/68 = 66.2% (KTC 95%: 54–76%), recall 45/68 = 66.2% (KTC 95%: 54–76%); mẫu số recall = 68 đối tượng gold (4 ảnh). Con số lạc quan (in-sample).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Bắt bẫy: 10/12 = 83.3%** (KTC 95%: 55–95%), đạt ngưỡng 80% nhưng sát. Theo loại: missing 3/3, wrong_class 3/3, extra 2/3, loose 2/3.
- **Bẫy lọt:** img_035 (`loose`: box lỏng) và img_034 (`extra`: box ma). Cả hai là loại lỗi "nhìn có vẻ hợp lý": box lỏng vài pixel khó thấy khi màn hình đầy chữ, box thừa nằm giữa đám đông. Bước bị làm qua loa là bước 2 (thừa) và bước 4 (lỏng) của checklist.
- **Accepted 76.3% và bắt bẫy 83.3%:** mình giữ phần lớn box máy, và vẫn bắt được bẫy sai lớp / sót, nhưng lọt đúng loại lỗi nhỏ, tức là thiên kiến tự động hóa thể hiện ở chỗ tin box đã "đủ sát" mà không kiểm mép.

## 4. Máy có đáng dùng không?

- **Thời gian** (`time_log.csv`): gán tay 26 phút / 4 ảnh = **6.5 phút/ảnh**; duyệt có pre-label 98 phút / 24 ảnh = **4.1 phút/ảnh**, tức nhanh hơn khoảng 37%. Thiên lệch: gold gán trước nên lúc duyệt đã quen cảnh; ảnh batch đông hơn (589 đối tượng / 24 ảnh ≈ 25 so với 90 / 4 ≈ 22).
- **Kết luận: có, cho person / motorbike / car** (accepted cao, tiết kiệm thời gian); **chưa, cho `three_wheeler`** ở ngưỡng 0.15 (xoá nhiều hơn giữ). Máy vẫn sót khoảng 1/3 vật (recall 66% trên gold, added 21.3%), nên bắt buộc có người duyệt.
- **Ba quy tắc giảm thiên kiến của mình:**
  1. Quét ảnh khi **ẩn hết box máy** và đếm vật trước (bước 0) trên mọi ảnh; kiểm bằng số: `added` không giảm bất thường so với lô trước.
  2. **Tắt chữ chi tiết trên box** (chỉ hiện label) và làm bước 2 + 4 (thừa, lỏng) riêng cho từng box, phóng to mép box; kiểm bằng số: bẫy `loose` và `extra` lọt = 0.
  3. Nâng ngưỡng `three_wheeler` lên ~0.3 và lọc tuk-tuk bằng quy tắc đếm bánh (CAL-01); kiểm bằng số: tỉ lệ deleted của `three_wheeler` < 30% ở lô sau.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_004 (tuk-tuk xanh) | SAM 2.1-b | 4 (3 dương + 1 âm) | không người; không có hàng | lấp kín khoang khách và kính trước | `tuktuk_img004.jpg` (+ MobileSAM: `tuktuk_img004_mobilesam.jpg`) |
| img_013 (tuk-tuk có tài xế) | SAM 2.1-b | 4, rồi 7 (3 dương + 4 âm) | lần 1 gồm cả tài xế và người phía sau (44% ảnh); lần 2 bỏ được người phía sau nhưng tài xế vẫn lẫn một phần | mask "lốm đốm" quanh tài xế | `tuktuk_img013.jpg`, `tuktuk_img013_v2.jpg` |
| img_003 (xe ba gác chở rau) | SAM 2.1-b | 5 (3 dương + 2 âm) | gồm hàng hoá (rau) và một phần 2 người ngồi trên xe | 2 polygon; người không tách được bằng click âm | `bagac_img003.jpg` |

- **Đối chiếu guideline 4.2:** quy tắc "hai vật chạm nhau, vùng chạm thuộc vật phía trước" giúp quyết tài xế phải tách khỏi xe, nhưng SAM không làm được bằng click, phải sửa polygon tay. Thiếu quy tắc: hàng hoá trong mask (câu hỏi mở 3 chỉ nói box) và kính / khoang nhìn xuyên (câu hỏi mở 5) chưa chốt.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không làm.

## 6. Calibration (Part 6)

- **6 quyết định** trong `calibration_decisions.csv` (2 `loi_may`, 2 `cau_hoi_mo`, 1 `can_review`, 1 `bay_lot`). Quan trọng nhất: **CAL-01** (`three_wheeler` chỉ khi thấy 3 bánh / khung đặc trưng) và **CAL-03** (mỗi box một xe; khối xe không tách được là `ignore_region`).
- **Quy tắc mới trong `guideline_v1.1_calibrated.md`:** chốt câu hỏi mở 2 và 3; quy tắc đếm bánh cho `three_wheeler`; xích lô trống vẫn là `three_wheeler`; `ignore_region` chỉ dùng cho cụm ≥ 5 vật. Câu hỏi 1, 4, 5, 6, 7 chưa chốt (không đủ ca trong lô).
- **Nguyên nhân gốc:** `three_wheeler` là lỗi của **máy** (lớp COCO thiếu + ngưỡng thấp); khối xe dày và hàng nhô là **guideline mơ hồ**; `ignore_region` cho 1 vật (img_027) và bẫy lọt là **người duyệt chưa nắm quy tắc / làm qua loa**.

## 7. QC và quyết định (Part 6)

- **Mẫu:** 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42): img_003, img_006, img_014, img_031, img_035; 106 đối tượng. **Người kiểm: Le Ngoc Bich Loan** — hạn chế: mình vừa là người duyệt vừa là người kiểm, nên không độc lập.
- **Lỗi theo đối tượng: 1/106 = 0.9%** · **theo ảnh: 1/5 = 20.0%** (ngưỡng Release ≤ 2%).
- **Top lỗi:** `sai_vi_tri_kich_thuoc` 1 lỗi: img_006 #96, một box `motorbike` bao cả hàng xe đỗ (đúng ra phải tách từng xe hoặc `ignore_region`, CAL-03). Không thấy bỏ sót / thừa / sai lớp trong mẫu.
- **Giới hạn mẫu nhỏ:** 1 ảnh lỗi / 5 ảnh, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **62%**; với mẫu 5 ảnh, ngưỡng 2% không thể được chứng minh.
- **Script kiểm được:** định dạng, đủ ảnh, box < 20 px (16 cảnh báo, chủ yếu img_018), số đo bẫy và trạng thái máy. **Không kiểm được:** box có ôm sát vật không, vật mà cả mình lẫn máy cùng sót, mình có làm đủ checklist không.

**Quyết định: Rework**, ngày 2026-11-19. Lý do: guideline đã đủ rõ sau v1.1 (không Hold), nhưng tỉ lệ ảnh lỗi 1/5 = 20% > 2% và lỗi thuộc loại lặp lại (box bao nhiều xe ở ảnh xe đỗ dày).

## 8. Ghi chú cá nhân

- **Quá trình thực hiện: Tôi tự gán nhãn thủ công 4 ảnh gold và trực tiếp duyệt, chỉnh sửa nhãn trên 24 ảnh batch trong CVAT. Tôi dùng Claude để hỗ trợ tổng hợp nhận xét từ model_log.csv, xuất và kiểm tra nhãn, chọn điểm click và chạy SAM, rà mẫu QC, soạn guideline v1.1 và báo cáo. Tôi đọc lại kết quả và nội dung báo cáo. 
- **Gold có bị nhiễm không:** Tôi hoàn tất gán nhãn tay và export cả 4 ảnh gold trước khi xem pre-label của các ảnh này. Sau đó, tôi dùng nhãn gold đã xuất để đối chiếu kết quả mô hình và thử các ngưỡng.
- **Máy / CVAT:** CVAT 2.81 chạy bằng Docker Desktop trên Windows; model chạy CPU (theo `work/runs.csv`); SAM ở Part 5 chạy trên CPU của máy.
- **Dùng cache `data/cache/`?** không.
- **Lỗi / bug:** CVAT hiện chi tiết mọi box (ID, kích thước, conf) làm khó nhìn khi duyệt; không có bug script.
- **Lần sau làm khác:** tắt chữ chi tiết trong CVAT trước khi duyệt và đặt ngưỡng `three_wheeler` cao hơn ngay từ đầu.
