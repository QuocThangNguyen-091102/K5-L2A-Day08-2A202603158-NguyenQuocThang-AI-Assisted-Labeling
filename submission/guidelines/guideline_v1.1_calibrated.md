# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603158 - Nguyễn Quốc Thắng

> Chỉ phần thay đổi so với v1.0. Mỗi quyết định khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày gần camera: từng xe hay `ignore_region`? | Gán từng xe khi thấy rõ ≥ 1 bánh + yên **và** box cao ≥ 20 px; phần dính nhau không tách được cá thể → một `ignore_region` | img_006, img_024, img_036 | CAL-05 |
| 3 | Hàng nhô ra: box xe có gồm hàng không? | Không gồm hàng nhô > 20% chiều dài xe (box và mask); nhô ≤ 20% thì gồm | img_009 (chuối), img_031 (khay trái cây) | CAL-04 |
| 4 | Người ngồi trong xe ba bánh có mui, chỉ thấy thấp thoáng | Gán `person` khi thấy **thân trên (vai + tay)** hoặc **đầu + vai**; chỉ thấy tay/chân → không gán. Luôn tick `can_review` | img_012 (3 tài xế tuk-tuk) | CAL-03 |
| 1, 5, 6, 7 | Xe đạp điện; kính xe; box chồng đêm; mức conf tối thiểu | **Chưa chốt**: lô này không có ca đủ rõ để quyết | — | — |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013, img_004, img_002 | Tuk-tuk chở khách | YOLO26 gọi `car` 0.37–0.42 hoặc không có box | Luôn `three_wheeler` | Mục 2: lớp `car` không bao giờ dùng cho xe 3 bánh có mui, kể cả khi máy gọi `car` điểm cao (CAL-01) |
| img_002, img_003, img_005, img_009 | Quầy hàng, xe đẩy tay, tủ điện, xe máy chở hàng | YOLOE gắn `three_wheeler` 0.15–0.40 | Xoá / đổi lớp | Mục 2: `three_wheeler` phải thấy bánh xe **và** chỗ người lái/đạp (CAL-02) |
| img_006, img_024, img_021 | Box lệch vài chục px trên xe/người rõ | (bẫy box lỏng) | Sửa box | Mục 7: người duyệt phóng to kiểm 4 cạnh mọi box conf ≥ 0.5 (CAL-06) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở 2, 3, 4; thêm loại trừ cho `car`/`three_wheeler`; thêm bước kiểm cạnh box vào quy trình duyệt | CAL-01 … CAL-06 |
