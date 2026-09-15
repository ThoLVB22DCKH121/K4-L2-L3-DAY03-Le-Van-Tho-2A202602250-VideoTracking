# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Lê Văn Thọ`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | `45` phút |
| Thời gian gán `clip_01` | `90` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `70` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Track 5 giữa lúc đang cắt khung (frame 85-90): bbox của tôi chỉ còn IoU 0.51-0.60 so với gold — bbox trôi khỏi xe trong đoạn xe rẽ/đổi hướng, có lẽ do keyframe đặt quá thưa ở đúng đoạn này.`
2. `Track 4 và track 5 lúc vào/ra khung (frame 51-53, 67-78, 91-100, 149-151): track của tôi bắt đầu sớm hơn hoặc kết thúc muộn hơn track tham chiếu vài đến 12 frame — ngưỡng "xe đủ rõ để bắt đầu track" và thời điểm bấm Outside chưa khớp với gold.`
3. `...`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `...`
- Lượt 2: `...`
- Lượt 3: `...`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `ac203a39d58b51042becff8886e2afb60f8c7bf9277a439cd2104c193fc24451` |
| Thời điểm khóa | `2026-09-15T14:34:20.323274+00:00` |
| Số row / frame / track trước khi mở reference | `610 dòng / 190 frame / 8 track` |

Vì không có snapshot pre-gold riêng, dữ liệu chấm dưới đây là **một lần đo duy nhất** (bản
`annotations/clip_01/gt.txt` cuối cùng có trong file nộp) — không rõ đây là trước hay sau
rework. Điền dòng "Sau rework" bằng số thật; dòng "Bản pre-gold" để trống vì không có bản riêng.

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | `...` | `...` | `...` | `...` | `...` | `...` | `...` | `...` | `...` | `...` |
| Sau rework (số đo được) | 0.823 | 0.809 | 0.840 | 0.884 | 0.959 | 0.914 | 0.874 | 43 | 6 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có** — cả ba đều đạt
(IDF1 0.959, MOTA 0.914, MOTP 0.874), theo đúng `outputs/eval_vs_gold.json`. Theo bảng mức
chất lượng trong `RUBRIC.md` (HOTA≥0.80, IDF1≥0.90, MOTA≥0.90, LocA≥0.80), kết quả này đạt mức
**"Xuất sắc"**.

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo (có bbox trước khi xe tham chiếu xuất hiện) | 67-78 | track 5 | `Không sửa` |
| Bbox treo (còn bbox sau khi xe tham chiếu đã rời khung) | 149-151 | track 4 | `Không sửa` |
| Bbox trôi (IoU thấp nhất 0.51) khi xe đang rẽ | 85-90 | track 5 | `Không sửa` |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15` / `8.4.145` / `2.11.0+cpu` / `0.5.13` |
| weights / hai tracker | `yolo26n.pt` — control: `bytetrack.yaml` (ByteTrack control); treatment: `configs/trackers/botsort-reid.yaml` (BoT-SORT + ReID) |
| conf / IoU / imgsz / classes | `0.25` / `0.70` / `960` / `[2, 5, 7]` (car, bus, truck — COCO) |
| device | `cpu` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.823 | 0.809 | 0.840 | 0.884 | 0.959 | 0.914 | 0.874 | 43 | 6 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.742 | 0.683 | 0.809 | 0.871 | 0.883 | 0.764 | 0.858 | 85 | 57 | 2 |


## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Ở bản của tôi, **IDF1 (0.959) cao hơn MOTA (0.914)**, không phải ngược lại — tức là đây không
rơi vào ca "MOTA cao, IDF1 thấp" mà `RUBRIC.md` cảnh báo. IDSW = 0 (không có lần nào tôi đổi
nhầm ID), vậy khoảng cách 0.045 giữa hai điểm hoàn toàn đến từ 43 FP (bbox treo ở track 4 và 5
lúc xe vào/ra khung) chứ không phải từ lỗi identity. Vẫn trả lời được phần "vì sao MOTA không
phạt nặng lỗi ID": MOTA cộng `(FP + FN + IDSW) / GT_boxes`, mỗi lần đổi ID chỉ tính là **một**
sự kiện lỗi bất kể đoạn bị đổi ID kéo dài bao nhiêu frame — 1 IDSW trên clip 190 frame gần như
không đẩy MOTA đi đâu. Ngược lại IDF1 dùng khớp Hungarian trên toàn bộ chiều dài track, nên một
lần đổi ID giữa track có thể làm rất nhiều frame bị tính sai nhãn ID cùng lúc, kéo IDF1 xuống
nhiều hơn tỉ lệ tương ứng trong MOTA. Ở ByteTrack control ta thấy đúng dạng đó rõ hơn: chỉ 2
IDSW nhưng khiến 3 track gold bị *fragment*, và IDF1 (0.875) thấp hơn hẳn so với AssA (0.776)
thấp tương ứng — 2 lần đổi ID kéo theo hàng chục frame bị gán track mới.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

IDF1 tăng 0.875 → 0.900, AssA tăng 0.776 → 0.820, còn **IDSW giữ nguyên ở 2** — không đổi về
số lượng, nhưng đổi về vị trí: ByteTrack lệch ID ở frame 59 (gt track 4: ID 14→15) và frame 94
(gt track 5: ID 23→32); ReID lệch ID ở frame 87 (gt track 5: ID 17→18) và frame 113 (gt track 6:
ID 24→31). Số track fragment cũng gần như y nhau (ByteTrack: track 4, 5, 7 bị chia; ReID: track
5, 6, 7 bị chia) — nên **AssA/IDF1 tăng ở đây chủ yếu không phải nhờ ít lỗi identity hơn, mà
nhờ track được phủ đầy đủ hơn** (partially_covered_gt_tracks giảm từ 3 track xuống 1 track,
FN giảm mạnh 54 → 26). Một frame sequence không đổi đáng kể: cả hai tracker đều bám nhầm một
vật thể tĩnh (một quầy/kiosk ven đường, nằm giữa hai biển chỉ dẫn lớn) suốt gần như cùng một
đoạn — ByteTrack track 10 ở frame 17-116 (dài 42 frame), ReID track 7 ở frame 16-116 (dài 43
frame) — cùng vị trí, cùng độ dài, ReID không sửa được lỗi này chút nào. Vì ByteTrack và
BoT-SORT+ReID là hai cài đặt tracker khác nhau (không chỉ khác đúng phần ReID), chênh lệch đo
được là **so sánh hệ thống**, không phải ablation cô lập được đóng góp riêng của ReID.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng 0.649 → 0.711, FN giảm mạnh 54 → 26, còn FP tăng nhẹ 88 → 91. Detector (YOLO) chạy
với đúng cùng `conf/iou/imgsz/classes` ở cả hai run, nên chênh lệch FN/FP không đến từ việc đổi
mô hình phát hiện mà từ cách tracker giữ/ trả về box khi việc khớp giữa các frame khác nhau
(BoT-SORT+ReID khớp lại được nhiều box bị ByteTrack bỏ qua khi xe tạm mờ/khuất một phần, nên ít
FN hơn). Phần lỗi còn lại sau ReID nghiêng về **detector** nhiều hơn association: AssA đã khá
cao (0.820) và IDSW rất thấp (2/8 track), nhưng DetA vẫn chỉ 0.711 — phần lớn do đúng một vật
thể tĩnh giống-xe (kiosk ở trên) gây FP kéo dài, cộng một số box bị lỏng ở LocA 0.872 (chưa
tuyệt đối khít).

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Frame 91-116 (rõ nhất ở frame 91, 104, 105, 106), track ReID T7/T9: model gắn một bbox ổn định
suốt hơn 40 frame lên đúng vị trí một quầy/kiosk ven đường (vật thể tĩnh, không di chuyển suốt
đoạn đó) và tính nó là "vehicle". Nhãn của tôi hoàn toàn không có track nào ở vị trí này — đúng
theo phạm vi gán ("không gán... xe trong ảnh quảng cáo") vì đây rõ ràng không phải xe bốn bánh.
Đây chính là kiểu lỗi notebook cảnh báo trước ("track đứng im" là dấu hiệu model bắt nhầm vật
thể tĩnh) — ReID sai, tôi đúng.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Ở frame 190 (frame cuối cùng của clip) và frame 138, track của tôi (gt track 3) so với ReID
(pred track 3) chỉ còn IoU 0.527-0.579 — thấp hơn hẳn so với các frame giữa clip. Đây là dấu
hiệu đáng xem lại: có thể ở đúng những frame cuối, khi xe đang ra khỏi khung, bbox của tôi
không được cập nhật khít bằng các đoạn giữa (ít keyframe hơn ở biên clip). Tôi không sửa nhãn
chỉ vì model lệch, nhưng cần tự tua lại đúng frame 138-190 track 3 để xác nhận bbox của mình có
ôm sát phần nhìn thấy được của xe hay không, theo đúng luật bbox trong `GUIDELINE_MINI.md`.

## 6. Nếu phải gán thêm 10 clip nữa

Dựa trên các lỗi thật tìm được ở trên (không phải liệt kê chung), tôi sẽ sửa `GUIDELINE_MINI.md`
theo ba điểm cụ thể:

1. **Thêm luật rõ cho vật thể tĩnh giống xe** (quầy/kiosk, bốt điện...): ghi thành luật cứng
   "nếu bbox không nhúc nhích trong >10 frame liên tiếp mà không có gì che, dừng lại kiểm tra
   xem đó có phải xe thật hay vật cố định trước khi giữ track" — vì cả ByteTrack và ReID đều mắc
   đúng lỗi này trên cùng một vật thể.
2. **Định lượng lại ngưỡng bắt đầu/kết thúc track**: nhãn của tôi lệch gold vài đến 12 frame ở
   lúc xe vào/ra khung (track 4, 5) — cần ghi cụ thể số frame đệm cho phép và ví dụ hình ảnh xe
   "đủ rõ" để bắt đầu track, thay vì chỉ nói chung là "xác định được là xe bốn bánh".
3. **Thêm keyframe dày hơn ở đoạn xe đổi hướng/rẽ**: track 5 tại frame 85-90 tụt IoU xuống 0.51
   vì bbox trôi đúng lúc xe rẽ — nên quy định cụ thể: mỗi khi xe đổi hướng rõ, thêm ít nhất 1
   keyframe mỗi 2-3 frame trong đoạn đó.

Về quy trình làm việc, tôi sẽ chạy `tools/lock_pre_gold.py` **ngay sau khi kiểm chéo xong**,
trước khi mở bất kỳ thứ gì liên quan đến gold hoặc model — file nộp lần này thiếu đúng bước đó
nên không có bằng chứng pre-gold để đối chiếu trước/sau rework.

## 7. Tệp đã nộp

- [X] `annotations/clip_01/gt.txt`
- [X] `annotations/clip_02/gt.txt`
- [X] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [X] `GUIDELINE_MINI.md` đã điền
- [X] `outputs/eval_vs_gold.json`
- [X] `outputs/model_bytetrack_clip_01.txt`
- [X] `outputs/model_reid_clip_01.txt`
- [X] `outputs/model_run_config.json`
- [X] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [X] `reports/REPORT.md` (file này)