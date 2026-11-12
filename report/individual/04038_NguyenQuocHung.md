# Báo cáo thực hành Day 3 — điền trực tiếp trong fork của bạn

- Mã học viên theo lớp: 04038
- Ngày / CVAT local: 12/11/2026 / http://localhost:8080
- Công cụ đã dùng: CVAT Local, Polygon, Brush 2px

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: Ảnh `000000181542.jpg`, xe ô tô con (`car`) màu bạc ở trung tâm làn đường.
- Class và quy tắc tôi dùng để chọn biên: Class `car`. Tôi vẽ bám sát viền vỏ kim loại thân xe nhìn thấy rõ; bỏ qua phần bóng râm (shadow) dưới gầm xe; bo sát chu vi bánh xe tiếp xúc mặt đường; giữ nguyên phần kính cửa sổ như một phần thuộc về xe theo quy định task.
- Nếu dùng gợi ý sau đó: vùng gợi ý tự động thường bị lem vào bóng râm mặt đường hoặc dính lốp xe vào nhựa đường; tôi dùng polygon thủ công để cắt gọt chuẩn xác theo mép lốp.
- Nếu không dùng gợi ý: Tự vẽ 100% bằng Polygon và Brush nhỏ để đảm bảo không bị phạt lỗi biên.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: Task `cp5_occlusion`, ảnh `000000336232.jpg`.
- Lỗi thuộc loại: Sai lớp (schema ban đầu của task bị cấu hình nhầm thành `road, sidewalk` thay vì bộ nhãn phương tiện).
- Bằng chứng tôi nhìn thấy: Bảng danh sách nhãn trên CVAT không có nhãn `car, motorcycle, person, bus, truck` theo đúng `classes.json` của trạm `cp5_occlusion`.
- Quy tắc và hành động sửa: Sử dụng API cập nhật lại danh sách nhãn chuẩn theo `data/checkpoints/cp5_occlusion/cvat-labels.json`; sau đó tiến hành gán nhãn chiếc taxi vàng bị che khuất và bật thuộc tính `occluded: true` trên đối tượng.
- Sau sửa đã Save và export lại chưa? Đã Save và export lại thành công file `cp5_occlusion.zip` (COCO 1.0), chạy script kiểm tra hợp đồng đạt `[OK]`.
- Kết quả tự đánh giá: Sau khi chuẩn hóa các lớp và khớp biên polygon, script `scripts/inspect_submissions.py` xác nhận đạt hợp đồng `[OK]` cho toàn bộ 9 task, và bộ ba tiers tự chấm đạt trọn vẹn 82.0 / 82 điểm.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `839f7736-abe28069.jpg` (cp3_thin) / Khung giàn thép đỡ biển báo cao tốc | (A) Gộp chung cả giàn thép và biển chỉ dẫn thành `traffic sign`; (B) Tách riêng biển màu xanh là `traffic sign`, thanh giàn và cột kim loại là `pole`. | Đối chiếu `classes.json`: có cả 2 lớp `pole` và `traffic sign`. Quy tắc yêu cầu kiểm tra cấu trúc mỏng của cột/giàn. | Chọn cách (B): tách biển xanh gán `traffic sign`, cột đứng và giàn thép ngang gán `pole`. |
| `7d83710e-4697c3b2.jpg` (cp4_curb) / Mép vỉa hè bị cát bẩn và lá cây rơi vãi | (A) Vẽ đường ranh giới uốn lượn né các đống rác/cát bẩn trên mép đường; (B) Vẽ đường ranh thẳng bám theo sống gờ đá bó vỉa bê tông (`curb`). | Quy tắc `cp4_curb`: Ranh giới chức năng phân định theo bó vỉa, không chạy theo vệt màu hoặc chướng ngại vật ngẫu nhiên trên mặt đường. | Chọn cách (B): vẽ đường phân ranh giữa `road` và `sidewalk` bám thẳng theo gờ đá bó vỉa bê tông. |
| `000000017627.jpg` (cp2_slice) / Khe tiếp giáp giữa sedan đen và wagon trắng | (A) Dùng 1 mask bọc chung cả 2 xe vì màu sắc tối và tiếp giáp khít; (B) Tách đôi thành 2 instance riêng biệt tại khe tiếp giáp. | Quy tắc `cp2_slice`: Cùng class `car` nhưng là 2 đối tượng cá thể riêng biệt, bắt buộc phải tách rời mask. | Chọn cách (B): kẻ đường cắt đứng dứt khoát phân đôi 2 xe, không để pixel nào bị chồng lấn giữa 2 đối tượng. |
