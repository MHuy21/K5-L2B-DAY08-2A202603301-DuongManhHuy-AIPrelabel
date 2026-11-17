# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 622; đối tượng sau duyệt: 578; ngưỡng accepted: IoU >= 0.9
- Đã **loại 24 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)

| Trạng thái            | Số  | Tỉ lệ           | Mẫu số                  |
| --------------------- | --- | --------------- | ----------------------- |
| accepted (giữ nguyên) | 541 | 541/622 = 87.0% | 622 box máy             |
| edited (sửa)          | 15  | 15/622 = 2.4%   | 622 box máy             |
| deleted (xoá)         | 66  | 66/622 = 10.6%  | 622 box máy             |
| added (vẽ thêm)       | 22  | 22/578 = 3.8%   | 578 đối tượng sau duyệt |

Trong số deleted + added, **10 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 12.

### Theo lớp

| Lớp           | accepted | edited | deleted | added |
| ------------- | -------- | ------ | ------- | ----- |
| person        | 262      | 1      | 8       | 5     |
| bicycle       | 6        | 0      | 2       | 5     |
| motorbike     | 214      | 2      | 22      | 2     |
| car           | 43       | 5      | 6       | 3     |
| bus           | 3        | 0      | 0       | 3     |
| truck         | 2        | 0      | 4       | 0     |
| three_wheeler | 11       | 7      | 24      | 4     |

## 2. So với gold (nhãn gán tay từ đầu)

Ảnh gold: 4; đối tượng gold: 55 (mẫu số của recall)

|                 | TP  | FP  | FN  | Precision               | Recall                  |
| --------------- | --- | --- | --- | ----------------------- | ----------------------- |
| Máy (pre-label) | 42  | 35  | 13  | 54.5% (KTC 95%: 43–65%) | 76.4% (KTC 95%: 64–86%) |

_Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn._

(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy

- Bắt được 5/12 = 41.7% (KTC 95%: 19–68%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 1/3
  - wrong_class: 2/3
  - extra: 2/3
  - loose: 0/3
  - lọt: img_028.jpg (missing), img_003.jpg (extra), img_021.jpg (missing), img_008.jpg (wrong_class), img_005.jpg (loose), img_026.jpg (loose), img_034.jpg (loose)

## Gợi ý đọc số

- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
