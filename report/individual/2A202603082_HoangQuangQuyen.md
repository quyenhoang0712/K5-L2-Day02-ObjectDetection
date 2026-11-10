# Báo cáo cá nhân — Ngày 2 Object Detection

Đổi tên file theo `MSSV_HoTen.md`, ví dụ `20241234_NguyenVanAn.md`. Điền kết quả thật, xóa dòng hướng dẫn khi nộp. Xem [quy định nộp](../../SUBMISSION.md).

## 1. Thông tin học viên

- Họ tên: Hoàng Quảng Quyền
- MSSV: 2A202603082
- Email: hqq.7.12.03@gmail.com
- Lớp:  L2A
- Link repo bài làm: https://github.com/quyenhoang0712/K5-L2-Day02-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/github/quyenhoang0712/K5-L2-Day02-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb

## 2. Schema và mini guideline

Quy tắc gán nhãn được trình bày chi tiết trong
[GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

Bộ dữ liệu sử dụng 4 lớp:
- 0: car
- 1: truck
- 2: bus
- 3: van

Ba trường hợp cần lưu ý:

1. drive_002: Phương tiện dễ nhầm giữa bus, van và truck.
   Đã rà soát và điều chỉnh class trong quá trình QC.

2. drive_030: Nhiều xe ở xa có kích thước nhỏ, khó nhận diện.
   Cần kiểm tra độ rõ của xe trước khi quyết định gán nhãn.

3. drive_030: Một phương tiện bị cắt bởi rìa ảnh.
   Bounding box chỉ bao quanh phần nhìn thấy,
   không suy đoán phần nằm ngoài khung hình.

Các trường hợp chưa chắc chắn cần giảng viên làm rõ
để bảo đảm tính nhất quán của nhãn.

## 3. Annotation và self-QC


- **Số file nhãn train:** 50/50 file `.txt`, tương ứng với 50 ảnh giao thông.
- **Công cụ gán nhãn:** CVAT.
- **Định dạng:** YOLO Detection, gồm 4 class `car`, `truck`, `bus`, `van`.

**Kết quả kiểm tra cấu trúc:**

- Đã chạy `tools/check_yolo_labels.py`.
- Kết quả: 50/50 file nhãn hợp lệ.
- Class ID và tọa độ đáp ứng yêu cầu định dạng YOLO.

**Kiểm tra trực quan:**

- `drive_002.jpg`: Kiểm tra các phương tiện dễ nhầm class.
- `drive_030.jpg`: Kiểm tra phương tiện ở xa, kích thước nhỏ và các bounding box chưa phù hợp.
- Chưa xác nhận đủ danh sách 10 ảnh được rà trực quan theo yêu cầu.

**Nhận xét:**

Quá trình gán nhãn giúp hiểu rõ cách xác định bounding box và phân biệt các loại phương tiện. Một số tình huống vẫn khó xử lý do xe quá nhỏ, bị che khuất hoặc có đặc điểm gần giống nhau.

**Commit khóa nhãn ban đầu:** Chưa xác nhận được mã commit trước khi nhận Gold Labels.
## 4. Gold QC và rework


| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 465 | 576 |
| FP | 550 | 436 |
| FN | 188 | 77 |
| Precision | 0.4581 | 0.5692 |
| Recall | 0.7121 | 0.8821 |
| F1 | 0.5576 | 0.6919 |
| mean_iou_of_matches | 0.8445 | 0.8502 |
| passed | False | False |

**File kết quả:** [annotation_evaluation.json](../../outputs/annotation_evaluation.json)

**Quá trình rework:**

- Đối chiếu kết quả annotation với bộ Gold Labels để xác định các trường hợp chưa khớp.
- Phát hiện nhiều trường hợp nhầm lẫn giữa `bus` và `van`, `car` và `truck`.
- Sử dụng Python với sự hỗ trợ của ChatGPT để phân tích nhãn và điều chỉnh 114 trường hợp khác class qua các bước.
- Phát hiện và loại bỏ 3 bounding box gần trùng cùng class.
- Kiểm tra thêm các phương tiện nhỏ, box có tỷ lệ bất thường và các ảnh có nhiều FP/FN.
- Một số chỉnh sửa theo Gold chưa được xác nhận lại hoàn toàn bằng mắt.

**Nhận xét:**

F1 tăng từ 0.5576 lên 0.6919. Mean IoU đạt 0.8502, vượt ngưỡng tối thiểu 0.70.

Tuy nhiên F1 vẫn thấp hơn yêu cầu 0.80. Bộ nhãn cần tiếp tục được kiểm tra để cải thiện tính nhất quán khi phân loại phương tiện và xác định bounding box.

## 5. Kết quả YOLO

-## 5. Kết quả YOLO

**Cấu hình huấn luyện:**

- Model: `yolo26n.pt`
- Epochs: 50
- Image size: 640
- GPU: NVIDIA Tesla T4
- Ultralytics: 8.4.208
- Python: 3.13.16
- PyTorch: 2.14.1+cu130
- Train: 50 ảnh
- Eval: 10 ảnh, 113 đối tượng
- Checkpoint: `vehicle_annotation/weights/best.pt` (lưu trên Colab, không đưa file `.pt` lên GitHub)

**Kết quả đánh giá trên tập eval:**

| Chỉ số | Giá trị |
|---|---:|
| Precision | 0.8149 |
| Recall | 0.3510 |
| mAP50 | 0.4692 |
| mAP50–95 | 0.2820 |

**Minh chứng kết quả:**

- [Ảnh dự đoán drive_052](assets/eval_predictions/visuals/drive_052.jpg)
- [Ảnh dự đoán drive_054](assets/eval_predictions/visuals/drive_054.jpg)
- [Log huấn luyện](assets/yolo26_runs/vehicle_annotation/results.csv)
- [Biểu đồ đánh giá mô hình](assets/yolo26_runs/vehicle_eval/)

**Nhận xét:** Mô hình hoàn thành 50 epochs. Precision đạt 0.8149 nhưng Recall chỉ đạt 0.3510, cho thấy nhiều phương tiện chưa được phát hiện. mAP50–95 thấp hơn mAP50, phản ánh khả năng định vị box ở các ngưỡng IoU cao vẫn còn hạn chế.

**Lưu ý:** Đây là kết quả chạy thử khi F1 annotation trước train đạt 0.6919, chưa vượt ngưỡng 0.80 theo yêu cầu CP5.

### Trả lời bốn câu hỏi cuối notebook

**1. Lớp nào Recall thấp nhất? Ảnh nào minh họa?**

Lớp `bus` có Recall thấp nhất, bằng **0.000**, trên tổng cộng 2 đối tượng bus trong tập eval.

Ví dụ, trong ảnh `drive_052.jpg` có một xe buýt lớn trên đường nhưng kết quả dự đoán chưa hiển thị bounding box `bus` tương ứng. Điều này minh họa tình trạng mô hình bỏ sót phương tiện.

[Ảnh minh họa drive_052](assets/eval_predictions/visuals/drive_052.jpg)

Do tập eval chỉ có 2 đối tượng bus, chưa thể kết luận khả năng nhận diện bus của mô hình trên mọi dữ liệu.

**2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**

Cần kiểm tra mức độ bounding box bám sát vật thể, kích thước box, độ lệch tâm và khả năng bao phủ đúng phần xe.

mAP50 chỉ đánh giá tại ngưỡng IoU 0.50, còn mAP50–95 tính trung bình trên nhiều ngưỡng IoU khắt khe hơn. Vì vậy, box có thể phát hiện đúng xe nhưng chưa đủ chính xác về vị trí hoặc kích thước.

**3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**

Các nhóm class dễ nhầm lẫn gồm `bus` với `van` và `car` với `truck`.

Trong giai đoạn Gold QC, đã phát hiện nhiều bounding box trùng vị trí nhưng khác class giữa nhãn cá nhân và Gold Labels. Đây là bằng chứng về sự chưa nhất quán trong phân loại annotation.

Trên tập eval, Recall của `bus` bằng 0.000, `truck` bằng 0.200 và `van` bằng 0.417. Các kết quả này cho thấy mô hình vẫn gặp khó khăn với một số loại phương tiện, nhưng chưa đủ bằng chứng để khẳng định nguyên nhân chỉ do guideline. Cần xem thêm confusion matrix và ảnh dự đoán.

**4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**

Không sử dụng 10 ảnh eval để cập nhật trọng số mô hình, vì việc đó sẽ làm sai lệch quá trình đánh giá trên dữ liệu không dùng để huấn luyện.

Trong notebook, 50 ảnh train được dùng để huấn luyện, còn 10 ảnh eval được dùng làm validation nhằm theo dõi chất lượng, chọn checkpoint tốt nhất và tính các chỉ số Precision, Recall, mAP50, mAP50–95.

Do tập eval được dùng để lựa chọn checkpoint, kết quả này là validation performance, chưa phải kết quả trên một test set độc lập.

## 6. Khó khăn và cách xử lý


| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Annotation | Khó phân biệt các phương tiện có đặc điểm gần giống nhau | Xây dựng quy tắc phân biệt 4 class và rà lại nhãn | Hoàn thành 50 file nhãn |
| Self-QC | Cần kiểm tra cấu trúc và tọa độ các bounding box | Chạy công cụ kiểm tra định dạng YOLO | 50/50 file hợp lệ |
| Gold QC | Nhiều trường hợp sai class và nhãn chưa khớp | Phân tích kết quả, kiểm tra lỗi và thực hiện rework có hỗ trợ bằng Python | F1 tăng từ 0.5576 lên 0.6919 |
| Rework | Một số box bị trùng hoặc khó xác định do vật thể ở xa | Kiểm tra các box chồng nhau và ảnh trực quan | Loại bỏ 3 box trùng; còn lỗi chưa xử lý |
| Huấn luyện | Cần chuẩn bị dữ liệu đúng cấu trúc train/eval | Kiểm tra dữ liệu trên Google Colab | Đủ 50 train và 10 eval |
| Đánh giá mô hình | Recall thấp, đặc biệt với lớp `bus` | Phân tích metrics và ảnh dự đoán | Recall toàn bộ đạt 0.3510; cần cải thiện thêm |

**Kết luận:** Khó khăn chính nằm ở chất lượng annotation, tính nhất quán khi phân loại phương tiện và khả năng nhận diện các đối tượng khó. Bài thực hành giúp hiểu rõ mối liên hệ giữa chất lượng dữ liệu và kết quả mô hình Object Detection.

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md), kết quả tự đánh giá bài thực hành như sau:

| Tiêu chí | Điểm tối đa | Kết quả tự đánh giá |
|---|---:|---|
| Hoàn thành và định dạng YOLO | 10 | Đạt kiểm tra cấu trúc: 50/50 file nhãn hợp lệ, đúng định dạng YOLO. |
| Đầy đủ vật thể | 20 | Đã gán nhãn 50 ảnh, tuy nhiên còn 77 FN và 436 FP so với Gold. Cần tiếp tục cải thiện. |
| Độ bám sát và hình học box | 20 | Mean IoU đạt 0.8502 trên các box khớp. Tuy nhiên chưa thể khẳng định tất cả box đều chính xác. |
| Phân lớp nhất quán | 15 | Đã rà soát và chỉnh sửa các trường hợp nhầm giữa car, truck, bus và van. Một số chỉnh sửa chưa được xác minh trực quan đầy đủ. |
| Mini guideline và quyết định mơ hồ | 10 | Đã xây dựng guideline với 4 class, quy tắc bounding box và 3 tình huống cần lưu ý. |
| Gold-set annotation QC | 15 | Đã thực hiện evaluation và rework. F1 tăng từ 0.5576 lên 0.6919 nhưng chưa đạt ngưỡng 0.80. Mean IoU đạt 0.8502. |
| Pipeline YOLO và phân tích | 10 | Đã train YOLO26n đủ 50 epochs, đánh giá trên 10 ảnh eval và lưu kết quả dự đoán, metrics cùng log huấn luyện. |

**Những phần đã hoàn thành:**

- Hoàn thành 50 file nhãn YOLO hợp lệ.
- Xây dựng mini guideline và thực hiện kiểm tra chất lượng nhãn.
- Có kết quả Gold QC trước và sau rework.
- Hoàn thành huấn luyện YOLO26n trên Google Colab.
- Đánh giá mô hình trên 10 ảnh eval, không đưa ảnh eval vào tập train.
- Có kết quả Precision, Recall, mAP50, mAP50–95 và ảnh dự đoán.

**Những phần chưa đạt hoặc cần hoàn thiện:**

- F1 annotation mới đạt 0.6919, thấp hơn yêu cầu 0.80.
- Còn nhiều trường hợp FP/FN cần tiếp tục rà soát.
- Chưa xác nhận đầy đủ minh chứng self-QC tối thiểu 10 ảnh.
- Chưa xác nhận được commit khóa nhãn ban đầu trước khi nhận Gold.
- Một số thay đổi class trong rework chưa được kiểm tra lại hoàn toàn bằng mắt.

**Kết luận:**

Bài thực hành đã hoàn thành các bước chính của quy trình Object Detection, bao gồm annotation, kiểm tra cấu trúc, Gold QC, huấn luyện và đánh giá YOLO26n.

Tuy nhiên, chất lượng annotation chưa đạt đầy đủ ngưỡng yêu cầu. Cần tiếp tục cải thiện khả năng phân loại và tính nhất quán của bounding box.

Điểm số cuối cùng do giảng viên đánh giá theo rubric và minh chứng bài nộp. Không tự xác nhận điểm tối đa cho các tiêu chí chưa đủ bằng chứng.

## 8. Công cụ AI và tài liệu tham khảo

**Công cụ sử dụng:**

- CVAT: Tạo và kiểm tra annotation.
- Python: Kiểm tra cấu trúc dữ liệu và hỗ trợ quá trình QC.
- Google Colab và Ultralytics: Huấn luyện, đánh giá mô hình YOLO26n.
- ChatGPT: Hỗ trợ giải thích kiến thức, xử lý lỗi kỹ thuật, xây dựng script kiểm tra/chỉnh sửa nhãn sau Gold QC và trình bày báo cáo.

**Giới hạn sử dụng:**

Một số thao tác rework được thực hiện tự động sau khi đối chiếu Gold Labels và chưa được xác nhận hoàn toàn bằng mắt. Kết quả này được ghi nhận là hạn chế của bài thực hành.

**Tài liệu tham khảo:**

- `README.md`
- `GUIDE.md`
- `RULES.md`
- `RUBRIC.md`
- `SUBMISSION.md`
- `GUIDELINE_MINI.md`