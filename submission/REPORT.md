# Báo cáo Day 08 — 2A202603301 - Dương Mạnh Huy

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp                                  | Model + cỡ                                                                                     | Ngưỡng                  | Vì sao (dẫn `run_id` trong `model_log.csv`)                                                                                                                                                                                 |
| ------------------------------------ | ---------------------------------------------------------------------------------------------- | ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| person / bicycle / car / bus / truck | YOLO26m (`yolo26m.pt`, CPU)                                                                    | 0.3 (mặc định `--conf`) | r02/r03/r07. threshold_sweep: không ngưỡng ≤ 0.5 nào đạt recall 90% cho person (tối đa 0.84, 19 vật gold), bicycle (tối đa 0.50, 6 vật), car (tối đa 0.70, 10 vật); bus 1 vật, truck 0 vật trong gold (< 5) → giữ mặc định. |
| motorbike                            | YOLO26m (`yolo26m.pt`)                                                                         | 0.1                     | r02/r03/r07. threshold_sweep: ngưỡng cao nhất đạt recall ≥ 90% là 0.1 (P 0.46 / R 0.92, 12 vật gold); so với 0.3 (P 0.77 / R 0.83) chỉ thêm 1 TP nhưng thêm 10 FP (người duyệt phải xoá nhiều box hơn).                     |
| three_wheeler                        | YOLOE-26s (`yoloe-26s-seg.pt`), prompt: `tuk-tuk`, `cyclo`, `cargo tricycle` → `three_wheeler` | 0.15                    | r04/r08. threshold_sweep: P 0.78 / R 1.00 (7 vật gold). YOLO26 không có lớp xe ba bánh (r02). Khi merge, YOLOE được ưu tiên (`--iou 0.6`).                                                                                  |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` → 726 box cho 28 ảnh (batch + gold).
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: **SAM 2.1** (`sam2.1_b.pt`). Trên img_013, mask người lái (bị cột mui che) của SAM 2.1 phủ ~10% người so với ~5% của SAM 1 (`mobile_sam.pt`); trên img_004 hai bản gần như nhau; SAM 2.1 chậm hơn ~1.9× trên CPU (3.76 vs 2.03 s/ảnh) nhưng là bản lab khuyến nghị và không cần quyền truy cập như SAM 3 (r05, r06).
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? YOLO26 không có lớp xe ba bánh: xích lô img_032 → `bicycle 0.70` (vẫn còn ở conf 0.5), 3 xích lô img_007 → `motorbike 0.28–0.38`, tuk-tuk img_002 không có box nào. Cách sửa: dùng YOLOE (open-vocabulary) cho `three_wheeler` và để `merge_prelabels.py` cho `three_wheeler` thắng box `car`/`truck`/`bicycle` chồng lên (IoU ≥ 0.6); phần còn sót thì người duyệt đổi lớp (đếm số bánh, guideline 4.3).
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): điểm cao mà sai: img_032 `bicycle 0.70` (xích lô, đúng là `three_wheeler`), img_002 `truck 0.73` (xe bán tải, đúng là `car`); đối chứng: img_011 `truck 0.77` là `truck` đúng. Điểm thấp mà đúng: img_030 `motorbike 0.47` (xe tay ga tiền cảnh rất rõ, mất ở conf 0.5), img_007 các xe máy ở xa 0.40–0.49. → Điểm `conf` không bảo đảm nhãn đúng.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: ngưỡng được **chọn và chấm trên cùng** 4 ảnh gold (55 đối tượng) nên precision / recall của máy trên gold là số in-sample, cao hơn hiệu năng thật trên 24 ảnh batch; lớp ít vật (bus 1, truck 0, bicycle 6) chỉ là ước lượng thô. Muốn đo chặt hơn cần một tập pilot riêng (ADVANCED.md mục 3).

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số  | Tỉ lệ | Mẫu số                  |
| ---------- | --- | ----- | ----------------------- |
| accepted   | 541 | 87.0% | 622 box máy             |
| edited     | 15  | 2.4%  | 622 box máy             |
| deleted    | 66  | 10.6% | 622 box máy             |
| added      | 22  | 3.8%  | 578 đối tượng sau duyệt |

(Đã loại 24 đối tượng quanh vùng bẫy khỏi bảng; 10 trường hợp deleted + added là đổi lớp → máy sót thật ≈ 22 − 10 = 12.)

Theo lớp (accepted / edited / deleted / added): person 262/1/8/5 · bicycle 6/0/2/5 · motorbike 214/2/22/2 · car 43/5/6/3 · bus 3/0/0/3 · truck 2/0/4/0 · three_wheeler 11/7/24/4.

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? **`three_wheeler`**: chỉ giữ nguyên 11/42 box máy (26.2%), xoá 24/42 (57.1%), sửa 7. Nguyên nhân: ngưỡng 0.15 (chọn để đạt recall 100% trên 7 vật gold) cho YOLOE gán `three_wheeler` cho cả xe 4 bánh và vật không phải xe (QC: img_003 #2 xe tải thùng, #22 sạp quán; pilot: xe bán tải img_002 0.37, xe đẩy hàng img_002 0.40). Khi merge, YOLOE được ưu tiên và 22 box YOLO26 bị loại vì chồng lên box ưu tiên cao hơn (car ×11, truck ×6…), trong đó có thể có box `car`/`truck` đúng bị `three_wheeler` sai đè. Lớp kém thứ hai là `truck`: giữ 2/6 (33.3%). Ngược lại `person` giữ 262/271 (96.7%), `motorbike` 214/238 (89.9%).
- So với gold: precision / recall của **máy** 54.5% (42/77, KTC 95% 43–65%) / 76.4% (42/55, KTC 95% 64–86%) (mẫu số: 55 đối tượng gold, 4 ảnh)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 5/12 = 41.7% (KTC 95% 19–68%; ngưỡng đạt 80%); theo loại: missing 1/3, wrong_class 2/3, extra 2/3, loose 0/3; bẫy lọt: img_028 (missing), img_021 (missing), img_003 (extra), img_008 (wrong_class), img_005 / img_026 / img_034 (loose); vì sao (bạn đã bỏ bước nào của checklist)? **Bước 4 (lỏng)** gần như bị bỏ: loose 0/3, tôi chỉ xem box có nằm đúng vật không chứ không kéo kiểm 4 cạnh. **Bước 0–1 (quét trước, tìm sót)** làm vội ở ảnh đông: 2 bẫy missing lọt đều ở ảnh dày xe máy (img_021, img_028). Bẫy extra (img_003) và wrong_class (img_008) lọt vì box trông hợp lý nên tôi giữ mà không đối chiếu lại. Tôi duyệt 24 ảnh liên tục 40 phút không nghỉ (1.67 phút/ảnh, dưới mức ~3 phút/ảnh GUIDE gợi ý).
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Accepted 541/622 = 87.0% nhưng chỉ bắt 5/12 bẫy: phần lớn box được giữ vì **trông hợp lý**, không phải vì đã kiểm, tức là tôi bị thiên kiến tự động hóa. Accepted cao ở đây phản ánh mức độ tôi tin máy nhiều hơn là chất lượng máy. `added` thấp (22/578 = 3.8%) cũng không có nghĩa máy ít sót: recall của máy trên gold chỉ 42/55 = 76.4% (13 vật sót) và 2/3 bẫy missing lọt, nên nhiều vật sót đã không được tôi vẽ thêm.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 20 phút / 4 ảnh = 5 phút/ảnh vs duyệt 40 phút / 24 ảnh = 1.67 phút/ảnh (không nghỉ). Thiên lệch (bạn gán gold trước nên đã quen cảnh): mức tiết kiệm ~67% là **lạc quan**: (1) gán gold trước nên khi duyệt tôi đã quen kiểu ảnh và guideline; (2) 4 ảnh gold và 24 ảnh batch không cùng độ khó; (3) duyệt nhanh một phần vì duyệt chưa kỹ (bẫy 5/12, loose 0/3), nếu duyệt đúng checklist thì thời gian/ảnh sẽ cao hơn.
- Kết luận: **có, nhưng chỉ cho `person` và `motorbike`** (kèm checklist chặt); **không** cho `three_wheeler` và `truck`, dựa trên accepted theo lớp: person 262/271 = 96.7%, motorbike 214/238 = 89.9%, car 43/54 = 79.6%, trong khi three_wheeler 11/42 = 26.2% và truck 2/6 = 33.3% (dưới 50%: sửa từng box lâu hơn vẽ mới). Với three_wheeler cần nâng ngưỡng YOLOE (CAL-04) hoặc gán tay. Thời gian giảm từ 5 xuống 1.67 phút/ảnh, nhưng chỉ có ý nghĩa nếu chất lượng duyệt đạt (hiện bắt bẫy 5/12 < 80%).
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Ẩn toàn bộ nhãn máy và tự đếm vật theo lớp trước khi hiện nhãn ở **24/24 ảnh**, ghi số đếm vào log review (kiểm bằng log: đủ 24 dòng có số đếm).
  2. Mỗi box: zoom ≥ 200% và kiểm đủ 4 cạnh (bước 4) trước khi sang box tiếp; mục tiêu vòng sau **bắt ≥ 2/3 bẫy loose** (kiểm bằng `review_stats.md`).
  3. Nghỉ 5 phút sau mỗi 10 ảnh và không duyệt nhanh hơn **2.5 phút/ảnh** ở 6 ảnh dày xe máy (kiểm bằng `time_log.csv`: có dòng `break`, phút/ảnh).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh     | SAM (phiên bản)         | Số click          | Mask gồm người lái / hàng hoá?                                            | Lỗi quan sát                                                    | File trong `masks/`           |
| ------- | ----------------------- | ----------------- | ------------------------------------------------------------------------- | --------------------------------------------------------------- | ----------------------------- |
| img_013 | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Có: gồm gần hết người lái + người ngồi phía sau                           | Một click âm chỉ khoét một mảng áo người lái                    | `tuktuk_img013_sam2.jpg`      |
| img_013 | SAM 2.1                 | 6 (2 dương, 4 âm) | —                                                                         | Mask vỡ (13.0% ảnh, 2 polygon): chỉ còn mui + viền, mất thân xe | `tuktuk_img013_sam2_neg4.jpg` |
| img_013 | SAM 2.1                 | 5 (3 dương, 2 âm) | Còn dính đầu, áo, bàn chân người lái + người phía sau; đã loại quần + tay | Tốt nhất trong 3 lần thử trên img_013                           | `tuktuk_img013_sam2_p3n2.jpg` |
| img_004 | SAM 2.1                 | 3 (2 dương, 1 âm) | Không: người đứng cạnh nằm ngoài mask                                     | Click dương rơi vào bóng dưới gầm → mask dính vệt bóng          | `tuktuk_img004_sam2.jpg`      |
| img_017 | SAM 2.1                 | 3 (2 dương, 1 âm) | Không có người lái                                                        | Click âm rơi trên mui nhưng mui vẫn trong mask                  | `tuktuk_img017_sam2.jpg`      |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? **Giúp:** mục 2 (người là vật riêng) cho biết mask `three_wheeler` không được gồm người lái → cả 3 lần thử trên img_013 đều vi phạm ở mức khác nhau; 4.2.3 (bóng đổ không thuộc vật) cho biết mask img_004 dính vệt bóng dưới gầm là sai. **Thiếu:** 4.2.4 chỉ nói "vùng chạm thuộc vật ở phía trước", chưa nói trường hợp người ngồi **bên trong** khung xe (người lái tuk-tuk ở img_013 vừa trước vừa sau các thanh mui); guideline cũng chưa hướng dẫn cách đặt click (thêm 4 click âm làm mask vỡ, img_013 `neg4`), nên cần ghi rõ: click âm đặt vào thân người, tối đa 2–3 click, kiểm sau mỗi click.
- _(Tuỳ chọn)_ Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: Không thử: CVAT Docker mặc định chưa cài model AI (cần serverless/Nuclio); phần "hiểu" SAM dùng `sam2_points.py` thay thế.

## 6. Calibration (Part 6)

- Số quyết định: 4 (`calibration_decisions.csv`: CAL-01, CAL-02 `cau_hoi_mo`; CAL-03, CAL-04 `loi_may`); quan trọng nhất (dẫn `CAL-xx`): **CAL-01** (cụm xe máy dày → `ignore_region`) vì ảnh hưởng nhiều nhãn nhất: batch có 13 box < 20 px (img_018 có 6) nhưng 0 `ignore_region`, và 2 bẫy missing lọt đều ở ảnh dày xe máy (img_021, img_028).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: CAL-01 (cụm xe máy dày, kể cả gần camera → `ignore_region`, bổ sung 4.1.6); CAL-02 (khay / giỏ gắn cố định thuộc box xe, bổ sung 4.1.5); CAL-03 (thêm ví dụ xích lô img_032, tuk-tuk img_002 vào mục 2); CAL-04 không đổi guideline (recipe: nâng ngưỡng `three_wheeler` > 0.15)
- Nguyên nhân gốc: chủ yếu **máy** (YOLO26 không có lớp xe ba bánh; YOLOE ở ngưỡng 0.15 gán nhầm xe 4 bánh) và **người duyệt** (không áp quy tắc 4.1.6 về vật < 20 px, bỏ bước 4); **guideline mơ hồ** chỉ ở 2 ca: câu hỏi mở 2 (cụm xe gần camera) và 3 (khay hàng nhô ra).

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42: img_003, img_006, img_014, img_031, img_035; 119 đối tượng); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: 13/119 = 10.9% · theo **ảnh**: 5/5 = 100% (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: `thua` 6 (img_003 #23 box `truck` trên lan can xe ba gác; img_014 #155 box trùng; img_035 #562) · `sai_vi_tri_kich_thuoc` 5 (img_006 box bao 2–3 xe máy đỗ; img_031 #496 box người thiếu phần chân) · `sai_lop` 2 (img_003 #2 xe tải thùng gán `three_wheeler`)
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): 5/5 ảnh lỗi → cận trên Wilson 95% là 100%. Mẫu chỉ 5 ảnh: kể cả 0/5 ảnh lỗi thì cận trên vẫn ~43%, nên ngưỡng ≤ 2% **không thể được chứng minh** bằng mẫu này; kết quả 5/5 chỉ cho thấy lỗi xuất hiện ở hầu hết ảnh, không đo chính xác được tỉ lệ lỗi thật của cả lô.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): script **kiểm được** tên file, lớp có trong taxonomy, box nằm trong ảnh, box < 20 px, đủ ảnh, accepted / edited / deleted / added, bắt bẫy và các số đo QC; script **không kiểm được** box có ôm sát vật thật không, lớp có đúng thực tế không, vật mà cả người lẫn máy cùng bỏ sót (ngoài gold), và người duyệt có theo checklist thật hay không → cần mắt người (phiếu QC).

**Quyết định: Rework**, ngày 2026-11-17. Lý do (đi theo Hold → Rework → Release): **Không Hold**: còn 5 câu hỏi mở (1, 4, 5, 6, 7) chưa chốt, nhưng không lỗi nào trong 13 lỗi QC và 7 bẫy lọt bắt nguồn từ các câu này; mọi lỗi đều sửa được theo v1.0 / v1.1, dữ liệu ảnh không hỏng. **Rework**: ảnh có lỗi 5/5 = 100% > 2% và bắt bẫy 5/12 = 41.7% < 80% (lô duyệt chưa đáng tin). Lỗi `thua` 6/13 = 46% chưa tới ngưỡng "lỗi hệ thống" (≥ 50%) nhưng là loại lặp nhiều nhất. **Không Release** vì không đạt ngưỡng nào.
Nếu Rework / Hold, việc cần làm tiếp: (1) sửa 13 lỗi QC trong CVAT (box thừa / trùng, box bao nhiều xe ở img_006, box người img_031); (2) áp v1.1: CAL-01 đổi các cụm xe < 20 px thành `ignore_region` (img_018, img_027…), CAL-02 mở box xe đạp img_031 #498 gồm khay; (3) duyệt lại cả 24 ảnh theo đủ checklist, nhất là bước 0 và bước 4, áp 3 quy tắc ở mục 4; (4) export `qc_fixed.json` → `build_release.py` → `qc_sample.py --force` → QC vòng 2; (5) lô sau nâng ngưỡng `three_wheeler` > 0.15 (CAL-04).

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không):
  - Công cụ: Claude Code (Kiểm tra setup môi trường, phân tích số liệu tính toán, chẩn đoán lỗi `ModuleNotFoundError `- do chưa kích hoạt venv).

- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? **Không xem pre-label**: chưa chạy model nào lên 4 ảnh gold cho tới khi export `gold_manual.json`. Trong lúc gán có hỏi Claude về **quy tắc guideline** (người ngồi trong ô tô, `ignore_region`, xe máy có gồm gương, hai người đèo nhau) và có gửi 1 ảnh cắt hỏi; Claude chỉ giải thích quy tắc, không quyết nhãn. Mọi box gold do tôi tự vẽ.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, CPU only (AMD Radeon 780M, không CUDA), RAM 32 GB, Python 3.12.10 + torch 2.15 CPU, CVAT v2.81.0 Docker. Vướng mắc: trọng số tải sẵn nằm ngoài repo nên phải copy vào `models/` + tải thêm `yolo26m.pt`; PowerShell chưa kích hoạt venv → `ModuleNotFoundError: numpy`.
- Dùng cache `data/cache/` thay vì tự chạy? Không: tự chạy YOLO26m và YOLOE trên 36 ảnh (r07, r08) trên CPU.
- Lỗi / bug script đã gặp (nếu có): Không gặp bug script.
- Một điều bạn làm khác lần sau: chia lô duyệt thành nhóm 10 ảnh có nghỉ, và với ảnh dày xe máy thì quyết `ignore_region` / `can_review` ngay khi phân vân thay vì giữ nguyên box máy.
