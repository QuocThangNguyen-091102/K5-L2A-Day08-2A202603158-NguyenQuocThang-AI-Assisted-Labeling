# Báo cáo Day 08 — 2A202603158 - Nguyễn Quốc Thắng

> Mọi con số kèm mẫu số và nguồn. **Đọc mục 8 trước**: phần duyệt batch, bắt bẫy và QC do công cụ AI (Claude Code) làm, không phải học viên.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / motorbike | YOLO26m | person 0.3, motorbike 0.1 | r01/r02: 0.5 làm mất xe máy xa (motorbike 55→32 trên pilot); sweep: person recall đứng 87% từ 0.1–0.3 nên lấy 0.3 (precision cao nhất), motorbike chỉ 0.1 đạt recall 14/14 |
| car | YOLO26m | **0.7** | Recall 7/8 không đổi từ 0.1 tới 0.85, còn FP giảm 7→2: các box `car` điểm 0.37–0.42 chính là tuk-tuk (r04, img_013/img_004) |
| bicycle / bus / truck | YOLO26m | 0.3 (`--conf`) | gold chỉ có 3 / 1 / 0 vật: không đủ để chọn ngưỡng riêng |
| three_wheeler | YOLOE-26s, prompt `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15 | r03: bắt tuk-tuk 0.93, xích lô 0.92 mà YOLO26 bỏ; sweep: recall 7/7 ở 0.15, rớt 5/7 ở 0.25 |

- Lệnh merge: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.3,motorbike=0.1,car=0.7,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM: SAM 2.1 base (`sam2.1_b.pt`)**. So với SAM 1 (`mobile_sam.pt`, r07) trên cùng box: SAM 1 nhanh gấp đôi (3.7 vs 7.6 s/ảnh CPU) nhưng mask người áo hồng img_013 vụn và lem sang lan can hơn. Máy 11.7 GB RAM chạy SAM 2.1 b ổn; SAM 3 không thử (chưa có quyền HuggingFace).
- **Lỗi không sửa được bằng ngưỡng**: tuk-tuk bị YOLO26 gọi `car` (img_013 0.42, img_004 0.37) hoặc không có box nào (img_002, tuk-tuk xanh to nhất img_004); xích lô bị gọi `bicycle` 0.70 (img_032) / `motorbike` 0.28 (img_007). Tăng ngưỡng chỉ làm mất box, không đổi lớp → sửa bằng **đổi model** cho lớp đó (YOLOE + prompt) và đặt ngưỡng `car` cao (0.7).
- **Điểm cao mà sai / điểm thấp mà đúng**: img_034 #1732 `three_wheeler` 0.85 thực ra là xe tải nhỏ; img_005 #1195 `truck` trên trời/cột điện. Ngược lại xích lô bên trái img_032 chỉ 0.48, tuk-tuk gold ở 0.15–0.25 vẫn đúng (sweep: hạ 0.25→0.15 thêm 2 TP).
- **Giới hạn**: ngưỡng chọn và chấm trên **cùng 4 ảnh gold** (56 vật) → precision/recall của máy trên gold là lạc quan; three_wheeler chỉ 7 vật, car 8 vật.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 495 | 83.5% | 593 box máy (đã loại 31 vật quanh bẫy) |
| edited | 26 | 4.4% | 593 box máy |
| deleted | 68 | 11.5% | 593 box máy |
| added | 68 | 11.5% | 590 đối tượng sau duyệt |

- 11 trong deleted+added là **đổi lớp** → máy sót thật ≈ 68 − 11 = 57 vật.
- Lớp tệ nhất: **three_wheeler**: deleted 23 + edited 9 trên 40 box máy (chỉ 8 accepted) → YOLOE ở ngưỡng 0.15 gắn nhầm quầy hàng, xe đẩy, tủ điện, xe máy chở hàng (img_002, img_003 #1180/#1181, img_005 #1237, img_009 #1291). **car**: added 19 → ngưỡng 0.7 làm sót xe bị che (img_024 thêm 8 xe, img_028 thêm 3). **motorbike**: deleted 29 (mảnh/trùng ở ảnh dày img_005).
- So với gold: máy precision **51/73 = 69.9%** (KTC 95% 59–79%), recall **51/56 = 91.1%** (KTC 81–96%); mẫu số 56 đối tượng gold, 4 ảnh.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy **8/12 = 66.7%** (KTC 95% 39–86%) < 80%: wrong_class 3/3, extra 3/3, missing 2/3, **loose 0/3**. Lọt: img_006 motorbike lệch ~50 px, img_024 truck lệch ~13 px, img_021 person cao dư ~30 px, img_034 motorbike bị xoá ở mép trái.
- Người duyệt ở đây là **AI** (mục 8), nên số này đo thiên kiến của quy trình AI, không phải của học viên. Nguyên nhân: (1) bước 4 "box lỏng" bị làm hời hợt: chỉ dẫn "đừng siết lại box đã gần đúng" khiến người duyệt bỏ qua box điểm cao trông hợp lý (img_006, img_024), đúng kiểu neo nhãn / tin điểm cao; (2) **img_014 và img_021 không được duyệt** (dừng vì hết giờ) → bẫy img_021 lọt chắc chắn; (3) img_034: người duyệt thêm một người ở mép trái nhưng không thấy xe máy ngay cạnh → bỏ bước quét mép ảnh.
- Accepted cao (83.5%) đi cùng bắt bẫy loose 0/3: box được chấp nhận vì "gần đúng", chưa thật sự kiểm cạnh.

## 4. Máy có đáng dùng không?

- Thời gian (`time_log.csv`): gán tay gold **18.4 phút / 4 ảnh = 4.6 phút/ảnh** (học viên, ~14 vật/ảnh); duyệt batch 29.1 phút / 24 ảnh do AI chạy song song → **không so sánh được** với người. Thiên lệch thêm: gold ít vật hơn batch (~25 vật/ảnh).
- Kết luận: **có, cho person và motorbike** (person accepted 255/272, motorbike 195/232 box máy của lớp; recall gold 91%); **không tin** pre-label `three_wheeler` (8/40 accepted) và `car` (19 vật phải thêm) → hai lớp này duyệt kỹ hoặc vẽ tay.
- **Ba quy tắc giảm thiên kiến** (kiểm được):
  1. Phóng to kiểm 4 cạnh **mọi** box conf ≥ 0.5 ở ảnh < 30 vật. Đo: bắt bẫy loose ≥ 2/3.
  2. Bước 0: đếm vật theo lớp trước khi bật nhãn máy, ghi số đếm cạnh số box; lệch > 2 thì quét lại theo góc phần tư và **mép ảnh**. Đo: missing 3/3, có số đếm cho 24/24 ảnh.
  3. Không export khi còn ảnh chưa duyệt: hết giờ thì báo **Hold**. Đo: số ảnh đã duyệt = 24/24 (lần này 22/24).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM | Số click | Gồm người lái / hàng? | Lỗi quan sát | File |
|---|---|---|---|---|---|
| img_013 tuk-tuk có tài xế | SAM 2.1 b | 8 (4+/4−), 3 lần thử | Lần 1 (2+/1−) trùm cả tài xế + xe máy phía sau (45.9% ảnh); lần 3 loại được tài xế | Còn lem mép dép, bàn tay, mặt tài xế; lần 2 mất bánh xe + ghế đen | `tuktuk_img013.jpg`, `tuktuk_img013_try1_driver_leak.jpg` |
| img_032 xích lô trước | SAM 2.1 b | 3 (2+/1−) | Gồm nệm ghế (gắn xe), không gồm người đứng cạnh | Sạch, 1 polygon | `cyclo_img032.jpg` |
| img_004 tuk-tuk xanh lá bị người che | SAM 2.1 b | 3 (2+/1−) | Dừng đúng ở người che phía trước | Trùm luôn hành khách nhỏ bên trong (~x305) → trái 4.2.4 | `tuktuk_occluded_img004.jpg` |

- Đối chiếu 4.2: quy tắc "người là vật riêng" và 4.2.4 (không chồng mask) giúp quyết click âm; **thiếu** quy tắc cho người ngồi *dính* vào xe (tài xế tuk-tuk): ranh giới chân/dép với sàn xe không có sai số cho phép → đưa vào câu hỏi mở 4 (CAL-03).
- Interactor SAM trong CVAT: không thử (CVAT chưa cài Nuclio/serverless).

## 6. Calibration (Part 6)

- 6 quyết định (`calibration_decisions.csv`): 2 `loi_may`, 3 `cau_hoi_mo`, 1 `bay_lot`. Quan trọng nhất: **CAL-01** (tuk-tuk luôn `three_wheeler`, `car` ngưỡng 0.7) và **CAL-02** (three_wheeler phải có bánh + chỗ lái).
- Đã vào v1.1: chốt câu hỏi mở 2, 3, 4; loại trừ mới cho `car`/`three_wheeler`; bước kiểm cạnh box trong quy trình.
- Nguyên nhân gốc: CAL-01, CAL-02 là **máy sai** (v1.0 đã đủ, chỉ ghi rõ thêm); CAL-03/04/05 là **guideline mơ hồ**; CAL-06 là **người duyệt sai quy trình**.

## 7. QC và quyết định (Part 6)

- Mẫu: 8 ảnh (3 ảnh có `can_review` + 5 ngẫu nhiên, seed 42), 224 đối tượng; người kiểm: Claude (**hạn chế**: cùng là bên duyệt; ảnh dày chỉ kiểm ở độ phân giải tổng).
- Lỗi theo **đối tượng**: 5/224 = 2.2% · theo **ảnh**: 3/8 = 37.5% (ngưỡng Release ≤ 2%).
- Top lỗi: `thua` 3 (img_005 #41, #39 xe máy 0.12 trùng/rác; img_035 #580 người trùng), `sai_vi_tri_kich_thuoc` 1 (img_021 #279), `bo_sot` 1 (img_021 cụm xa chưa thành ignore_region).
- Giới hạn mẫu nhỏ: 3/8 ảnh lỗi → cận trên 95% (Wilson) **69%**; với 8 ảnh không thể chứng minh ≤ 2%.
- Script kiểm được số lượng, lớp, box trong ảnh; **không** kiểm được box có ôm sát vật thật, vật cả máy lẫn người cùng bỏ sót, và việc có theo checklist thật (2 ảnh chưa duyệt).

**Quyết định: Rework**, ngày 2026-11-17. Hold? Không: v1.1 đã chốt các ca gặp, ảnh không hỏng. Rework? Có: ảnh lỗi 3/8 > 2%, lỗi `thua` lặp lại, bắt bẫy 8/12 < 80%, 2/24 ảnh chưa duyệt.
Việc tiếp: duyệt img_014, img_021; chạy lại bước kiểm cạnh box trên cả lô; xoá box motorbike điểm < 0.2 trùng ở ảnh dày; export `qc_fixed.json`, chạy lại `build_release.py` và `qc_sample.py --force`.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (Claude Code): cài môi trường, tải model, tạo Project/Task CVAT qua API; chạy mọi lệnh Part 2, 3, 5 và viết nhận xét `model_log.csv` sau khi xem ảnh viz; chọn click SAM; **duyệt 22/24 ảnh batch** (nhiều agent AI song song, áp sửa qua API CVAT) — vì vậy bắt bẫy, AI error profile và thời gian `batch_review` là của AI, **không phải của học viên**; điền phiếu QC; viết calibration, guideline v1.1 và báo cáo này. Học viên: gán tay gold trong CVAT và giao việc. Kiểm tra: `validate_labels`, `check_submission`, đối chiếu box sửa bằng ảnh phóng to.
- **Gold có bị nhiễm không?** Học viên gán 55 box trước khi xem nhãn máy. Sau đó AI rà theo guideline, **chỉ nhìn ảnh gốc, không dùng pre-label** của ảnh gold, và sửa 14 nhãn: thêm người áo đỏ + ô tô đen cắt mép (img_012), 3 tài xế tuk-tuk (`can_review`, img_012), xe máy lề phải (img_036), 3 `ignore_region`; xoá 5 xe máy < 20 px (img_036). Gold cuối 59 đối tượng là **người + AI**.
- Máy: Windows 11, CPU Ryzen 5 PRO 215, 11.7 GB RAM, không GPU; CVAT v2.82.0 (tài liệu viết cho 2.81, giao diện khớp). Vướng: PowerShell chặn `Activate.ps1` (sửa bằng `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`); YOLOE lần đầu tự cài CLIP từ GitHub.
- Không dùng cache `data/cache/`: tự chạy trên CPU (0.3–1.6 s/ảnh; SAM 2.1 7.6 s/ảnh).
- Lỗi script: không gặp.
- Lần sau: tự duyệt batch để số bắt bẫy đo đúng bản thân, và chừa đủ thời gian để không bỏ ảnh chưa duyệt.
