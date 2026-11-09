## 1. Các Kết Quả AI Đã Hỗ Trợ

1. **Phân định cấu trúc bài tập và quy tắc phân nhánh Git:**
   - Làm rõ cơ chế chấm điểm độc lập cho mã nguồn Python trên nhánh cá nhân (không merge vào nhánh main).
   - Xác định trách nhiệm của nhóm trong việc chọn lọc báo cáo markdown để merge vào nhánh main phục vụ chấm điểm chung.
2. **Mô hình hóa tác động kinh tế bằng số liệu định lượng:**
   - Quy đổi giá trị bài toán sang đơn vị VNĐ dựa trên quy mô vận hành thực tế của GSM.
   - Phân biệt rõ sự khác nhau giữa bài toán cắt giảm chi phí vận hành tại Vinhomes (khoảng 12 tỷ VNĐ/năm) và bài toán phục hồi doanh thu trực tiếp tại Xanh SM (khoảng 176,6 tỷ VNĐ/năm), cung cấp căn cứ để nhóm lựa chọn đề tài.
3. **Chuẩn hóa cấu trúc tài liệu:**
   - Soạn thảo bảng phân tích vấn đề 6 trường, ma trận đánh giá công nghệ và quy trình tương lai có tích hợp bước kiểm duyệt của con người cùng cơ chế dự phòng.

---

## 2. Các Sai Lệch Của AI và Biện Pháp Can Thiệp Điều Chỉnh

Trong quá trình làm việc, AI đã xuất hiện 3 lỗi sai lệch về nghiệp vụ và định dạng. Tôi đã trực tiếp phát hiện và xử lý như sau:

|  STT  | Sai lệch của AI                                  | Nguyên nhân                                                                                                                | Biện pháp can thiệp của học viên                                                                                                               |
| :---: | :----------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------- |
| **1** | Bỏ qua yêu cầu làm việc độc lập của từng cá nhân | AI có xu hướng đi tắt, tự động chọn một đề tài chung cho cả nhóm ngay từ đầu.                                              | Yêu cầu AI bám sát tài liệu hướng dẫn, phân định rõ Phase 1 và Phase 2 là nhiệm vụ cá nhân độc lập trên từng nhánh riêng trước khi nhóm họp.   |
| **2** | Lỗi hiển thị cú pháp Markdown                    | AI sử dụng các khối mã nguồn lồng nhau không đúng quy cách, khiến văn bản phía sau bị ngắt quãng và mất nội dung.          | Nhắc nhở AI về lỗi hiển thị, yêu cầu chuyển đổi toàn bộ nội dung sang văn bản thuần và bảng biểu chuẩn để sao chép thuận tiện.                 |
| **3** | Sai lệch ngữ cảnh của tài liệu phản ánh          | AI tự tạo lập một nhật ký kiểm thử kỹ thuật không có thật thay vì ghi nhận quá trình làm việc thực tế của phiên đối thoại. | Yêu cầu AI dừng việc tạo dữ liệu giả lập, viết lại nội dung dựa trên đúng tiến trình tương tác và các vấn đề đã phát sinh trong buổi làm việc. |

---

## 3. Kỹ Thuật Điều Chỉnh Câu Lệnh Đã Áp Dụng

1. **Ràng buộc tiêu chí đánh giá bằng biến số kinh tế:**
   Thay vì yêu cầu so sánh bài toán chung chung, tôi đặt câu lệnh ràng buộc AI phải lấy giá trị tài chính (VNĐ) làm thước đo duy nhất để xếp hạng mức độ ưu tiên.
2. **Cung cấp dữ liệu gốc làm ranh giới thông tin:**
   Tải trực tiếp tệp hướng dẫn và hình ảnh ghi chú từ lớp học để giới hạn phạm vi xử lý của AI, hạn chế tình trạng mô hình tự suy đoán dữ liệu.
3. **Phân rã tác vụ theo trình tự logic:**
   Chia nhỏ yêu cầu từ việc quét bài toán, lập thẻ đánh giá nhanh, tính toán dòng tiền đến hoàn thiện báo cáo chuyên sâu, kiểm tra kết quả từng bước trước khi chuyển sang tác vụ tiếp theo.

---

## 4. Bài Học Thực Tiễn

- **Tính tất yếu của việc kiểm duyệt thông tin:** AI có tốc độ xử lý và khả năng tổng hợp dữ liệu cao, nhưng thường xuyên mắc lỗi trôi ngữ cảnh hoặc hiểu sai quy chế nếu thiếu sự giám sát của người dùng.
- **Vai trò của con người trong chu trình AI:** Trong mọi quy trình nghiệp vụ, con người luôn phải là mắt xích kiểm tra và ra quyết định cuối cùng, cả trong quá trình xây dựng tài liệu lẫn khi vận hành hệ thống thực tế.
