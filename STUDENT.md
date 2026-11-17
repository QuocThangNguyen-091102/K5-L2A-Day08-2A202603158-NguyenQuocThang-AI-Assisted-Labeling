# Thông tin học viên

- Họ và tên: Nguyễn Quốc Thắng
- MSHV: 2A202603158
- Lớp (L2 / L3 / khác): K5 — L23 (theo tên repo đề K5-L23-Day08)
- Tài khoản GitHub: (điền khi tạo repo)
- Repo bài nộp (URL): https://github.com/(tài khoản)/K5-L23-DAY08-2A202603158-NguyenQuocThang-AIPrelabel
- Commit chốt bài (hash): 9b533d6
- Máy: Windows 11 Pro, CPU AMD Ryzen 5 PRO 215 (12 luồng), RAM 11.7 GB, GPU: không có CUDA/MPS (AMD Radeon 740M tích hợp) → model chạy CPU (torch 2.15.0+cpu)
- CVAT (Docker): v2.82.0 (tài liệu lab viết cho v2.81.0)

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 495 / 26 / 68 (mẫu số: 593 box máy); added: 68 (mẫu số: 590 đối tượng)
- So với gold: precision / recall của **máy** 69.9% (51/73) / 91.1% (51/56)
- Bắt bẫy: 8/12 = 66.7% (người duyệt là AI, xem REPORT mục 8)
- Thời gian: gán tay 4.6 phút/ảnh (18.4 phút / 4 ảnh); duyệt có pre-label 1.2 phút/ảnh (29.1 phút / 24 ảnh, AI chạy song song, 22/24 ảnh được duyệt)
- Lỗi QC: 37.5% theo ảnh (3/8), 2.2% theo đối tượng (5/224)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 3/8 > 2%, bắt bẫy 8/12 < 80%, 2 ảnh chưa duyệt
