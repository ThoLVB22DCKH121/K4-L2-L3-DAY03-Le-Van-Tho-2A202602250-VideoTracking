# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Lê Văn Thọ`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (= 2 giây @ 12.5 fps, theo mặc định của lab) | Ở 12.5 fps, dưới 2 giây là khoảng thời gian một xe bị cột đèn/xe khác che ngang mà vẫn đang di chuyển liên tục theo cùng hướng — đủ ngắn để suy luận bằng mắt là cùng một xe vật lý, không cần appearance. Đây cũng là ngưỡng mà cả ByteTrack và BoT-SORT+ReID coi là "còn track được" trước khi tự tách ID. |
| Xe bị che lâu hơn ngưỡng trên | mở **track mới**; nếu chắc chắn là cùng xe khi tua lại toàn clip, ghi chú "liên kết với track cũ" trong `reports/REPORT.md` nhưng **vẫn để hai ID khác nhau** trong file `gt.txt` | Sau ~25 frame, dự đoán vị trí (nếu ngoại suy theo tốc độ cũ) đã trôi quá xa để chắc chắn — rủi ro gán nhầm ID cho một xe khác đứng cùng chỗ. Bằng chứng thật: cả ByteTrack (track 4, 5, 7) và ReID (track 5, 6, 7) đều tự tách ID đúng ở các đoạn khuất dài (frame 59, 94, 87, 113) — máy cũng "bỏ cuộc" giữ ID ở ngưỡng tương tự, nên annotator giữ ID quá lâu qua occlusion dài sẽ khó tái lập và khó chấm điểm AssA. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** — chỉ tính là "rời khung" khi bbox biến mất hoàn toàn (diện tích = 0), **không tính** lúc xe chỉ bị cắt bởi rìa ảnh (bbox vẫn chạm rìa, theo luật Mục 3) | Một khi xe ra hẳn khỏi khung, không còn vị trí/motion liên tục để annotator khẳng định chắc 100% là đúng xe cũ quay lại (có thể là một xe khác giống màu/kiểu). Tách track mới an toàn hơn, đúng theo mặc định `GUIDE.md`. Cần nói rõ ranh giới "rời khung" vs "bị cắt rìa" vì đây là chỗ tôi thấy track 4 và 5 lệch gold 3-12 frame (Mục 4, Ca 1 & 2) — nhiều khả năng do nhầm giữa hai trường hợp này. |
| Hai xe cắt nhau / chồng lên nhau | **Không đổi ID cho nhau.** Giữ ID mỗi xe theo hướng và tốc độ di chuyển ngay **trước** lúc chồng lấp (không quyết định theo bbox tại đúng frame che khuất). Trong lúc chồng lấp, mỗi xe vẫn có bbox riêng ôm phần nhìn thấy được (theo Mục 3), và tăng mật độ keyframe lên **mỗi 1-2 frame** trong suốt đoạn cắt nhau | Đây là đúng chỗ cả hai tracker sai nhiều nhất: AssA chỉ 0.776 (ByteTrack) và 0.820 (ReID), và các lần ID switch (frame 59, 94, 87, 113) đều rơi ngay tại/kề đoạn hai xe ở gần nhau. Máy chỉ có motion + IoU (+ appearance) tại từng frame nên dễ hoán đổi ID khi hai box chồng khít; người gán có lợi thế xem được cả đoạn video trước/sau, nên phải dùng lợi thế đó (nhìn hướng đi trước khi che) thay vì đoán theo bbox tại đúng khung hình khuất — nếu không, annotator sẽ mắc đúng lỗi giống model. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên mà **hình dạng bốn bánh + khung xe** phân biệt được bằng mắt thường khi zoom ảnh gốc — không lùi lại thêm để "chắc ăn", không đợi xe lớn hẳn. Ngưỡng tối thiểu: ước lượng bề ngang bbox ≥ khoảng 15-20px ở đúng frame 190×540 của clip này |
| Xe đang đỗ, không di chuyển | vẫn track suốt thời gian xe còn trong khung (không phải "không track vì không di chuyển"); vẫn cần **ít nhất 1 keyframe mỗi 30-40 frame** để xác nhận bbox chưa trôi do camera rung/nén ảnh, dù xe không đổi vị trí |
| Keyframe đặt dày ở đâu | mặc định 1 keyframe mỗi ~10 frame (khớp trung bình bạn đang dùng); **tăng lên 1 keyframe mỗi 2-3 frame** trong bất kỳ đoạn xe đổi hướng, rẽ, hoặc bị che/hiện lại — và tự kiểm bằng cách tua qua đúng đoạn đó xem bbox có "giật" không |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

### Ca 1
- Clip / frame / ID: `clip_01 / frame 67-78 / track 5`
- Tình huống: `Bbox của tôi tồn tại ở các frame 67-78 (12 frame) nhưng track tham chiếu (gold) cho track 5 chưa xuất hiện ở đó — tức là tôi bắt đầu/giữ track sớm hơn gold.`
- Quyết định: `bắt đầu track ngay khi thấy phần đầu xe xuất hiện ở rìa khung, kể cả còn bị che một phần`
- Lý do: `tôi cho rằng nếu AI có thể nhận diện được một vật thể thì tôi sẽ bắt đầu track từ lúc đó, kể cả vật thể chưa fully-visible.`

### Ca 2
- Clip / frame / ID: `clip_01 / frame 149-151 và 51-53 / track 4`
- Tình huống: `Bbox của tôi vẫn còn ở frame 149-151 sau khi track tham chiếu 4 đã rời khung (kết thúc muộn), và cũng có bbox ở frame 51-53 trước khi track tham chiếu 4 xuất hiện (bắt đầu sớm) — cùng một track bị lệch ở cả hai đầu.`
- Quyết định: `chọn bấm Outside muộn hơn một chút để chắc chắn xe đã hoàn toàn ra khỏi khung`
- Lý do: `tôi nghĩ rằng khi xe đã rời khỏi khung hình thì tôi nên bấm outside để kết thúc track`

### Ca 3
- Clip / frame / ID: `clip_01 / frame 85-90 / track 5`
- Tình huống: `Trong đoạn xe đang rẽ/đổi hướng, bbox của tôi chỉ còn IoU 0.51-0.60 so với track tham chiếu (thấp nhất trong toàn clip) — bbox không theo kịp hình dạng xe khi xe xoay.`
- Quyết định: `không thêm keyframe phụ trong đoạn xe rẽ vì nghĩ nội suy tuyến tính của CVAT là đủ`
- Lý do: `tôi không nghĩ rằng với đoạn xe đang rẽ thì việc thêm keyframe là thực sự cần thiết, bởi vì tôi nghĩ rằng nội suy tuyến tính của CVAT là đủ để theo kịp hình dạng xe khi xe xoay.`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Luật cho vật thể tĩnh giống xe (quầy/kiosk ven đường): cả ByteTrack control và BoT-SORT+ReID đều bám nhầm đúng một vật thể tĩnh thành "vehicle" suốt hơn 40 frame (xem outputs/eval_bytetrack_vs_gold.json, outputs/eval_reid_vs_gold.json) — nên guideline nên nói rõ hơn cách phân biệt xe đỗ (vẫn gán) với vật cố định không phải xe (không gán), vì cả hai đều "không di chuyển".`
- `Ngưỡng frame cụ thể cho lúc xe vào/ra khung: Mục 3 ở trên đang để trống "ngưỡng nhóm chọn", trong khi dữ liệu cho thấy đây chính là chỗ annotation lệch gold nhiều nhất (Ca 1, Ca 2). Lần sau nên chốt một số frame đệm cụ thể ngay từ đầu.`