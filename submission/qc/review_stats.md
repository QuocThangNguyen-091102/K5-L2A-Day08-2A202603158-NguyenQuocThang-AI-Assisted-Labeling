# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 593; đối tượng sau duyệt: 590; ngưỡng accepted: IoU >= 0.9
- Đã **loại 31 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 495 | 495/593 = 83.5% | 593 box máy |
| edited (sửa) | 26 | 26/593 = 4.4% | 593 box máy |
| deleted (xoá) | 68 | 68/593 = 11.5% | 593 box máy |
| added (vẽ thêm) | 68 | 68/590 = 11.5% | 590 đối tượng sau duyệt |

Trong số deleted + added, **11 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 57.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 255 | 8 | 9 | 27 |
| bicycle | 5 | 0 | 3 | 4 |
| motorbike | 195 | 8 | 29 | 8 |
| car | 28 | 0 | 1 | 19 |
| bus | 3 | 0 | 0 | 3 |
| truck | 1 | 1 | 3 | 3 |
| three_wheeler | 8 | 9 | 23 | 4 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 56 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 51 | 22 | 5 | 69.9% (KTC 95%: 59–79%) | 91.1% (KTC 95%: 81–96%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 2/3
  - loose: 0/3
  - wrong_class: 3/3
  - extra: 3/3
  - lọt: img_024.jpg (loose), img_021.jpg (loose), img_006.jpg (loose), img_034.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
