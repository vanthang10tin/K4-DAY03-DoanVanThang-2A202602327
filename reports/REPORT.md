# Báo cáo Ngày 3 - Tracking Annotation

Họ tên: Đoàn Văn Thắng - Lớp T037
Mã sinh viên: 2A202602327
Ngày thực hiện: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ sử dụng | CVAT tại app.cvat.ai, chế độ vẽ Track, định dạng xuất MOT 1.1 |
| Thời gian hoàn thành clip 02 | 25 phút |
| Thời gian hoàn thành clip 01 | 85 phút |
| Số lượng track đã vẽ trong clip 01 | 8 track |
| Mật độ keyframe trung bình | Khoảng 8 đến 9 keyframe cho mỗi track |

Ba tình huống khó khăn nhất khi thực hiện clip 01 và giải pháp xử lý:

1. Chiếc xe ô tô màu trắng đỗ cố định bên lề đường phía bên trái từ frame 1 đến hết frame 190. Đây là vật thể đứng yên suốt toàn bộ clip, nếu không chú ý thì rất dễ bỏ qua. Em đã vẽ một bbox ôm sát phần nhìn thấy của xe ngay từ frame 1 và duy trì một track ID duy nhất cho đến frame 190, không bấm phím outside giữa chừng để tránh làm đứt track.
2. Chiếc xe xuất hiện từ mép đáy màn hình chạy chéo lên ở frame 136 đến frame 169. Ở những frame đầu tiên, xe chỉ mới nhú một góc nhỏ của nóc xe sát mép dưới khung hình nên rất dễ bị bỏ sót. Em đã xác định xe ngay từ frame 136 khi phần mui xe có diện tích nhìn thấy rõ ràng, cho cạnh đáy bbox chạm sát mép ảnh và bám sát hành trình của xe.
3. Xe từ ngã ba phía xa rẽ vào đường chính và tăng tốc tiến lại gần camera ở Track 4 và Track 6. Khi xe chuyển hướng và lại gần, kích thước xe tăng rất nhanh đồng thời góc phối cảnh thay đổi liên tục. Nếu đặt keyframe quá thưa thì đường nội suy tự động sẽ bị lệch tâm và trôi ra ngoài thân xe. Em đã xử lý bằng cách tăng mật độ keyframe dày đặc từ 3 đến 5 frame một điểm ở toàn bộ khúc cua để bbox luôn ôm khít xe.

## 2. Quá trình tự kiểm tra và kiểm chéo

Kết quả sau ba lượt rà soát video:

- Lượt 1 kiểm tra tính liên tục của ID: Phát video ở tốc độ nhanh để mắt tập trung theo dõi các con số ID trên từng xe. Toàn bộ 8 xe đều giữ nguyên một ID duy nhất từ lúc xuất hiện đến lúc rời đi, không có hiện tượng nhảy số hoặc đổi chéo ID giữa các xe.
- Lượt 2 kiểm tra frame xuất hiện và biến mất: Tua chậm từng frame ở điểm đầu và điểm cuối của từng track. Em đã bấm phím O để outside ngay khi xe hoàn toàn ra khỏi khung hình ở lề trái, tránh để lại bbox treo ở khoảng trống. Đồng thời phát hiện và bù lại một số frame đầu của các xe mới chớm vào khung hình.
- Lượt 3 kiểm tra độ khít của bbox ở các đoạn dài: Nhảy vào các frame nằm chính giữa hai keyframe cách nhau xa nhất. Em phát hiện một vài frame giữa ở Track 5 và Track 7 bị trôi nhẹ do xe đổi hướng, sau đó đã đặt bổ sung các keyframe trung gian để kéo bbox về đúng vị trí thân xe nhìn thấy.

Tình huống phân vân nhất và quy tắc bổ sung vào guideline:

- Tình huống phân vân nhất: Chiếc xe ở Track 8 chớm xuất hiện từ mép dưới khung hình ở frame 136 đến 138. Ban đầu em ngần ngại chưa gán vì diện tích nhìn thấy còn nhỏ, sang frame 139 xe nhô cao mới vẽ keyframe đầu tiên.
- Bổ sung vào guideline: Cần quy định ngưỡng vào khung hình rõ ràng theo kích thước. Bắt đầu vẽ track ngay khi chi tiết nhận diện của xe xuất hiện ở mép ảnh với kích thước từ 20 pixel trở lên, không chờ xe đi sâu vào khung hình mới vẽ.

## 3. Kết quả đối chiếu với bộ nhãn gold trước và sau sửa lỗi

Bảng chỉ số đánh giá:

| Lần đánh giá | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Trước khi sửa | 0.753 | 0.727 | 0.784 | 0.846 | 0.938 | 0.880 | 0.822 | 17 | 52 | 0 |
| Sau khi sửa | 0.741 | 0.717 | 0.771 | 0.818 | 0.954 | 0.909 | 0.789 | 17 | 35 | 0 |

Kiểm tra điều kiện qua cổng:
- Chỉ số IDF1 đạt 0.954, vượt yêu cầu tối thiểu 0.80
- Chỉ số MOTA đạt 0.909, vượt yêu cầu tối thiểu 0.75
- Chỉ số MOTP đạt 0.789, vượt yêu cầu tối thiểu 0.70
- Kết luận: Đạt chuẩn cổng yêu cầu của bài thực hành.

Chi tiết các lỗi đã khắc phục dựa theo báo cáo đối chiếu:

| Loại lỗi | Vị trí frame | Mã ID | Thao tác chỉnh sửa |
| --- | --- | --- | --- |
| Bỏ sót frame đầu | Frame 86 đến 93 | ID 5 | Mở rộng thêm 8 frame ở đầu track từ frame 86 thay vì frame 94, giúp giảm số lỗi bỏ sót từ 52 xuống 35 |
| Bỏ sót frame đầu | Frame 55 | ID 4 | Bổ sung thêm frame 55 khi đầu xe vừa chớm rẽ vào từ mép phải khung hình |
| Biên kết thúc track | Frame 11 | ID 1 | Điều chỉnh cạnh trái chạm sát mép ảnh ở frame 11 và bấm outside đúng frame 12 khi xe vừa ra khỏi ảnh |
| Bbox bị trôi | Frame 88 đến 102 | ID 5 | Bổ sung thêm 3 keyframe tại các frame 90, 96 và 100 để sửa độ lệch tâm khi xe di chuyển xa |
| Bbox bị trôi | Frame 117 đến 121 | ID 7 | Nắn lại kích thước bbox bám sát thân xe khi xe lại gần camera |

## 4. Kết quả chạy mô hình và so sánh ba chiều

Cấu hình chạy: mô hình yolo26n.pt kết hợp thuật toán bytetrack.yaml, ngưỡng tin cậy 0.25, kích thước ảnh 960, các lớp xe cơ giới gồm car, bus, truck.

Bảng kết quả so sánh ba chiều:

| Phép so sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.741 | 0.717 | 0.771 | 0.818 | 0.954 | 0.909 | 0.789 | 17 | 35 | 0 |
| model vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| model vs bạn | 0.641 | 0.589 | 0.703 | 0.784 | 0.893 | 0.784 | 0.734 | 85 | 33 | 2 |

## 5. Trả lời năm câu hỏi phân tích

### Câu hỏi 1: MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói lên điều gì, và vì sao MOTA không phạt nặng lỗi ID?

Kết quả của em có MOTA đạt 0.909, thấp hơn chỉ số IDF1 đạt 0.954.

Trong trường hợp MOTA cao nhưng IDF1 lại thấp, điều đó cho thấy hệ thống phát hiện vị trí vật thể trên từng frame đơn lẻ rất tốt, số lượng bbox tìm được khá đầy đủ và ít bị bỏ sót. Tuy nhiên, khâu liên kết danh tính qua các frame lại bị sai hỏng nghiêm trọng, ví dụ như xảy ra nhiều lần nhảy đổi ID hoặc một chiếc xe bị chia nhỏ thành nhiều track khác nhau.

Lý do MOTA không phạt nặng lỗi ID là vì trong công thức của MOTA, mỗi lần xảy ra hoán đổi ID chỉ bị cộng thêm đúng một đơn vị lỗi tại duy nhất frame xảy ra sự cố. Sau frame đó, nếu xe vẫn tiếp tục chạy với ID mới thì MOTA xem các bbox đó vẫn là phát hiện đúng vị trí và không tiếp tục trừ điểm. Vì vậy trong một clip dài có hàng trăm frame, vài lần đổi ID chỉ làm giảm MOTA một tỷ lệ rất nhỏ. 

Trái lại, IDF1 tính toán sự nhất quán danh tính trên toàn bộ chiều dài thời gian của track. Khi một chiếc xe bị đổi ID ở giữa chặng đường, chỉ có một nửa chặng đường đầu được tính là đúng danh tính, còn toàn bộ nửa chặng đường sau sẽ bị tính là gán sai danh tính cho từng frame kéo dài về sau. Do đó IDF1 phản ánh độ ổn định danh tính chặt chẽ hơn nhiều so với MOTA.

### Câu hỏi 2: DetA và AssA của model lệch nhau bao nhiêu? Chỉ số nào kéo điểm HOTA xuống?

Điểm DetA của mô hình đạt 0.649, trong khi điểm AssA đạt 0.776. Độ chênh lệch giữa hai chỉ số là 0.127, tương đương 12.7%.

Yếu tố kéo điểm tổng HOTA của mô hình xuống chính là DetA với mức 0.649. Điều này cho thấy điểm yếu lớn nhất của mô hình nằm ở khâu phát hiện vật thể của YOLO chứ không phải ở khâu liên kết của ByteTrack. 

Cụ thể, mô hình sinh ra lượng lớn bbox thừa với 88 lỗi False Positive do nhận diện nhầm các vật thể tĩnh bên đường và các nhiễu ở rìa ảnh. Đồng thời mô hình cũng bỏ sót 54 frame do xe ở quá xa hoặc bị che khuất một phần. Ngược lại, thuật toán ByteTrack duy trì liên kết track tương đối tốt, đạt AssA 0.776 và chỉ để xảy ra đúng 2 lần hoán đổi ID trong cả clip.

### Câu hỏi 3: Chỉ ra một vị trí bạn đúng và model sai, và một vị trí model đúng và bạn sai

Vị trí em đúng và mô hình sai:
- Địa điểm: Từ frame 17 đến frame 116, khu vực bên lề đường phía trên bên phải.
- Hiện tượng: Mô hình nhận diện nhầm một kiến trúc ki-ốt tĩnh bên đường thành xe ô tô và tạo ra track ID 10 tồn tại suốt 42 frame. Em quan sát thấy vật thể này đứng yên hoàn toàn và là công trình ven đường nên không gán nhãn, bộ nhãn gold cũng không có track nào tại đây. Ngoài ra tại frame 59 khi xe rẽ, mô hình bị nhảy ID từ 14 sang 15, trong khi em giữ được một ID duy nhất xuyên suốt hành trình của xe.

Vị trí mô hình đúng và em sai:
- Địa điểm: Từ frame 136 đến frame 138, chiếc xe xuất hiện ở mép đáy khung hình.
- Hiện tượng: Em bắt đầu vẽ track từ frame 139 khi xe đã nhô lên rõ ràng, bỏ lỡ 3 frame đầu tiên. Trong khi đó mô hình đã bắt được phần mui xe ngay từ frame 136 với ID 54. Khi đối chiếu với bộ nhãn gold, track này thực sự bắt đầu từ đúng frame 136.

### Câu hỏi 4: Trong ba loại bất đồng giữa bạn và model, loại nào chiếm nhiều nhất và điều đó nói lên điều gì về clip này?

Thống kê ba loại bất đồng:
- Bbox chỉ có ở mô hình: 85 trường hợp, chiếm đa số tuyệt đối.
- Bbox chỉ có ở bản gán tay: 33 trường hợp.
- Hai bên cùng có bbox nhưng lệch ID: 0 trường hợp ở các frame chung.

Loại bất đồng chiếm nhiều nhất là Bbox chỉ có ở mô hình với 85 trường hợp.

Đặc điểm của clip phản ánh qua kết quả này:
- Không gian ven đường có nhiều kiến trúc tĩnh dễ gây hiểu lầm cho mô hình phát hiện được huấn luyện tổng quát, dẫn đến việc mô hình nhận nhầm mái che và biển hiệu thành xe ô tô.
- Mép ảnh bên trái có nhiều phương tiện đi ra ngoài vùng nhìn thấy, khiến mô hình liên tục tạo ra các track ngắn vụn vài frame ở sát viền ảnh trước khi mất dấu.
- Mắt người có ưu thế vượt trội trong việc phán đoán ngữ cảnh chuỗi thời gian, giúp nhận diện chính xác các xe ở xa và xe bị che khuất mà mô hình bị mất độ tự tin.

### Câu hỏi 5: Nếu phải gán thêm 10 clip nữa, bạn sẽ sửa gì trong guideline và thay đổi gì trong quy trình làm việc?

Nội dung sửa đổi trong guideline:
- Quy định định lượng rõ ràng cho việc bắt đầu track: Bắt đầu vẽ ngay khi nhìn thấy bất kỳ bộ phận nào của xe ở mép ảnh với kích thước từ 20 pixel trở lên, không chờ xe vào sâu.
- Bắt buộc kiểm tra xe đỗ tĩnh: Nhấn mạnh quy định phải rà soát kỹ các phương tiện bốn bánh đỗ bên lề đường ngay từ frame đầu tiên và duy trì track liên tục đến hết video.
- Quy định khoảng cách đặt keyframe: Bắt buộc đặt keyframe dày từ 3 đến 5 frame ở các đoạn xe chuyển hướng hoặc tăng giảm tốc, và đặt từ 15 đến 20 frame ở các đoạn xe đi thẳng đều.

Nội dung thay đổi trong quy trình làm việc:
- Áp dụng nguyên tắc chốt biên trước: Khi gán một chiếc xe, đầu tiên tua nhanh đến frame xe chuẩn bị ra khỏi ảnh để bấm outside dứt điểm, sau đó mới quay lại giữa để đặt các keyframe chỉnh sửa. Cách làm này giúp loại bỏ hoàn toàn lỗi bbox treo lơ lửng.
- Tua kiểm tra tốc độ chậm ở các đoạn dài: Dành lượt tua thứ ba chạy ở tốc độ 0.5 để kiểm tra kỹ độ khít của bbox ở các đoạn nội suy tự động.
- Kiểm tra mã lệnh tự động ngay sau khi xuất nhãn: Chạy script kiểm tra định dạng ngay trên máy tính để phát hiện sớm các lỗi trùng ID hoặc vượt khung hình trước khi nộp bài.

## 6. Danh mục các tệp nộp bài

- annotations/clip_01/gt.txt
- annotations/clip_02/gt.txt
- GUIDELINE_MINI.md
- outputs/eval_vs_gold.json
- outputs/eval_model_vs_gold.json
- outputs/eval_model_vs_me.json
- outputs/model_clip_01.txt
- reports/REPORT.md
