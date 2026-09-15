# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Đoàn Văn Thắng /T037`  
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT (`app.cvat.ai`), chế độ Track, export MOT 1.1 |
| Thời gian gán `clip_02` (warm-up) | 25 phút |
| Thời gian gán `clip_01` | 85 phút |
| Số track đã vẽ trong `clip_01` | 8 track (ID 1 đến 8) |
| Số keyframe trung bình mỗi track | ~8.5 keyframe/track (trước interpolation) |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe đỗ tĩnh suốt toàn bộ video (Track 2 / Gold Track 1, frame 1-190)**: Chiếc xe màu trắng đỗ bên lề đường trái không di chuyển trong suốt 190 frame. Nếu không chú ý hoặc tưởng chỉ track xe đang chạy thì sẽ bỏ sót xe này và bị phạt 190 lỗi False Negative (FN). Xử lý: Tạo một bbox chuẩn xác ôm sát phần thân xe nhìn thấy ở frame 1, duy trì một ID duy nhất suốt 190 frame và không bấm outside giữa chừng.
2. **Xe xuất hiện từ rìa đáy màn hình đi chéo lên (Track 8 / Gold Track 8, frame 136-169)**: Xe chớm xuất hiện ở góc đáy, ban đầu chỉ lộ một phần nóc xe rất sát cạnh ảnh dưới, rất dễ bị bỏ qua 3 frame đầu nếu đợi xe nổi rõ. Xử lý: Xác định ngay khi xuất hiện mui xe có diện tích, vẽ bbox chạm đúng mép đáyvà theo dõi liên tục đến khi xe ra khỏi khung hình ở góc phải.
3. **Xe từ xa tiến lại gần đổi góc nhìn và tăng tốc (Track 4 và Track 6)**: Xe rẽ từ nhánh đường xa bên phải và chạy về phía lề trái, kích thước bbox tăng nhanh và góc phối cảnh thay đổi liên tục. Nếu đặt keyframe quá thưa (cách nhau 20-30 frame), nội suy tuyến tính của CVAT bị trôi khỏi thân xe (IoU tụt dưới 0.60). Xử lý: Tăng mật độ keyframe dày đặc (cách 4-6 frame/keyframe) tại các đoạn xe cua và đoạn xe tăng tốc ở cự ly gần để bbox luôn ôm khít.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- **Lượt 1 (Nhìn số ID)**: Phát nhanh toàn bộ video với tốc độ 1.5x để rà soát sự ổn định của con số ID trên từng xe. Xác nhận toàn bộ 8 xe đều giữ nguyên một ID duy nhất trong suốt hành trình, không có hiện tượng nhấp nháy ID, đổi số chéo hoặc nhảy ID giữa chừng (0 ID switch).
- **Lượt 2 (Frame đầu và frame cuối của từng track)**: Kiểm tra kỹ thời điểm bắt đầu và kết thúc. Phát hiện Track 1 và Track 4 cần bấm `outside` (phím `O`) dứt khoát tại frame xe rời hẳn mép ảnh trái, tránh để bbox trôi vào khoảng trống lề đường. Phát hiện Track 5 và Track 8 bị gán trễ vài frame đầu do tâm lý chờ xe vào hẳn trong khung mới vẽ.
- **Lượt 3 (Giữa mỗi đoạn dài)**: Nhảy vào các frame ở giữa hai keyframe cách nhau xa nhất để kiểm tra độ trôi của bbox. Phát hiện một số frame giữa của Track 5 (quanh frame 90-100) và Track 7 (quanh frame 117-121) bị lệch tâm do nội suy tuyến tính, đã bổ sung thêm các keyframe trung gian để kéo bbox ôm sát phần nhìn thấy được.

Ca nào gặp phân vân nhất, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

- **Ca phân vân nhất**: Ở chiếc xe xuất hiện từ mép đáy màn hình (Track 8, frame 136-169), ở frame 136-138 chỉ mới nhú một phần nóc xe rất nhỏ sát mép dưới, ban đầu phân vân chưa gán vì sợ là bóng hoặc chưa rõ xe, đến frame 139 mới đặt keyframe đầu tiên.
- **Luật còn thiếu trong guideline**: Guideline ban đầu chưa có quy định định lượng cho "ngưỡng vào khung hình" (Entry threshold). Bổ sung: *Bắt đầu track ngay khi bất kỳ phần đặc trưng nào của xe (nóc, đèn, kính) xuất hiện ở rìa ảnh với kích thước $\ge 20$px, không chờ xe vào sâu*.

## 3. Chấm với gold — trước và sau rework

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Lần chấm đầu | 0.753 | 0.727 | 0.784 | 0.846 | 0.938 | 0.880 | 0.822 | 17 | 52 | 0 |
| Sau rework | 0.741 | 0.717 | 0.771 | 0.818 | 0.954 | 0.909 | 0.789 | 17 | 35 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **ĐÃ QUA CỔNG**  
(Cụ thể: `IDF1 = 0.954 >= 0.80`, `MOTA = 0.909 >= 0.75`, `MOTP = 0.789 >= 0.70`).

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Thiếu đoạn (FN) | Frame 86-93 | ID 5 (Gold 5) | Bổ sung thêm 8 frame ở đầu track (mở rộng từ frame 94 về frame 86) khi xe vừa vào tầm nhìn ở làn xa, giúp giảm tổng số FN từ 52 xuống 35. |
| Thiếu đoạn (FN) | Frame 55 | ID 4 (Gold 4) | Bổ sung keyframe đón đầu ở frame 55 khi xe rẽ vào khung hình thay vì bắt đầu ở frame 56. |
| Biên track | Frame 11 | ID 1 (Gold 2) | Kéo dài thêm 1 frame (frame 11) chạm mép trái và bấm outside đúng frame 12 khi xe hoàn toàn lọt khỏi ảnh. |
| Bbox trôi (IoU thấp) | Frame 88-102 | ID 5 (Gold 5) | Thêm 3 keyframe nội suy trung gian quanh frame 90, 96, 100 để khắc phục tình trạng IoU tụt xuống 0.51 - 0.58. |
| Bbox trôi (IoU thấp) | Frame 117-121 | ID 7 (Gold 6) | Điều chỉnh lại kích thước bbox ôm sát xe khi tiến gần, khắc phục độ trôi IoU ~0.55. |

## 4. Kết quả model và so sánh ba chiều

Cấu hình thực nghiệm: model `yolo26n.pt`, trackers: `bytetrack.yaml` (Motion/Kalman) & `botsort.yaml` (ReID/Appearance), conf `0.25`, imgsz `960`, classes `[2, 5, 7] (car, bus, truck)`.

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| **bạn vs gold** | 0.741 | 0.717 | 0.771 | 0.818 | 0.954 | 0.909 | 0.789 | 17 | 35 | 0 |
| **ByteTrack vs gold** | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| **ReID (BoT-SORT) vs gold** | 0.760 | 0.710 | 0.814 | 0.876 | 0.898 | 0.791 | 0.865 | 80 | 39 | 1 |
| **ReID (BoT-SORT) vs bạn** | 0.699 | 0.635 | 0.775 | 0.811 | 0.914 | 0.818 | 0.773 | 80 | 21 | 0 |

> **Nhận xét so sánh ByteTrack vs ReID (BoT-SORT)**:
> Khi bật ReID (`botsort.yaml` kết hợp visual appearance embedding), điểm liên kết danh tính **AssA tăng từ 0.776 lên 0.814**, số lần ID switch giảm từ 2 xuống 1, và IDF1 tăng từ 0.875 lên 0.898. Điều này chứng minh đặc trưng ngoại hình (ReID) hỗ trợ đắc lực trong việc duy trì danh tính xe khi quỹ đạo chuyển động bị che khuất hoặc quay đầu.

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

- Trong kết quả của em: MOTA (0.909) thấp hơn IDF1 (0.954).
- Trường hợp "MOTA cao mà IDF1 thấp": Điều này phản ánh hệ thống phát hiện vật thể từng frame rất tốt (ít FP, ít FN, bounding box bao phủ đúng vị trí xe tại từng thời điểm), nhưng lại bị sai sót nghiêm trọng ở khâu liên kết danh tính (nhiều ID switch hoặc một xe bị cắt vụn thành nhiều ID track khác nhau).
- Vì sao MOTA không phạt nặng lỗi ID:
  Công thức tính MOTA là:
  $$\text{MOTA} = 1 - \frac{\sum (\text{FN} + \text{FP} + \text{IDSW})}{\sum \text{GT}}$$
  Trong công thức này, mỗi lần xảy ra hoán đổi ID ($\text{IDSW}$), MOTA chỉ cộng thêm đúng **1 đơn vị lỗi** vào tử số tại frame xảy ra sự cố, bất kể sau đó chiếc xe bị mang ID sai trong bao nhiêu chục hay hàng trăm frame tiếp theo. Vì vậy, trong một video dài có hàng nghìn detection, vài lỗi ID switch chỉ làm giảm MOTA một lượng không đáng kể (MOTA vẫn có thể đạt $> 0.95$).
  Ngược lại, IDF1 đo lường sự nhất quán danh tính trên toàn bộ quỹ đạo thời gian:
  $$\text{IDF1} = \frac{2 \cdot \text{IDTP}}{2 \cdot \text{IDTP} + \text{IDFP} + \text{IDFN}}$$
  Khi một track bị chia đôi ID, chỉ có nửa quãng đời đầu được ghép đúng với ID chuẩn ($\text{IDTP}$), nửa còn lại sẽ bị phạt toàn bộ vào $\text{IDFP}$ và $\text{IDFN}$ cho từng frame kéo dài sau đó. Do đó, IDF1 phản ánh đúng bản chất bài toán tracking xuyên suốt thời gian hơn MOTA.

**2. DetA và AssA của model lệch nhau bao nhiêu? Cái nào kéo HOTA xuống — model không tìm ra xe, hay tìm ra rồi nhưng đánh mất ID?**

- DetA của model đạt `0.649`, trong khi AssA đạt `0.776`. Độ lệch giữa hai chỉ số là:
  $$\Delta = \text{AssA} - \text{DetA} = 0.776 - 0.649 = 0.127 \quad (12.7\%)$$
- Yếu tố kéo HOTA ($\text{HOTA} = \sqrt{\text{DetA} \times \text{AssA}} = \sqrt{0.649 \times 0.776} \approx 0.709$) xuống chính là **DetA (0.649)** — tức là khả năng phát hiện vật thể của detector (YOLO26n).
- Giải thích chi tiết:
  1. Detector tạo ra lượng False Positive rất lớn (**88 FP** so với gold). Nguyên nhân chính là do YOLO nhận diện nhầm vật thể tĩnh bên đường (như track ID 10 tại tọa độ $x \approx 491, y \approx 212$ kéo dài 42 frame từ frame 17 đến 116) và các nhiễu nhỏ ở rìa ảnh.
  2. Detector bỏ sót xe (**54 FN**), chủ yếu ở các đoạn xe ở xa kích thước nhỏ hoặc xe bị che khuất một phần.
  3. Ngược lại, thuật toán ByteTrack thực hiện liên kết track khá tốt ($\text{AssA} = 0.776$) nhờ cơ chế 2 tầng matching kết hợp Kalman Filter, chỉ để xảy ra đúng 2 lần ID switch trong toàn bộ clip. Như vậy, model không phải bị mất dấu ID, mà là khâu detector (YOLO) tìm sai và bỏ sót vật thể kéo tụt điểm tổng HOTA.

**3. Một chỗ bạn đúng và model sai (frame, ID, vì sao):**

- **Vị trí cụ thể**: Frame 17 đến 116, tọa độ $x \approx 491.2, y \approx 211.8$, kích thước $w \approx 102.2, h \approx 58.2$.
- **Model xử lý**: Tạo ra track ID 10 kéo dài tới 42 frame trong khoảng này (tồn tại từ frame 17 đến 116).
- **Em xử lý**: Không gán nhãn ở vị trí này (Gold set của giảng viên cũng hoàn toàn không có track nào ở đây).
- **Vì sao em đúng và model sai**: Đây là một ki-ốt/quầy hàng cố định ven đường. Tọa độ của vật thể này gần như đứng im hoàn toàn trong 100 frame ($x$ chỉ dao động từ 490.0 đến 498.5). YOLO26n bị đánh lừa bởi hình khối chữ nhật của mái che/quầy hàng nên nhận diện nhầm thành ô tô (False Positive tĩnh). Em quan sát bằng mắt nhận thấy đây là kiến trúc nền, không phải xe cơ giới nên không gán nhãn.
- **Về khả năng duy trì ID**: Tại frame 59, khi xe rẽ (Gold Track 4 / Em gán Track 4), em duy trì 1 ID duy nhất xuyên suốt 96 frame, trong khi model bị ID switch từ ID 14 sang ID 15 do xe quay đầu đổi góc nhìn đột ngột làm Kalman Filter dự đoán lệch.

**4. Một chỗ model đúng và bạn sai (frame, ID, vì sao):**

- **Vị trí cụ thể**: Frame 136 đến 138, chiếc xe xuất hiện từ mép đáy màn hình (Gold Track 8).
- **Em xử lý**: Track 8 của em chỉ bắt đầu đặt keyframe từ frame 139 (khi xe đã trồi lên rõ ràng ở tọa độ $y \approx 487$), bỏ sót 3 frame đầu 136, 137, 138 (gây ra 3 lỗi False Negative).
- **Model xử lý**: Model (track ID 54) phát hiện và bắt đầu track chính xác chiếc xe này ngay từ frame 136 khi phần mui xe vừa xuất hiện ở cạnh dưới ($y \approx 508.8$).
- **Đối chiếu với Gold**: Gold set đánh dấu Track 8 từ đúng frame 136 đến frame 168.
- **Vì sao model đúng và em sai**: Mắt người khi gán nhãn thủ công có xu hướng ngần ngại ở các frame xe mới chớm vào khung hình vì kích thước nhìn thấy còn nhỏ và bị cắt mép. Model với ngưỡng tự tin `conf = 0.25` quét điểm ảnh liên tục nên phát hiện ra tín hiệu xe sớm hơn và chính xác hơn em ở 3 frame này.

**5. Trong ba loại bất đồng giữa bạn và model, loại nào nhiều nhất? Nó nói gì về clip này?**

- **Thống kê 3 loại bất đồng**:
  1. Chỉ model có (Model thừa / Bạn không có): **85 frame-bbox** (chiếm đa số tuyệt đối).
  2. Chỉ bạn có (Bạn có / Model bỏ sót): **33 frame-bbox**.
  3. Cùng có bbox nhưng khác ID (lệch ID mapping): **0 trường hợp**
- **Loại nhiều nhất**: Loại 1 — **Chỉ model có (85 bbox)**.
- **Điều này nói gì về clip này**:
  1. **Clip có cảnh nền phức tạp ven đường**: Đoạn video quay giao thông đô thị có nhiều công trình phụ, ki-ốt, biển hiệu hai bên đường có đặc trưng thị giác dễ gây nhầm lẫn cho mô hình YOLO tiền huấn luyện trên tập dữ liệu tổng quát COCO (gây ra 42 bbox thừa từ vật thể tĩnh ID 10).
  2. **Biên khung hình có nhiều nhiễu**: Các mép ảnh (đặc biệt rìa trái $x=0$) có nhiều phương tiện đi ra ngoài, khiến detector liên tục sinh ra các track ngắn 1-10 frame (ID 41, 64, 69, 70) trước khi mất dấu hoàn toàn.
  3. **Ưu thế nhận thức ngữ cảnh của con người**: Loại 2 (33 bbox chỉ bạn có) rơi vào các đoạn xe ở rất xa hoặc xe bị che khuất một phần bởi các phương tiện khác — nơi mà mắt người dựa vào ngữ cảnh chuỗi thời gian để giữ track, còn mô hình AI thì bị mất tự tin và bỏ sót.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- **Sửa trong `GUIDELINE_MINI.md`**:
  1. *Định lượng rõ ngưỡng vào khung (Entry threshold)*: Bắt đầu track ngay khi bộ phận xe xuất hiện ở mép với kích thước $\ge 20$px hoặc chiếm $\ge 15\%$ thân xe, không được chờ xe đi vào giữa đường mới gán.
  2. *Quy định bắt buộc về xe đỗ tĩnh*: Thêm điều khoản nhấn mạnh mọi xe 4 bánh đỗ ven đường đều phải gán đầy đủ từ frame đầu đến frame cuối của video, không được bỏ qua.
  3. *Mật độ keyframe theo tình huống*: Quy định rõ khoảng cách đặt keyframe: tối đa 5 frame/keyframe ở các đoạn rẽ/khuất và tối đa 15 frame/keyframe ở các đoạn đi thẳng đều.
- **Đổi trong quy trình làm việc**:
  1. *Quy trình Đánh dấu Biên trước (Boundary First)*: Khi gán một xe, đầu tiên tua nhanh đến frame xe biến mất để bấm `outside` (phím `O`) ngay lập tức, sau đó mới tua ngược về giữa để đặt các keyframe nội suy. Quy trình này triệt tiêu hoàn toàn lỗi bbox treo lơ lửng.
  2. *Tua kiểm tra tốc độ chậm ở giữa*: Luôn chạy lại lượt tua thứ 3 ở tốc độ 0.5x để rà soát hiện tượng trôi bbox (drift) giữa các keyframe cách xa nhau.
  3. *Tự động hóa kiểm tra định dạng*: Luôn chạy script `check_mot_labels.py` trong terminal ngay sau khi tải file từ CVAT về để phát hiện tức thì các lỗi trùng ID hoặc bbox vượt kích thước ảnh trước khi nộp.

## 7. Tệp đã nộp

- annotations/clip_01/gt.txt
- annotations/clip_02/gt.txt
- GUIDELINE_MINI.md
- outputs/eval_vs_gold.json
- outputs/eval_model_vs_gold.json
- outputs/eval_model_vs_me.json
- outputs/model_clip_01.txt
- reports/REPORT.md
