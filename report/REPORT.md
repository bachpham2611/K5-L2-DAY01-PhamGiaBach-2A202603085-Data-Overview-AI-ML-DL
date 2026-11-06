## BÁO CÁO BÀI THỰC HÀNH NGÀY 1

## Đọc prediction YOLO11 để hiểu định dạng nhãn

Checkpoint sử dụng

- Classification: yolo11n-cls.pt

- Detection: yolo11n.pt

- Instance segmentation: yolo11n-seg.pt

Ngưỡng confidence của detection và segmentation: 0.35

Phiên bản Ultralytics ghi trong JSON: 8.4.145

Các sample trong bộ dữ liệu: traffic, kitchen, dining

Kích thước ảnh: traffic 640 × 428 px; kitchen 640 × 427 px; dining 480 × 640 px.

Lưu ý về phạm vi dữ liệu: Kết quả classification được cung cấp đầy đủ cho sample traffic. Phần JSON detection và segmentation được dán trong nội dung đầu vào cũng ghi sample_id = traffic; ảnh minh họa detection/segmentation lại thể hiện sample kitchen. Vì vậy, báo cáo phân biệt rõ số liệu JSON của traffic với nhận xét trực quan trên ảnh kitchen, không gán nhầm tọa độ hoặc confidence của ảnh này sang ảnh kia. JSON segmentation được cung cấp là một phần dữ liệu (có các record minh họa), không phải toàn bộ 49 record.

Mục tiêu: Thực hành ba dạng bài toán thị giác máy tính gồm phân loại ảnh (image classification), phát hiện vật thể (object detection) và phân đoạn từng đối tượng (instance segmentation); đọc các trường dữ liệu prediction và liên hệ với quy trình tạo ground truth, kiểm tra chất lượng (QC).

## 1. Phân loại ảnh (Image Classification)

Model: yolo11n-cls.pt

Taxonomy: ImageNet-1K

Sample phân tích: traffic — COCO image ID 210273, kích thước 640 × 428 px.

## 1.1. Kết quả Top-5

| Rank | Class ID | Class name | Confidence |
| --- | --- | --- | --- |
| 1 | 468 | cab | 0.510915 |
| 2 | 654 | minibus | 0.164284 |
| 3 | 734 | police_van | 0.085848 |
| 4 | 757 | recreational_vehicle | 0.054110 |
| 5 | 829 | streetcar | 0.048194 |

Hình 1. Kết quả Top-5 classification của sample traffic. Confidence cao nhất thuộc lớp cab.

## 1.2. Record hạng 1 mô tả điều gì?

Record hạng 1 có các trường:

- sample_id: traffic

- coco_image_id: 210273

• task: image_classification

- taxonomy_name: ImageNet-1K

- model_file: yolo11n-cls.pt

- ultralytics_version: 8.4.145

- rank: 1

- class_id: 468


- class_name: cab

• score: 0.510915

Đây là dự đoán cho nội dung tổng thể của ảnh. Model xếp lớp cab ở vị trí đầu tiên với confidence khoảng 51,09%. Bốn lớp tiếp theo lần lượt là minibus, police_van, recreational_vehicle và streetcar.

Ảnh traffic có nhiều phương tiện giao thông, vì vậy các lớp phương tiện xuất hiện trong Top-5 là phù hợp với ngữ cảnh tổng thể. Tuy nhiên, nhãn đứng đầu cab không đồng nghĩa model đã xác định chính xác mọi phương tiện trong ảnh là taxi. Classification chỉ đưa ra lớp dự đoán cho toàn ảnh, không chỉ ra vị trí của từng phương tiện.

## 1.3. Danh sách lớp đến từ đâu?

Danh sách lớp được quy định bởi taxonomy mà checkpoint đã được huấn luyện, ở đây là ImageNet-1K. Model lựa chọn các lớp trong danh sách có sẵn và trả về mã lớp (class_id), tên lớp (class_name) cùng điểm confidence (score). Model không tự tạo ra một taxonomy mới trong bước prediction.

## 1.4. Vì sao cần cả class_id, class_name và taxonomy_name?

- class_id: mã định danh lớp, thuận tiện cho việc lưu trữ, đối chiếu và xử lý bằng máy.

- class_name: tên lớp dễ đọc đối với người kiểm tra.

- taxonomy_name: cho biết hệ thống phân loại đang được sử dụng.

Cần giữ cả ba trường vì cùng một số ID có thể mang ý nghĩa khác nhau giữa các taxonomy hoặc bộ dữ liệu

khác nhau.

## 1.5. Guideline nên quy định gì khi ảnh có nhiều vật thể hoặc chủ đề?

Guideline cần xác định rõ tiêu chí chọn nhãn cho toàn ảnh: ưu tiên chủ thể chính, đối tượng chiếm vai trò trung tâm hay chủ đề tổng thể. Đồng thời cần quy định cách xử lý ảnh có nhiều chủ thể ngang nhau, ảnh không rõ chủ đề và ảnh không thuộc danh sách lớp. Nếu không có quy tắc thống nhất, các annotator có thể gán nhãn khác nhau cho cùng một ảnh.

## 1.6. Vì sao không dùng score làm ground truth?

score là confidence do model dự đoán, không phải nhãn đúng đã được con người xác minh. Model có thể đưa ra confidence cao nhưng vẫn sai. Ground truth phải được xác định dựa trên ảnh gốc, taxonomy và guideline đã thống nhất, sau đó được kiểm tra chất lượng.

## 2. Phát hiện vật thể (Object Detection)

Model: yolo11n.pt

Taxonomy: COCO-80

Ngưỡng confidence: 0.35

Thông tin tổng hợp được cung cấp: 53 đối tượng được phát hiện.

Record JSON được dán: các record mẫu có sample_id = traffic, COCO image ID 210273, kích thước 640 × 428 px.

## 2.1. Một số prediction tiêu biểu trong JSON

| # | Class | Class ID | Confidence | Bounding box [x1, | Width × Height (px) |
| --- | --- | --- | --- | --- | --- |
|   |   |   |   | y1, x2, y2] (px) |   |
| 1 | bus | 5 | 0.912558 | [93.17, 187.95, | 129.84 × 132.96 |
|   |   |   |   | 223.01, 320.91] |   |
| 2 | car | 2 | 0.863074 | [419.05, 264.24, | 78.90 × 75.61 |
|   |   |   |   | 497.95, 339.85] |   |
| 3 | bus | 5 | 0.847436 | [225.60, 132.68, | 70.48 × 87.76 |
|   |   |   |   | 296.09, 220.45] |   |
| 4 | car | 2 | 0.846923 | [179.98, 298.91, | 74.07 × 68.60 |
|   |   |   |   | 254.05, 367.51] |   |


| # | Class | Class ID | Confidence | Bounding box [x1, | Width × Height (px) |
| --- | --- | --- | --- | --- | --- |
|   |   |   |   | y1, x2, y2] (px) |   |
| 5 | car | 2 | 0.822025 | [318.02, 308.44, | 72.25 × 61.37 |
|   |   |   |   | 390.28, 369.80] |   |
| 6 | car | 2 | 0.771383 | [300.78, 252.08, | 69.55 × 68.33 |
|   |   |   |   | 370.32, 320.41] |   |
| 7 | person | 0 | 0.769676 | [0.08, 256.79, | 18.32 × 56.33 |
|   |   |   |   | 18.39, 313.12] |   |
| 8 | car | 2 | 0.759713 | [268.46, 217.40, | 58.06 × 50.72 |
|   |   |   |   | 326.52, 268.12] |   |

Các dòng trên là 8 record đầu được cung cấp, không phải toàn bộ 53 prediction. Do đó, bảng chỉ minh họa cấu trúc và một số kết quả tiêu biểu, không đại diện cho phân bố đầy đủ của tất cả lớp.

## 2.2. Bounding box có ý nghĩa gì?

Trường bbox_xyxy lưu tọa độ của hình chữ nhật bao quanh đối tượng theo thứ tự:

[x1, y1, x2, y2]

Trong đó:

- (x1, y1) là góc trên bên trái.

- (x2, y2) là góc dưới bên phải.

- Chiều rộng = x2 - x1.

- Chiều cao = y2 - y1.

Ví dụ record đầu tiên là một bus:

- bbox_xyxy = [93.17, 187.95, 223.01, 320.91]

- bbox_width = 223.01 - 93.17 = 129.84 px

- bbox_height = 320.91 - 187.95 = 132.96 px

coordinate_unit = pixel cho biết tọa độ được tính theo pixel của ảnh; bbox_format = xyxy cho biết thứ tự biểu diễn tọa độ.

## 2.3. Ngưỡng confidence ảnh hưởng thế nào?

Ngưỡng đang dùng là 0.35. Một prediction chỉ được giữ lại khi confidence đạt hoặc vượt ngưỡng này theo cách lọc của pipeline. Khi hạ ngưỡng, hệ thống thường giữ lại nhiều prediction hơn, nhưng có thể tăng số dự đoán sai, nhiễu hoặc box trùng. Khi nâng ngưỡng, số prediction có thể giảm, nhưng một số đối tượng thật có confidence thấp cũng có thể bị loại bỏ.

Trong dữ liệu được cung cấp không có bảng đếm riêng cho ngưỡng 0.20 và 0.60, vì vậy báo cáo không đưa ra con số giả định cho hai ngưỡng này.

## 2.4. Reviewer cần làm gì khi ngưỡng thay đổi?

Reviewer cần kiểm tra:

- 1. Lớp dự đoán có đúng với đối tượng trong ảnh không.

- 2. Bounding box có bao trọn phần đối tượng nhìn thấy và có sát biên không.

- 3. Có đối tượng nào bị bỏ sót hay không.

- 4. Có nhiều box trùng nhau cho cùng một đối tượng không.

- 5. Các đối tượng nhỏ, bị che khuất hoặc nằm sát mép ảnh có được xử lý nhất quán không.

## 2.5. Quy tắc vẽ box đề xuất

- Mỗi bounding box tương ứng với một đối tượng riêng biệt.

- Box bao quanh toàn bộ phần đối tượng nhìn thấy, không cố ước lượng phần bị che khuất.

- Box nên sát biên đối tượng, tránh lấy quá nhiều nền hoặc cắt mất phần nhìn thấy.


- Không gộp nhiều đối tượng riêng biệt vào một box, trừ khi guideline của dự án quy định rõ.

- Đối tượng bị cắt ở mép ảnh vẫn được xử lý theo phần thực sự xuất hiện trong khung hình và theo quy tắc của dự án.

## 2.6. Nhận xét ảnh minh họa kitchen

Ảnh minh họa detection cho thấy nhiều đối tượng trong không gian bếp: người đứng nấu ăn, bàn, bát, lò/thiết bị bếp, thìa và chậu cây. Trên hình có các confidence hiển thị như person 0.90, potted plant 0.63, dining table 0.58, oven 0.49, spoon 0.49 và một số bowl với confidence khác nhau.

Một số box/mask ở khu vực bàn và các vật dụng đặt gần nhau có thể chồng lấn. Đây là trường hợp reviewer cần quan sát ảnh gốc để xác định từng vật thể, tránh gộp nhiều vật thành một nhãn hoặc bỏ sót vật thể nhỏ. Ảnh minh họa hỗ trợ nhận xét trực quan; do JSON được dán ở trên thuộc sample traffic, không sử dụng tọa độ trong bảng 2.1 để mô tả các box trong ảnh kitchen.

## 3. Phân đoạn từng đối tượng (Instance Segmentation)

Model: yolo11n-seg.pt

Taxonomy: COCO-80

Ngưỡng confidence: 0.35

Số đối tượng segmentation được cung cấp: 49

JSON chi tiết được dán: có record mẫu thuộc sample_id = traffic, COCO image ID 210273.

## 3.1. Hai record minh họa trong JSON

| Instance ID | Class | Class ID | Confidence | Bounding box [x1, | Số điểm polygon |
| --- | --- | --- | --- | --- | --- |
|   |   |   |   | y1, x2, y2] (px) |   |
| traffic-001 | bus | 5 | 0.925745 | [95.30, 188.72, | 120 |
|   |   |   |   | 224.08, 319.96] |   |
| traffic-002 | car | 2 | 0.891092 | [418.82, 264.54, | 92 |
|   |   |   |   | 498.47, 339.52] |   |

Ví dụ traffic-001 có:

• task: instance_segmentation

- taxonomy_name: COCO-80

• model_file: yolo11n-seg.pt

• score_threshold: 0.35

- coordinate_unit: pixel

- bbox_format: xyxy

- polygon_point_count: 120

Ví dụ traffic-002 có 92 điểm polygon. Các điểm trong polygon_xy mô tả đường biên mask theo tọa độ pixel. Hai record trên chỉ là các record được hiển thị trong phần JSON đầu vào, không phải toàn bộ 49 instance.

## 3.2. Polygon cung cấp thêm thông tin gì so với bounding box?

Bounding box chỉ xác định một vùng chữ nhật bao quanh đối tượng. Polygon mô tả đường biên của đối tượng chi tiết hơn, giúp xác định vùng pixel thuộc về đối tượng và thể hiện hình dạng không vuông vức. Vì vậy, segmentation phù hợp hơn khi cần phân biệt hình dạng và diện tích hiển thị của từng vật thể.

## 3.3. Ý nghĩa của instance_id

instance_id phân biệt từng cá thể/đối tượng riêng biệt trong kết quả segmentation. Ví dụ traffic-001 là một instance thuộc lớp bus, còn traffic-002 là một instance thuộc lớp car.

instance_id khác với class_id: nhiều instance có thể cùng một class. Nó cũng không mặc nhiên là tracking ID để theo dõi một đối tượng xuyên suốt nhiều khung hình.


## 3.4. Quy tắc vẽ polygon/mask đề xuất

• Polygon bám theo đường biên phần đối tượng nhìn thấy. • Không lấy phần nền xung quanh vào mask. • Không tự suy đoán và vẽ thêm phần bị che khuất. • Các đối tượng cùng lớp nhưng tách biệt phải được gán thành các instance riêng. • Khi hai vật chạm nhau, cần xác định ranh giới giữa chúng theo ảnh gốc; nếu không đủ căn cứ thì đánh dấu để reviewer/Lab Coach quyết định.

• Cần tránh polygon tự giao cắt, thiếu điểm biên quan trọng hoặc lấn sang vật thể khác.

## 3.5. Nhận xét ảnh minh họa kitchen

Ảnh segmentation minh họa thể hiện mask màu trên các đối tượng như người, bàn ăn, bát, lò, thìa và chậu cây. Đường biên mask cho thấy segmentation cung cấp thông tin chi tiết hơn box, đặc biệt ở người đứng giữa ảnh và các vật dụng có hình dạng không đều. Khu vực mặt bàn có nhiều vật thể đặt gần nhau, một số vùng mask và nhãn hiển thị chồng lấn. Đây là tình huống cần reviewer kiểm tra kỹ ranh giới giữa bàn, bát và các vật dụng khác. Confidence trên ảnh chỉ phản

ánh mức độ tin cậy của model, không chứng minh mask đã đúng hoàn toàn.

- 4. Vòng đời dữ liệu và kiểm tra chất lượng (QC)

Quy trình tổng quát:

Ảnh thô → Guideline → Ground truth → Huấn luyện model → Prediction → QC/Review → Rework

Tác vụ

vị nhãn)

Image classification

và guideline.

taxonomy. Box quá rộng/hẹp; bỏ

Object detection

sót; box trùng; đối tượng thấy của từng đối tượng; độ sát box, box trùng và nhỏ, bị che hoặc nằm không suy đoán phần bị đối tượng bị bỏ sót.

sát mép ảnh.

Instance segmentation Một class và một

mask/polygon riêng cho nhau; mask lấn nền

từng instance.

khác.

Nhận xét: Prediction của YOLO11 là kết quả do model tạo ra và có thể hỗ trợ annotator/reviewer, nhưng không thay thế ground truth. Ground truth phải được xây dựng theo guideline, sau đó được reviewer kiểm tra

và yêu cầu rework khi phát hiện nhãn sai, thiếu hoặc không nhất quán.

Ground truth là gì? (đơn Lỗi hoặc điểm khó quyết Annotator cần làm gì? Reviewer cần kiểm tra

định

gì?

Một nhãn lớp cho toàn Ảnh có nhiều chủ đề; bộ ảnh, theo taxonomy khó xác định chủ thể

Chọn nhãn theo tiêu chí Kiểm tra nhãn có phản ưu tiên đã thống nhất; ánh nội dung ảnh và có

chính; ảnh không thuộc yêu cầu hỗ trợ nếu

tuân thủ guideline không. Kiểm tra class, số lượng,

không đủ căn cứ. Vẽ box sát phần nhìn

Một class và một bounding box [x1, y1, x2, y2] cho mỗi đối tượng.

che.

Biên mờ; vật thể chạm Tạo polygon theo đường Kiểm tra độ chính xác

biên phần nhìn thấy;

đường biên, vùng mask, sự phân tách instance và tính nhất quán.

hoặc lấn sang instance tách riêng từng

instance.

## 5. An toàn dữ liệu (Data Safety)

- 1. Chỉ sử dụng dữ liệu được cung cấp cho mục đích của bài lab và tuân thủ phạm vi quyền truy cập. 2. Không tự ý chia sẻ ảnh, JSON, thông tin nội bộ hoặc dữ liệu có thể nhận diện cá nhân lên nền tảng không được phép. 3. Không đưa thông tin cá nhân hoặc dữ liệu nhạy cảm vào báo cáo, ảnh minh họa hay repository công khai. 4. Nếu phát hiện dữ liệu không đúng phạm vi, có dấu hiệu nhạy cảm hoặc không rõ quyền sử dụng, cần dừng xử lý và báo người phụ trách/Lab Coach. 5. Không tự ý sửa dữ liệu nguồn để làm cho kết quả có vẻ hợp lý hơn; mọi thay đổi cần có lý do và được ghi

- nhận.

## 6. Tổng kết


Qua bài thực hành, có thể phân biệt ba dạng output chính của YOLO11:

- Classification: dự đoán lớp cho toàn bộ ảnh, trả về thứ hạng lớp và confidence.

- Object detection: xác định lớp và vị trí đối tượng bằng bounding box.

- Instance segmentation: xác định lớp và vùng mask/polygon riêng cho từng đối tượng.

Ba dạng bài toán có đơn vị ground truth khác nhau, do đó guideline và tiêu chí QC cũng phải khác nhau. Classification cần thống nhất cách chọn nhãn toàn ảnh; detection cần quy tắc vẽ box; segmentation cần quy tắc xác định đường biên và tách instance. Trong cả ba trường hợp, confidence của model không phải ground truth và kết quả prediction vẫn cần được kiểm tra.

## Thông tin kỹ thuật được xác nhận từ output

| Trường | Giá trị |
| --- | --- |
| Ultralytics version | 8.4.145 |
| Classification checkpoint | yolo11n-cls.pt |
| Detection checkpoint | yolo11n.pt |
| Segmentation checkpoint | yolo11n-seg.pt |
| Detection threshold | 0.35 |
| Segmentation threshold | 0.35 |
| Detection count được ghi trong output | 53 |
| Segmentation count được ghi trong output | 49 |
| Classification sample | traffic |
| COCO image ID của traffic | 210273 |

## Checklist trước khi nộp

- ☐ Đã đặt báo cáo tại report/REPORT.md.

- ☐ Đã đính kèm classification_predictions.json.

- ☐ Đã đính kèm detection_predictions.json.

- ☐ Đã đính kèm segmentation_predictions.json.

- ☐ Đã đính kèm IMAGE_ATTRIBUTION.md.

- ☐ Đã đính kèm các hình minh họa classification, detection và segmentation.

- ☐ Validation cuối notebook hiển thị PASS.

- ☐ Đã kiểm tra không có thông tin cá nhân hoặc dữ liệu nhạy cảm trong repository.

- ☐ Đã kiểm tra các file JSON và hình ảnh khớp đúng sample ID trước khi nộp.

- ☐ Đã nộp link repository theo yêu cầu của lớp/LMS.

Ghi chú về tính toàn vẹn của báo cáo: Trong dữ liệu nhận được, phần classification JSON là của traffic; các record detection/segmentation được dán cũng mang sample_id = traffic, trong khi ảnh minh họa detection/segmentation là kitchen. Báo cáo giữ nguyên sự khác biệt này và không suy diễn rằng các tọa độ trong JSON traffic thuộc ảnh kitchen. Nếu cần báo cáo chi tiết riêng cho kitchen, cần xuất đúng các record sample_id = kitchen từ hai file JSON tương ứng.
