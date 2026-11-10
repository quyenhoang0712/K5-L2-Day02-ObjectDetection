# Báo cáo cá nhân — Ngày 2 Object Detection

Đổi tên file theo `MSSV_HoTen.md`, ví dụ `20241234_NguyenVanAn.md`. Điền kết quả thật, xóa dòng hướng dẫn khi nộp. Xem [quy định nộp](../../SUBMISSION.md).

## 1. Thông tin học viên

- Họ tên: Hoàng Quảng Quyền
- MSSV: 2A202603082
- Email: hqq.7.12.03@gmail.com
- Lớp:  L2A
- Link repo bài làm: https://github.com/quyenhoang0712/K5-L2-Day02-ObjectDetection
- Link Colab đã chạy:

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

- **Số file nhãn train:** 50/50 file `.txt`, tương ứng với 50 ảnh trong `dataset/images/train/`.

- **Kết quả checker và lỗi đã xử lý:**
  - Sử dụng CVAT để gán nhãn bốn lớp `car`, `truck`, `bus`, `van`.
  - Xuất annotation theo định dạng YOLO Detection và lưu vào `dataset/labels/train/`.
  - Chạy lệnh `python3 tools/check_yolo_labels.py`.
  - Kết quả: `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ`.
  - Không phát hiện lỗi định dạng nhãn, class ID hoặc tọa độ vượt biên tại lần kiểm tra cuối.

- **Danh sách ít nhất 10 ảnh đã rà bằng mắt:**
  - Đã xác nhận kiểm tra trực quan: `drive_002.jpg`, `drive_030.jpg`.
  - Các ảnh còn lại chưa có đủ ghi nhận kiểm tra trực quan để xác nhận hoàn thành yêu cầu rà tối thiểu 10 ảnh.

- **Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:**
  - `drive_002.jpg`: Phát hiện hai trường hợp phân loại khác với Gold giữa `bus`, `van` và `truck`; đã điều chỉnh nhãn sau khi đối chiếu.
  - `drive_030.jpg`: Phát hiện nhiều bounding box nhỏ ở xa camera, khó xác định chính xác loại phương tiện. Đã kiểm tra trực quan và giữ lại các box chưa đủ căn cứ để loại bỏ.
  - Sử dụng Python để kiểm tra các bounding box gần trùng nhau, tỷ lệ hình học bất thường và các nhãn cần rà soát.
  - Các thay đổi sau khi nhận Gold Labels được trình bày chi tiết tại Phần 4 — Gold QC và rework.

- **Commit khóa nhãn ban đầu:** Chưa có mã commit được xác nhận trước khi nhận Gold Labels. Cần bổ sung nếu có commit tương ứng trong lịch sử Git.

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

- `drive_002`: Kiểm tra và sửa 2 nhãn khác class (`bus` → `van` và `bus` → `truck`).
- Các ảnh train: Sử dụng Python để phát hiện và điều chỉnh thêm 112 trường hợp khác class dựa trên kết quả đối chiếu với Gold. Các thay đổi này chưa được xác minh trực quan đầy đủ.
- `drive_030`, `drive_031`, `drive_003`: Phát hiện nhiều box chưa khớp Gold, kiểm tra các vật thể nhỏ và box ở xa.
- Kiểm tra các bounding box gần trùng và loại bỏ 3 box trùng cùng class. Những box không chắc chắn được giữ lại để tiếp tục xem xét.

**Nhận xét:** Sau rework, F1 tăng từ 0.5576 lên 0.6919, Mean IoU đạt 0.8502. Tuy nhiên F1 vẫn chưa đạt mức yêu cầu 0.80. Các trường hợp FP/FN còn lại cần tiếp tục kiểm tra trực quan để xác định nguyên nhân; không thể kết luận chỉ dựa trên số liệu.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch:
- Các tham số thay đổi so với notebook nếu có và lý do:
- Đường dẫn best checkpoint trong phiên chạy:

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | |
| Recall | |
| mAP50 | |
| mAP50–95 | |

Đặt cả 2 folder `assets/eval_predictions/` và `assets/yolo26_runs/` (giải nén từ `day2_colab_outputs.zip`) cạnh báo cáo và dẫn link tương đối. Không chỉ ghi số; chỉ ra ít nhất một ảnh và vật thể cụ thể.

Trả lời bốn câu hỏi cuối notebook:

1. Lớp nào recall thấp nhất? Ảnh nào minh họa?
2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?
3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?
4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| | | | |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md): phần nào đã có kết quả, phần nào còn thiếu hoặc chưa kiểm được. Không tự nhận hoàn thành bước chưa chạy.

## 8. Công cụ AI và tài liệu tham khảo

Ghi công cụ đã dùng, dùng ở bước nào và phần đã tự kiểm tra; không dùng thì ghi rõ. Liệt kê nguồn ngoài repo được sử dụng. Không ghi mật khẩu/token hoặc đính kèm gold train.
