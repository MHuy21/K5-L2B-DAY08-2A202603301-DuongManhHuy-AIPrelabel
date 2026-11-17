# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603301 - Dương Mạnh Huy

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.
>
> **Phạm vi áp dụng:** v1.1 áp dụng cho lần gán / sửa nhãn tiếp theo. Nhãn `batch_reviewed.json` đã được gán theo v1.0, nên những chỗ chưa khớp v1.1 (CAL-01, CAL-02) nằm trong danh sách Rework của `REPORT.md` mục 7.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày **gần camera** (không phải ở xa): vẽ từng xe hay `ignore_region`? | Xe cao < 20 px **hoặc** cụm ≥ 5 xe chồng nhau không tách được từng chiếc → vẽ **1 `ignore_region`** bao cả cụm, **kể cả khi cụm ở gần camera**. Xe ≥ 20 px tách rõ vẫn vẽ từng box. (Bổ sung mục 4.1.6) | img_018, img_027, img_006 | CAL-01 |
| 3 | Hàng nhô ra: xe đạp / ba gác chở hàng nhô xa, box xe có gồm hàng không? | **Khay / giỏ gắn cố định** vào xe thuộc box xe, **kể cả phần nhô ra** ngoài khung xe (ngưỡng ~20% ở mục 4.1.5 không áp dụng cho khay / giỏ gắn cố định). | img_031 (khay cà chua trước xe đạp) | CAL-02 |

Câu hỏi mở **chưa chốt** trong v1.1: câu 1 (xe đạp điện), 4 (ngưỡng "còn thấy ≥ 20%" cho người trong xe có mui), 5 (kính chắn gió có thuộc mask xe), 6 (box chồng nhau trong ảnh đêm), 7 (mức `conf` tối thiểu). Giữ nguyên như v1.0, để lại cho vòng calibration sau.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_032, img_007, img_002 | Xích lô và tuk-tuk trong ảnh phố | YOLO26 (80 lớp COCO, không có xe ba bánh): xích lô → `bicycle 0.70` (img_032), xích lô đỗ vỉa hè → `motorbike 0.28–0.38` (img_007); tuk-tuk img_002 không có box | Giữ quy tắc mục 2 (xích lô, tuk-tuk, ba gác = `three_wheeler`); bổ sung ảnh ví dụ để người duyệt nhận ra | Mục 2, lớp `three_wheeler`: thêm ví dụ **xích lô** img_032, **tuk-tuk** img_002 (CAL-03) |
| img_003, img_002, img_011 | Xe 4 bánh (xe tải thùng, bán tải, xe tải) | YOLOE gán `three_wheeler` (img_003 0.61, img_002 0.37, img_011 0.28) | Không đổi guideline (quy tắc lớp đã rõ); xử lý bằng recipe: nâng ngưỡng `three_wheeler` > 0.15 cho lô sau (REPORT mục 1) | Không có quy tắc mới (CAL-04) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở 2 (cụm xe máy dày gần camera → `ignore_region`, bổ sung 4.1.6) và 3 (khay / giỏ gắn cố định thuộc box xe, bổ sung 4.1.5); thêm ảnh ví dụ xích lô / tuk-tuk vào mục 2 | CAL-01, CAL-02, CAL-03 (CAL-04: không đổi guideline, chỉ đổi recipe) |
