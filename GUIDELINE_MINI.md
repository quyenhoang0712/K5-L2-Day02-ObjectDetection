# Mini guideline — bài làm cá nhân

## 1. Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
|---|---|---|
| `car` | Xe ô tô con phục vụ di chuyển cá nhân, bao gồm sedan, SUV, taxi và xe bán tải. | Xe 4–7 chỗ, SUV, taxi. Không nhầm với xe tải hoặc xe buýt. |
| `truck` | Xe tải chuyên dùng để vận chuyển hàng hóa, bao gồm xe tải thùng và xe đầu kéo. | Phân biệt với `car` dựa trên thiết kế và chức năng chở hàng. |
| `bus` | Xe chuyên chở hành khách, thường có nhiều hàng ghế và cửa sổ, bao gồm xe buýt và minibus chở khách. | Phân biệt với `van` dựa trên thiết kế xe và mục đích chuyên chở hành khách. |
| `van` | Xe van hoặc minivan có khoang thân kín, dùng để chở hàng hoặc chở người. | Phân biệt với `bus` qua kiểu thân xe, kích thước và cấu trúc khoang hành khách. |

## 2. Quy tắc chung

- **Box bám sát vật thể:** Bounding box phải bao quanh phần thân xe nhìn thấy được, hạn chế lấy thêm mặt đường, nền hoặc các vật thể xung quanh. Mỗi xe được gán một box riêng.
- **Vật thể bị che khuất:** Nếu xe bị che một phần nhưng vẫn đủ thông tin để nhận diện, tiếp tục gán nhãn cho phần nhìn thấy. Không gộp hai xe thành một box.
- **Vật thể bị cắt bởi rìa ảnh:** Chỉ gán bounding box trong phạm vi ảnh, không suy đoán phần thân xe nằm ngoài khung hình. Nếu không đủ thông tin xác định loại xe thì cần kiểm tra lại.
- **Vật thể rất nhỏ/mơ hồ:** Chỉ gán nhãn khi có thể nhận diện loại xe tương đối chắc chắn. Với xe quá xa, ảnh mờ hoặc không đủ đặc điểm phân biệt, không tự suy đoán class.
- **Vật thể ngoài phạm vi:** Không gán xe máy, xe đạp, người đi bộ, biển báo hoặc các đối tượng không thuộc bốn class quy định.
- **Tính nhất quán:** Giữ nguyên thứ tự class ID: `car = 0`, `truck = 1`, `bus = 2`, `van = 3`.

## 3. Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
|---|---|---|---|
| `drive_002.jpg` | Một phương tiện ban đầu được gán `bus`, nhưng có đặc điểm dễ nhầm với `van`. | Rà soát và điều chỉnh class từ `bus` sang `van` tại box dòng 9. | Xe thuộc nhóm van/minivan theo cách phân loại của bộ nhãn chuẩn; cần phân biệt với xe buýt chở khách. |
| `drive_002.jpg` | Một phương tiện khác được gán `bus` nhưng có đặc điểm dễ nhầm với `truck`. | Rà soát và điều chỉnh class từ `bus` sang `truck` tại box dòng 10. | Phân loại theo loại phương tiện và đặc điểm vận chuyển hàng hóa, không chỉ dựa vào kích thước. |
| `drive_030.jpg` | Nhiều phương tiện ở rất xa, kích thước nhỏ và khó xác định class. | Giữ những xe còn đủ đặc điểm nhận diện; đánh dấu các box quá nhỏ hoặc mơ hồ để kiểm tra lại, không tự động xóa toàn bộ. | Guideline yêu cầu gán xe nhận diện được, nhưng không suy đoán class khi thông tin hình ảnh không đủ rõ ràng. |

## 4. Kiểm tra chất lượng nhãn

Sau khi hoàn thành gán nhãn 50 ảnh, sử dụng script `check_yolo_labels.py` để kiểm tra định dạng và `evaluate_annotations.py` để so sánh với Gold Labels do giảng viên cung cấp.

**Kết quả đánh giá cuối cùng:**

| Chỉ số | Kết quả |
|---|---:|
| File nhãn hợp lệ | 50/50 |
| Precision | 0.5692 |
| Recall | 0.8821 |
| F1 Score | 0.6919 |
| Mean IoU | 0.8502 |
| True Positive | 576 |
| False Positive | 436 |
| False Negative | 77 |

Kết quả cho thấy độ trùng khớp vị trí của các box được ghép tương đối tốt, với Mean IoU đạt 0.8502. Tuy nhiên, F1 vẫn thấp hơn ngưỡng yêu cầu 0.80, chủ yếu do còn nhiều box chưa khớp với Gold Labels.

Trong quá trình kiểm tra, đã phát hiện các trường hợp khác class, bounding box gần trùng và nhiều xe nhỏ ở xa. Một số nhãn đã được điều chỉnh bằng Python và kiểm tra lại. Các trường hợp chưa chắc chắn cần tiếp tục rà soát theo ảnh và annotation guideline.

**Kết luận:** Bộ nhãn đã đạt yêu cầu về cấu trúc YOLO và ngưỡng Mean IoU, nhưng chưa đạt ngưỡng F1 của bài thực hành. Cần tiếp tục cải thiện tính nhất quán khi phân loại và xác định phạm vi vật thể cần gán nhãn.