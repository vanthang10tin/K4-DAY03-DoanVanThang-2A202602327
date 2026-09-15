# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Đoàn Văn Thắng (MSSV: 2A202602327 - T037)`  
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

Bổ sung của nhóm (nếu có): 
- **Xe đỗ cố định bên lề đường**: VẪN GÁN là `vehicle` và duy trì một `track_id` duy nhất xuyên suốt thời gian xe nằm trong khung hình (ví dụ xe màu trắng đỗ lề trái ở `clip_01` từ frame 1 đến 190).
- **Xe máy và người đi bộ**: Tuyệt đối KHÔNG gán, kể cả khi họ đi song song hoặc cắt ngang xe ô tô. Gán thêm sẽ bị tính là False Positive (FP) và trừ điểm nặng vào MOTA.
- **Phương tiện thô sơ khác**: Xe ba gác, xe xích lô hoặc xe đẩy hàng không thuộc danh mục xe 4 bánh, không gán.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Quán tính, vận tốc và hướng di chuyển của xe trong 2 giây còn xác định chắc chắn được; duy trì đúng danh tính thực tế của xe mà không làm gãy trajectory. |
| Xe bị che lâu hơn ngưỡng trên | Mở track mới (ID mới) | Sau 2 giây bị che hoàn toàn, xe có thể đã rẽ, dừng hoặc bị hoán đổi vị trí với xe khác; giữ ID cũ có rủi ro cao gây ID switch sai. |
| Xe rời khung hình rồi quay lại | Mặc định: **track mới** (ID mới) | Theo chuẩn MOTChallenge, khi vật thể hoàn toàn ra khỏi vùng nhìn thấy của camera thì vòng đời track đó kết thúc; việc xe quay lại tính là một quan sát độc lập mới. |
| Hai xe cắt nhau / chồng lên nhau | Giữ nguyên ID của từng xe dựa trên quỹ đạo và đặc điểm nhận dạng (màu xe, hướng đi, độ lớn) | Hai xe chỉ chồng lấp hình học trên mặt phẳng ảnh (occlusion), bản chất là hai thực thể tách biệt; tuyệt đối không đổi chéo ID khi giao cắt. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa ảnh ($x=0$, $y=0$, $x=W$ hoặc $y=H$), không phỏng đoán/vẽ phần thân xe nằm ngoài khung hình. |
| Xe bị xe khác che một phần | Bbox chỉ ôm sát **phần nhìn thấy được** (visible box), không vẽ bao trùm phần bị che khuất. |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: kích thước tối thiểu từ ~20px mỗi chiều và phân biệt rõ kết cấu đèn/kính xe khỏi xe máy/nền. |
| Xe đang đỗ, không di chuyển | Tạo bbox ôm sát ở frame 1, kiểm tra định kỳ để bbox không bị trôi do rung lắc camera; duy trì liên tục và KHÔNG bấm outside giữa chừng. |
| Keyframe đặt dày ở đâu | Đặt dày (3-5 frame/keyframe) khi xe rẽ hướng, đổi góc nhìn (từ làn xa về gần), tăng/giảm tốc hoặc đi qua vùng bị che; đặt thưa (15-20 frame) khi xe đi thẳng đều ở làn xa. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01` / frame 1-190 / ID 2 (ứng với Gold Track 1)
- Tình huống: Chiếc ô tô màu trắng đỗ cố định bên lề đường bên trái suốt toàn bộ 190 frame không di chuyển.
- Quyết định: Vẫn gán nhãn `vehicle`, duy trì một `track_id` duy nhất từ frame 1 đến 190, không bấm `outside`.
- Lý do: Schema tracking yêu cầu quản lý toàn bộ phương tiện 4 bánh hiện diện trong video. Xe đỗ vẫn là vehicle; nếu không gán sẽ bị tính 190 lỗi False Negative (FN), còn nếu bấm outside giữa chừng sẽ làm phân mảnh track.

### Ca 2
- Clip / frame / ID: `clip_01` / frame 55-150 / ID 4 (ứng với Gold Track 4)
- Tình huống: Xe rẽ từ nhánh đường phía trên bên phải (rìa ảnh $x \approx 936, y \approx 223$), ban đầu chỉ lộ một góc nhỏ đầu xe và bị cắt bởi cạnh phải khung hình.
- Quyết định: Bắt đầu track từ frame 55 ngay khi nhận diện được đầu xe ô tô, bbox bám sát mép phải ($x_2 = 960$). Đến frame 150 khi toàn bộ thân xe đi khỏi mép trái thì bấm ngay phím `O` (outside).
- Lý do: Tuân thủ luật chạm mép không đoán phần ngoài ảnh; bấm outside dứt khoát tại frame 150 để tránh việc CVAT tiếp tục giữ bbox lơ lửng ở khoảng trống lề đường gây lỗi bbox treo.

### Ca 3
- Clip / frame / ID: `clip_01` / frame 136-169 / ID 8 (ứng với Gold Track 8)
- Tình huống: Chiếc xe xuất hiện từ mép đáy màn hình ($y \approx 508-540$) chạy chéo lên góc phải. Ở frame 136-138, xe mới chỉ nhú một phần nhỏ nóc xe ở góc dưới cùng.
- Quyết định: Ban đầu nhóm bắt đầu gán từ frame 139 khi xe đã nhô lên rõ. Sau khi đối chiếu với Gold và Model (Model ID 54 bắt được từ frame 136), nhóm đã điều chỉnh luật: bắt đầu track ngay từ frame 136 khi phần mui xe có diện tích $> 20$px.
- Lý do: Chờ xe vào sâu mới gán sẽ làm mất 3 frame đầu tiên của vòng đời xe, gây ra FN và làm giảm độ bao phủ track (partially covered track).

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Quy tắc bắt đầu track ở rìa ảnh (Entry threshold)**: Trước đây chỉ ghi chung chung "bắt đầu khi xác định được là xe", dẫn đến việc ở các frame xe mới chớm vào từ mép dưới (frame 136 của Track 8) hoặc làn xa (frame 79 của Track 5) bị bỏ sót vài frame đầu. Đã sửa lại: *Bắt đầu track ngay khi bất kỳ bộ phận đặc trưng nào của xe (đèn, nóc, kính) xuất hiện ở rìa với kích thước $\ge 20$px*.
- **Quy tắc kiểm tra mật độ keyframe khi xe rẽ/đổi góc nhìn**: Khi xe từ làn xa tiến lại gần camera (như Track 4 và Track 6), kích thước bbox tăng theo cấp số nhân và góc phối cảnh thay đổi. Nếu để keyframe thưa (15-20 frame), bbox nội suy tuyến tính sẽ bị lệch tâm (IoU tụt dưới 0.60). Đã bổ sung quy tắc: *Bắt buộc đặt keyframe dày 3-5 frame ở mọi khúc cua hoặc khi xe thay đổi kích thước nhanh chóng*.
- **Quy tắc dứt điểm track (Exit threshold)**: Kiểm tra kỹ frame xe hoàn toàn khuất khỏi màn hình, bấm `O` (outside) ngay tại frame kế tiếp, tránh để bbox đứng im ở rìa tạo cảnh báo bbox treo.
