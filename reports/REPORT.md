# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: Nghiem Tra My / G-03 t-020
Ngày: 15/9/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `...` |
| Thời gian gán `clip_02` (warm-up) | 15 phút |
| Thời gian gán `clip_01` | 35 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | `...` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

- Xe rời khung hình (entry/exit) không bấm Outside đúng lúc — đây là nhóm lỗi lớn nhất trong bản pre-gold (6 mục "bbox treo/bbox thừa"), ví dụ ID 6 còn bbox từ frame 79–100 (22 frame) trước khi track gold 6 thật sự xuất hiện, và ID 4 còn bbox thừa ở frame 149–151 sau khi xe đã rời khung. Cách xử lý: rà lại từng frame quanh mốc xe vào/ra khung và bấm Outside đúng frame xe biến mất hoàn toàn, thay vì để track "trôi" thêm vài frame.
- Bbox trôi ở giữa hai keyframe khi xe đổi tốc độ/hướng — 5 trường hợp IoU tụt còn 0.54–0.59 so với gold, tập trung quanh frame 120–122 (track 5, 6). Cách xử lý: thêm keyframe dày hơn ở đúng đoạn xe đổi hướng/tốc độ thay vì để CVAT nội suy tuyến tính qua một khoảng dài.
- Xác định đúng frame xe "đủ rõ" để bắt đầu track khi xe vừa xuất hiện còn nhỏ/mờ ở rìa khung — đây là nguồn gây lệch nhỏ về số bbox tổng (617 bbox của bạn so với 573 của gold) dù số track đã khớp hoàn toàn.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1 — identity/timeline: kiểm số ID có nhấp nháy/đổi bất thường hay không;
tập trung vào crossing và occlusion.

- Lượt 2 — frame đầu/cuối: phát hiện các bbox entry/exit không đúng, đặc biệt
ID 6 (79–100), ID 4 (51–53 và 149–151), ID 5 (76–78), ID 7 (103–105) và
ID 8 (169–171).

- Lượt 3 — geometry/interpolation: phát hiện bbox drift tại frame 106 và
frame 120–122; đây là các vị trí cần thêm keyframe.

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `...` |
| Thời điểm khóa | `...` |
| Số row / frame / track trước khi mở reference | 617 bbox · 190 frame · 8 track |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.714 | 0.697 | 0.737 | 0.798 | 0.956 | 0.909 | 0.766 | 48 | 4 | 0 |
| Sau rework | | | | | | | | | | |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): có — cả ba chỉ số đều đạt ngay ở bản pre-gold (IDF1 0.956, MOTA 0.909, MOTP 0.766), notebook in rõ => ĐẠT — sang bước chạy model ở cell 10. Theo bảng mức chất lượng trong RUBRIC.md, HOTA 0.714 / IDF1 0.956 / MOTA 0.909 / LocA 0.798 nằm ở mức "Đạt" (đã vượt ngưỡng learning gate; để lên mức "Xuất sắc" cần HOTA >= 0.80, hiện đang thiếu chủ yếu ở DetA 0.697 — tức còn bỏ sót/thừa bbox hơn là lỗi ID).

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo trước khi xe xuất hiện | 79–100 | ID 6 | Cần bấm Outside đúng frame xe rời khung |
| Bbox treo sau khi xe đã rời khung | 149–151 | ID 4 | Cần bấm Outside sớm hơn |
| Bbox trôi (IoU tụt còn 0.54–0.59) | 120–122 | ID 5, ID 6 | Cần thêm keyframe |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | Python 3.13.15 / ultralytics 8.4.145 / torch 2.11.0+cpu / lap 0.5.13 |
| weights / hai tracker | yolo26n.pt / ByteTrack control (bytetrack.yaml) và BoT-SORT + ReID treatment (configs/trackers/botsort-reid.yaml) |
| conf / IoU / imgsz / classes | conf=0.25 / iou=0.70 / imgsz=960 / classes=[2, 5, 7] (car, bus, truck theo COCO) |
| device | CPU |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.714 | 0.697 | 0.737 | 0.798 | 0.956 | 0.909 | 0.766 | 48 | 4 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.665 | 0.610 | 0.734 | 0.811 | 0.859 | 0.715 | 0.786 | 98 | 77 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA là 0.909, thấp hơn IDF1 (0.956). Điều này cho thấy annotation có identity consistency rất tốt (IDSW = 0); phần điểm MOTA giảm chủ yếu do FP (48 bbox) và FN, thay vì lỗi ID.

Nhìn chung, trường hợp MOTA cao nhưng IDF1 thấp xảy ra khi tracker phát hiện đúng object ở từng frame nhưng thường xuyên đổi ID. MOTA gộp FP, FN và IDSW thành tổng lỗi, trong khi IDF1 đánh giá khả năng duy trì identity xuyên suốt track, nên việc một track bị tách thành nhiều ID có thể làm IDF1 giảm đáng kể.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

So với gold, ReID cải thiện IDF1 từ 0.875 → 0.900 và AssA từ 0.776 → 0.820, trong khi IDSW vẫn là 2. Điều này cho thấy ReID có xu hướng duy trì association tốt hơn, dù không làm giảm số lần ID switch.

ByteTrack bị tách các gold track 4, 5 và 7, trong khi ReID có coverage tốt hơn ở một số đoạn; ví dụ gold track 6 được phủ 44/56 frame thay vì 42/56 với ByteTrack. Tuy nhiên, ReID cũng có lỗi tại frame 87, khi ID chuyển 17 → 18.

Lưu ý rằng đây không phải causal ablation riêng của ReID, vì ByteTrack và BoT-SORT là hai tracker khác nhau với cơ chế association khác nhau.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng từ 0.649 → 0.711, FN giảm từ 54 → 26, trong khi FP tăng nhẹ từ 88 → 91. Như vậy, ReID treatment giúp giảm đáng kể số object bị bỏ sót nhưng vẫn tạo thêm một lượng nhỏ false positive.

Hai run sử dụng cùng detector (yolo26n.pt, conf=0.25, IoU=0.70, imgsz=960), nên khác biệt chủ yếu liên quan đến association và tracking, đặc biệt là khả năng duy trì hoặc tách track. Tuy nhiên, không thể quy toàn bộ chênh lệch cho ReID vì hai tracker không giống nhau.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Tại frame 87, gold track 5, annotation giữ nguyên identity trong khi ReID chuyển ID 17 → 18, tạo một ID switch. Đây là trường hợp annotation thủ công duy trì identity tốt hơn model, có thể do con người sử dụng được ngữ cảnh chuyển động trước và sau occlusion/crossing mà appearance embedding không nắm bắt đầy đủ.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Khoảng frame 104–111, bảng disagreement cho thấy ReID liên tục có nhiều bbox hơn annotation. Đây là vùng nên kiểm tra lại trong CVAT, đặc biệt quanh frame 106–121, nơi model tạo thêm track 27.

Tuy nhiên, chưa đủ evidence để kết luận annotation sai. Cần kiểm tra lại các frame này để xác định đó là vehicle thực sự bị bỏ sót hay false positive của detector. Vì vậy, đây nên được xem là review candidate, không phải lỗi annotation đã được xác nhận.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- Entry/exit là checkpoint riêng: với mỗi track, kiểm frame bắt đầu và frame kết thúc ngay sau khi vẽ; dùng Outside đúng lúc xe rời khung.

- Adaptive keyframing: không đặt keyframe theo một khoảng frame cố định.
Xe đi thẳng có thể thưa; xe rẽ/phanh/occlusion cần dày hơn.

- Bắt buộc review giữa keyframe: sau khi tạo keyframe, tua vào giữa đoạn dài nhất để phát hiện interpolation drift.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [ ] `reports/REPORT.md` (file này)
