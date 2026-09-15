# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên: `Nguyễn Đình Bảo Phúc'
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT Track Mode; validator local |
| Thời gian gán `clip_02` (warm-up) | 16:30 |
| Thời gian gán `clip_01` | 17h:15 |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | 7.5 |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. ID 3 kéo dài frame 1–190 và có cảnh báo static, nên phải kiểm bằng mắt trước khi Outside.
2. ID 1 kết thúc ở frame 7 khi xe ra rìa ảnh, không kéo bbox sang frame sau.
3. ID 8 chỉ có frame 147–159; cần xem lại entry/exit vì evaluator báo thiếu một phần track tham chiếu.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Validator không có ID trùng trong cùng frame; dùng 8 ID.
- Lượt 2: Kiểm span MOT và các ranh giới của ID 1, 5, 6, 8.
- Lượt 3: Đã tạo visualization tại các frame 1, 31, 59, 94, 113, 140, 190.

Kiểm chéo với: Chưa thực hiện. Chi tiết ở `reports/review_partner.md` khi có reviewer.
Số lỗi bạn tìm được trong bản của bạn ấy: Chưa có. Số lỗi bạn ấy tìm được trong bản của bạn: Chưa có.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Chưa có dữ liệu so sánh giữa hai người. Luật occlusion 25 frame và entry/exit đã được ghi trong `GUIDELINE_MINI.md`.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `e7f9c26b84023557335f1675bf5cdc49f45c0a87e1a356f21c3f5d3c4038d88a` |
| Thời điểm khóa | 2026-09-15 UTC; xem `manifest.json` |
| Số row / frame / track trước khi mở reference | 487 / 190 / 8 |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.696 | 0.673 | 0.724 | 0.826 | 0.915 | 0.843 | 0.803 | 2 | 88 | 0 |
| Sau rework | Chưa rework | - | - | - | - | - | - | - | - | - |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có với bản pre-gold**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Thiếu đoạn track | Theo diagnostics: gold 2, 5, 8 | Chưa rework trong CVAT | Chưa sửa trực tiếp MOT |
| Bbox static | 1–15 | ID 3 | Cần kiểm bằng mắt |
| Peer finding | Chưa có | Chưa có | Chưa review |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.12.9 / 8.4.150 / 2.14.0+cpu / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` / `configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / `[2, 5, 7]` |
| device | CPU |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.696 | 0.673 | 0.724 | 0.826 | 0.915 | 0.843 | 0.803 | 2 | 88 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.764 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.664 | 0.594 | 0.748 | 0.838 | 0.846 | 0.645 | 0.813 | 162 | 11 | 0 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA 0.843 thấp hơn IDF1 0.915. Identity khá ổn nhưng còn 88 FN; MOTA tổng hợp FP, FN và ID switch nên không diễn giải đơn độc chất lượng identity.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

ReID tăng IDF1 từ 0.875 lên 0.900 và AssA từ 0.776 lên 0.820; IDSW vẫn là 2. Treatment tốt hơn trên clip này, nhưng đây là system comparison vì ByteTrack và BoT-SORT là hai implementation khác nhau, không cô lập causal effect của ReID.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

ReID tăng DetA 0.649 -> 0.711, giảm FN 54 -> 26 nhưng FP tăng 88 -> 91. Phần còn lại liên quan cả detector coverage và association.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Evaluator cho thấy ReID có các bbox không khớp annotation quanh những đoạn như frame 140 của model ID 34. Đây là candidate để xem bằng visualization, chưa đủ để kết luận annotation sai.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

ReID bắt được các đoạn annotation chưa phủ, đặc biệt quanh frame 113 của track gold 6. Cần xem lại CVAT, không copy model output thành nhãn.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Giữ ngưỡng occlusion 25 frame, ghi frame–ID cho mọi ca mơ hồ, đặt keyframe dày hơn ở crossing/midpoint và chạy validator sau mỗi export. Peer review và thời gian thao tác phải được ghi trước khi nộp.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md` — chưa có reviewer và finding thực tế
- [x] `reports/REPORT.md` — bản báo cáo sau rework
