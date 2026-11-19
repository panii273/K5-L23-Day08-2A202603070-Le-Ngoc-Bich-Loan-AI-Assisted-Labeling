# Thông tin học viên

- Họ và tên: Lê Ngọc Bích Loan
- MSHV: 2A202603070
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: panii273
- Repo bài nộp (URL): https://github.com/panii273/K5-L23-Day08-2A202603070-Le-Ngoc-Bich-Loan-AI-Assisted-Labeling
- Commit chốt bài (hash): 12c2fa6
- Máy: Windows 11, chạy model trực tiếp trên máy (local, venv Python), không dùng Colab
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 396 / 45 / 72 = 76.3% / 8.7% / 13.9% (mẫu số: 519 box máy); added: 120/564 = 21.3% (mẫu số: 564 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 45/68 = 66.2% / 45/68 = 66.2% (68 đối tượng gold, 4 ảnh; lạc quan)
- Bắt bẫy: 10/12 = 83.3%
- Thời gian: gán tay 26/4 = 6.5 phút/ảnh; duyệt có pre-label 98/24 = 4.1 phút/ảnh
- Lỗi QC: 20.0% theo ảnh (1/5), 0.9% theo đối tượng (1/106)
- **Quyết định:** Rework — lý do một dòng: tỉ lệ ảnh lỗi 1/5 = 20% > 2% (box motorbike bao nhiều xe đỗ, img_006), cận trên 95% = 62%
