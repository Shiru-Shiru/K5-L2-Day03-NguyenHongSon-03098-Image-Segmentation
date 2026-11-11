# Báo cáo thực hành Day 3

- Mã học viên theo lớp: 2A202603098
- Ngày / CVAT local: 12/11/2026 - CVAT local
- Công cụ đã dùng: CVAT local (Polygon, Brush) + tool local (OpenCV, YOLO11s, MobileSAM, SegFormer) để tạo gợi ý trước rồi tự kiểm.

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | `cp1_holes.zip` | 1 / 1 | 3 |
| cp2_slice | `cp2_slice.zip` | 1 / 1 | 3 |
| cp5_occlusion | `cp5_occlusion.zip` | 1 / 1 | 3 |
| cp3_thin | `cp3_thin.zip` | 1 / 1 | 3 |
| cp4_curb | `cp4_curb.zip` | 1 / 1 | 3 |
| cp6_coverage | `cp6_coverage.zip` | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Đã chạy `scripts/inspect_submissions.py` kiểm tra cả 9 file ZIP trong `submissions/`, tất cả đều báo OK.

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: Ảnh 000000181542.jpg của medium_instance, vẽ người giữa ảnh đàu tiên.
- Class và quy tắc tôi dùng để chọn biên: Class "person" vẽ sát viền từ đầu đến chân của object đó.
- Nếu dùng gợi ý sau đó: vùng gợi ý sai/đúng, hành động sửa/giữ và lý do: Có dùng gợi ý mấy xe lớn ở gần, nhưng hay sót các người đi bộ nhỏ ở xa và đôi chỗ ranh giữa hai nhãn bị hở khe nhỏ. Vào CVAT xem lại từng object, giữ các mask cắt đúng và vẽ bù thêm người bị thiếu.
- Nếu không dùng gợi ý: ghi “không dùng”; vẫn giải thích một quyết định gán nhãn của mình.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `medium_instance` (ảnh `000000181542`, `000000373353`) và `hard_panoptic` (ảnh `000000350023`, `000000460147`) — phần người đi bộ nhỏ, xe máy và mấy cái đèn giao thông.
- Lỗi thuộc loại: thiếu-thừa vật
- Bằng chứng tôi nhìn thấy: Lúc kiểm tra lại và chạy thử tự đánh giá với ground truth 3 tier thì thấy `medium_instance` bị thiếu khá nhiều người (`FN = 25`), còn `hard_panoptic` bị 0 điểm `traffic light` do quên vẽ 4 cái đèn tín hiệu giao thông.
- Quy tắc và hành động sửa: Mở lại 2 task này trên CVAT để vẽ bù thêm mấy người còn thiếu ở `medium_instance` (tăng từ 62 lên 66 object) và vẽ thêm đèn giao thông, xe máy ở `hard_panoptic` (tăng từ 49 lên 56 object).
- Sau sửa đã Save và export lại chưa? Đã Save trên CVAT, export lại và chạy `inspect_submissions.py` thấy OK.

Kết quả tự kiểm tra 3 tier sau khi sửa (chạy local với file đáp án phát thêm, không đưa ground truth lên repo):
`easy_semantic`: 16.8/20 (mIoU 0.778); `medium_instance` tăng từ 7.5/32 lên 9.8/32 (Recall tăng từ 0.65 lên 0.69); `hard_panoptic` tăng từ 10.8/30 lên 13.1/30 (PQ tăng từ 0.362 lên 0.397). Tổng điểm tham khảo 3 tier tăng từ 35.1 lên 39.7/82.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1. `cp1_holes` (`000000144300.jpg`) — xe trailer cắm trại bên trái và kính xe mô tô | Trailer cắm trại không biết nên xếp vào `car`/`truck`/`bus` hay bỏ qua; phần kính chắn gió mô tô có khoét lỗ không | Trailer không có đầu kéo riêng; quy tắc trạm `cp1_holes` yêu cầu giữ kính/khe trong mask | Giữ nguyên kính chắn gió trong mask `motorcycle`, mấy xe phía sau vẫn gán nhãn (`bus`/`truck`/`car`). |
| 2. `cp4_curb` (`7d83710e-4697c3b2.jpg`) — dải đất trồng cây giữa vỉa hè và phần lát bê tông | Gộp dải đất này vào `sidewalk` hay để trống vì không phải phần gạch lát đi bộ | Task này chỉ có 2 lớp `road` và `sidewalk`, phân ranh theo vỉa hè và chức năng | Tách `road` và `sidewalk` ngay mép vỉa hè bê tông, bỏ trống cây/nhà/xe ngoài schema. |
| 3. `hard_panoptic` (`000000460147.jpg`) — làn đường chính và xe tải chở ô tô ở giữa | Làn xe chạy gán `road`, còn xe tải chở ô tô thì tách từng `car` trên thùng hay gộp thành 1 `truck` | Đúng chức năng thì lòng đường là `road`, các xe con nhìn rõ trên thùng vẫn đếm riêng được | Tách riêng từng `car` trên sàn xe tải và giữ `road` cho lòng đường thật. |
