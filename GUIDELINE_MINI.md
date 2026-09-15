# Mini annotation guideline - Ngày 3 tracking

Người thực hiện: Đoàn Văn Thắng - MSSV 2A202602327 - Lớp T037
Clip áp dụng: clip_01, clip_02

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất là vehicle gồm mọi xe bốn bánh di chuyển hoặc dừng đỗ trên đường.

| Gán | Không gán |
| --- | --- |
| Xe con, taxi, xe bán tải | Người đi bộ |
| Xe van, minivan | Xe đạp |
| Xe buýt, minibus | Xe máy, xe mô tô |
| Xe tải, xe đầu kéo | Xe trong biển quảng cáo, xe phản chiếu qua gương |

Quy định bổ sung của nhóm:
- Xe đỗ cố định bên lề đường vẫn phải gán nhãn vehicle và giữ một track ID duy nhất xuyên suốt các frame mà xe xuất hiện trong khung hình. Ví dụ chiếc xe màu trắng đỗ lề trái ở clip 01 từ frame 1 đến frame 190.
- Xe máy và người đi bộ tuyệt đối không gán, kể cả khi họ đi sát cạnh ô tô. Nếu gán thêm sẽ bị tính lỗi False Positive làm giảm điểm MOTA.
- Các phương tiện thô sơ như xe đẩy hàng, xe xích lô hoặc xe ba gác không thuộc nhóm xe bốn bánh nên không gán.

## 2. Luật ID

| Tình huống | Luật của nhóm | Lý do |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu thời gian bị che dưới 25 frame, tương đương 2 giây ở tốc độ 12.5 fps | Trong khoảng 2 giây, hướng di chuyển và vận tốc của xe vẫn có thể suy đoán chính xác, giữ ID cũ giúp đường đi của xe liền mạch và đúng thực tế |
| Xe bị che lâu hơn 25 frame | Kết thúc track cũ và mở track ID mới khi xe xuất hiện lại | Sau 2 giây bị che hoàn toàn, xe có thể đã chuyển hướng, dừng lại hoặc đổi làn, việc cố giữ ID cũ rất dễ gây lỗi ID switch |
| Xe rời khung hình rồi quay lại | Mở track ID mới | Khi xe đã đi ra ngoài góc nhìn của camera thì chu kỳ theo dõi của track đó coi như kết thúc, việc xe vào lại được tính là đối tượng mới |
| Hai xe cắt nhau hoặc chồng lấp lên nhau | Giữ nguyên ID của từng xe dựa theo màu sắc xe, kích thước và hướng chuyển động | Hai xe chỉ che khuất nhau trên mặt phẳng camera chứ không gộp làm một, cần quan sát liên tục trước và sau giao cắt để không gán nhầm ID của nhau |

## 3. Luật vẽ bbox

| Tình huống | Quy định thao tác |
| --- | --- |
| Xe bị cắt bởi mép ảnh | Bbox chạm đúng mép khung hình, tuyệt đối không phỏng đoán phần thân xe nằm ngoài ảnh |
| Xe bị xe khác che một phần | Bbox chỉ bao quanh phần nhìn thấy của xe, không vẽ trùm lên phần bị che |
| Xe mới xuất hiện ở mép ảnh | Bắt đầu track ngay từ frame đầu tiên nhìn rõ chi tiết đặc trưng như mui xe, đèn hoặc kính xe với kích thước tối thiểu từ 20 pixel mỗi cạnh |
| Xe đang đỗ không di chuyển | Vẽ bbox ôm sát xe ở frame đầu tiên, kiểm tra các frame sau để bbox không bị lệch, giữ nguyên ID và không bấm outside giữa chừng |
| Mật độ đặt keyframe | Đặt dày từ 3 đến 5 frame một keyframe khi xe rẽ cua, tăng giảm tốc hoặc đổi góc nhìn; đặt thưa từ 15 đến 20 frame khi xe chạy thẳng đều ở làn xa |

## 4. Ba ca mơ hồ cụ thể trong thực tế

### Ca 1
- Vị trí: clip 01, frame 1 đến 190, ID 2, tương ứng Gold Track 1
- Tình huống: Chiếc xe con màu trắng đỗ cố định bên lề đường phía bên trái trong suốt toàn bộ 190 frame.
- Quyết định: Vẫn gán nhãn vehicle, đặt một bbox ôm sát thân xe nhìn thấy ở frame 1 và duy trì một ID duy nhất đến hết frame 190, không bấm outside.
- Lý do: Yêu cầu của bài là quản lý toàn bộ xe bốn bánh có mặt trong khung hình. Xe đỗ vẫn là xe cơ giới, nếu bỏ qua sẽ bị phạt 190 lỗi False Negative, còn nếu bấm outside giữa chừng thì track sẽ bị đứt đoạn.

### Ca 2
- Vị trí: clip 01, frame 55 đến 150, ID 4, tương ứng Gold Track 4
- Tình huống: Chiếc xe rẽ từ ngã ba phía trên bên phải vào đường chính, ban đầu chỉ nhô một phần nhỏ đầu xe ở sát mép phải khung hình.
- Quyết định: Bắt đầu vẽ track từ frame 55 ngay khi nhận diện được đầu xe ô tô, cạnh phải bbox chạm sát mép ảnh. Đến frame 150 khi toàn bộ đuôi xe vừa trôi khỏi mép trái thì bấm ngay phím O để kết thúc track.
- Lý do: Bbox chạm mép ảnh không vẽ thừa ra ngoài. Bấm outside dứt khoát tại frame 150 giúp tránh lỗi bbox treo lơ lửng ở khoảng trống khi xe đã đi mất.

### Ca 3
- Vị trí: clip 01, frame 136 đến 169, ID 8, tương ứng Gold Track 8
- Tình huống: Xe xuất hiện từ góc đáy màn hình đi chéo lên phía trên. Ở frame 136 đến 138, xe mới chỉ nhô một phần nóc xe sát mép đáy.
- Quyết định: Ban đầu nhóm bắt đầu gán từ frame 139 khi xe đã lên rõ. Sau khi đối chiếu thấy xe đã có diện tích nhìn thấy từ frame 136, nhóm chỉnh sửa lại track bắt đầu ngay từ frame 136.
- Lý do: Chờ xe vào sâu mới vẽ sẽ làm mất 3 frame đầu tiên của xe, tạo ra lỗi False Negative và làm giảm độ bao phủ của track.

## 5. Cải tiến guideline sau khi kiểm chéo và chấm với gold

Sau khi kiểm tra với bộ nhãn gold và trao đổi với bạn cùng nhóm, nhóm đã làm rõ các điểm sau trong quy trình:

- Quy tắc bắt đầu track ở rìa ảnh: Trước đây chỉ ghi chung chung là bắt đầu khi nhận ra xe. Nhóm sửa lại rõ ràng: Bắt đầu vẽ ngay khi nhìn thấy bất kỳ bộ phận đặc trưng nào của xe như nóc, đèn hoặc kính xe xuất hiện ở mép ảnh với kích thước từ 20 pixel trở lên.
- Mật độ đặt keyframe ở đoạn xe rẽ: Khi xe từ ngã ba rẽ vào và tiến lại gần camera như Track 4 và Track 6, kích thước xe tăng nhanh và góc phối cảnh thay đổi liên tục. Nếu để keyframe thưa từ 15 đến 20 frame thì bbox tự động nội suy sẽ bị trôi khỏi thân xe. Nhóm bổ sung quy định bắt buộc đặt keyframe dày từ 3 đến 5 frame ở các đoạn xe rẽ hoặc đổi hướng di chuyển.
- Quy tắc bấm outside: Khi xe chuẩn bị rời khỏi khung hình, tua chậm từng frame để xác định đúng frame cuối cùng xe còn trong ảnh. Bấm phím O ở ngay frame kế tiếp để kết thúc track, tránh tình trạng bbox đứng yên ở rìa ảnh gây cảnh báo lỗi.
