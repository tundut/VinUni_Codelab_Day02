# 01 - Problem Scan & Quick Assessment (Vin Smart Future)

- **Học viên thực hiện:** Hồ Anh Tú (AI Product Engineer — Khối Di chuyển Thông minh & Dịch vụ Vingroup)
- **Đơn vị:** Vin Smart Future (Vingroup)
- **Mục tiêu:** Quét tìm các cơ hội tự động hóa/tối ưu vận hành bằng AI qua 4 Lenses và đánh giá nhanh 3 bài toán tiềm năng nhất.

---

# 🔍 Phase 1 — SCAN: Tìm kiếm cơ hội tối ưu hóa vận hành

Áp dụng **4 Lenses** (Lặp lại, Tốn thời gian, AI có thể tốt hơn, Pain từ người khác) để quét qua các công ty thành viên trong hệ sinh thái Vingroup:

|   #   | Subsidiary   | Lens                   | Mô tả ngắn bài toán & Điểm nghẽn vận hành                                                                                                                                                                                                                                                                                                                                                    |
| :---: | :----------- | :--------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1** | **Xanh SM**  | **Tốn thời gian**      | **Tự động phân loại & trích xuất lý do hủy chuyến thực địa:** Hằng ngày có hàng nghìn cuốc xe bị hủy vào giờ cao điểm hoặc trời mưa. Nhân viên vận hành phải nghe thủ công file ghi âm cuộc gọi và đọc ghi chú ngắn của tài xế để thống kê nguyên nhân gốc rễ (do app định vị sai, tài xế không liên lạc được hay khách đợi quá lâu), mất 8-10 phút/lượt phân tích.                          |
| **2** | **VinFast**  | **AI có thể tốt hơn**  | **Trợ lý định tuyến trạm sạc thông minh theo tình trạng pin & công suất:** Người dùng xe điện (VF5, VF8, VF9) thường xuyên gặp tình trạng đến trạm sạc nhưng hết trụ trống hoặc trụ không đúng công suất sạc nhanh yêu cầu. Hệ thống hiện tại chỉ hiển thị danh sách trạm tĩnh mà chưa tự động dự báo luồng xe và đề xuất trạm sạc tối ưu theo hành trình thời gian thực.                    |
| **3** | **Xanh SM**  | **Pain từ người khác** | **Điều phối và làm rõ điểm đón taxi điện tại các điểm nghẽn sảnh:** Khách hàng đặt xe tại các sảnh phức tạp (Sân bay Nội Bài/Tân Sơn Nhất) thường ghi chú điểm đón mơ hồ (_"đang đứng ở cột số 3 gần cây ATM"_), tài xế không tìm thấy khách dẫn đến gọi điện cãi vã, tắc nghẽn sảnh và tỉ lệ hủy cuốc tăng 18%.                                                                             |
| **4** | **Vinhomes** | **Lặp lại**            | **Phân loại và điều hướng sự cố kỹ thuật căn hộ trên App Vinhomes Resident:** Mỗi ngày Ban Quản lý tiếp nhận hàng trăm yêu cầu sửa chữa (điện nước, thấm dột, khóa cửa). Nhân viên CSKH phải đọc tin nhắn, tra mã căn hộ và gán thủ công cho từng tổ kỹ thuật chuyên trách, tốn 10-15 phút/ticket.                                                                                           |
| **5** | **Vinpearl** | **Pain từ người khác** | **Cảnh báo và phân luồng review tiêu cực khẩn cấp của du khách:** Quét các đánh giá trực tuyến (Google Maps, TripAdvisor, Agoda) của các cơ sở Vinpearl/VinWonders. Hiện tại đội ngũ PR/CSKH tổng hợp thủ công theo tuần, khiến các sự cố khẩn cấp (ngộ độc thực phẩm, phòng hỏng điều hòa giữa mùa hè) không được xử lý ngay trong ngày, ảnh hưởng nghiêm trọng đến danh tiếng thương hiệu. |

---

# 🃏 Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards

Từ 5 bài toán trên, chọn lọc **Top 3 bài toán tiềm năng nhất** có tính khả thi kỹ thuật cao, ranh giới rõ ràng và mang lại giá trị kinh tế trực tiếp cho Vingroup:

---

### 🎴 QUICK PROBLEM CARD #1 (Ưu tiên số 1 — Bài toán cốt lõi)

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                       │
│                                                             │
│ Bài toán (1 câu): Phân tích tự động lý do hủy chuyến từ ghi │
│ chú tài xế và log cuộc gọi để cảnh báo rò rỉ cuốc tại       │
│ các điểm nóng (Hotspots) theo thời gian thực.              │
│ Công ty thành viên: [ ] VinFast  [x] Xanh SM  [ ] Vinhomes  │
│                     [ ] Vinmec   [ ] Khác                   │
│                                                             │
│ Ai đang đau (Actor)? Chuyên viên Vận hành Điều vận Xanh SM  │
│ (Dispatcher Ops Analyst) & Đội ngũ Quản lý Đội xe.          │
│                                                             │
│ Workflow thủ công hiện tại (4 bước):                        │
│   1. Xuất file log các cuốc xe hủy đột biến (>15% / giờ)    │
│   ──> 2. Đọc thủ công ghi chú tài xế + nghe lại file ghi âm │
│   ──> 3. Điền mã phân loại lỗi (App lag/Tắc đường/Khách hủy)│
│   ──> 4. Tổng hợp báo cáo Excel gửi Giám đốc Vận hành tỉnh  │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2 & 3 (⏱ 8-10 min/lượt)│
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2 & 3: LLM đọc   │
│ text ghi chú + speech-to-text log để phân loại lý do tự động│
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│   1. Rút ngắn thời gian phân loại sự cố từ 8 min ──> <10s.   │
│   2. Độ chính xác trích xuất đúng lý do đạt >= 92%.         │
│   3. Giảm tỉ lệ hủy chuyến tại điểm nóng thêm 3.5% sau 1    │
│      tháng áp dụng cảnh báo điều vận.                       │
│                                                             │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent │
└─────────────────────────────────────────────────────────────┘
```

┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2 │
│ │
│ Bài toán (1 câu): Tự động chuẩn hóa mô tả điểm đón mập mờ │
│ của khách tại sảnh Sân bay thành chỉ dẫn định vị chuẩn │
│ gửi thẳng lên màn hình định vị xe Xanh SM. │
│ Công ty thành viên: [ ] VinFast [x] Xanh SM [ ] Vinhomes │
│ [ ] Vinmec [ ] Khác │
│ │
│ Ai đang đau (Actor)? Tài xế taxi Xanh SM & Khách đón xe. │
│ │
│ Workflow thủ công hiện tại (4 bước): │
│ 1. Khách gõ ghi chú điểm đón tự do trên ứng dụng │
│ ──> 2. Tài xế đến sảnh nhưng không nhìn thấy khách │
│ ──> 3. Tài xế gọi điện cho khách hỏi đường (vừa lái vừa nghe)│
│ ──> 4. Tìm thấy nhau hoặc khách bức xúc ấn hủy chuyến │
│ │
│ Bước nào tốn thời gian/lỗi nhất? Bước 3 (⏱ 4-6 min/cuốc đón)│
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 1-2: AI phân tích│
│ text mô tả của khách ──> map với cơ sở dữ liệu các Cột đón/ │
│ Cửa sảnh của Sân bay ──> hiển thị tọa độ ghim. │
│ │
│ Đo thành công bằng gì (Metric có số)? │
│ 1. Giảm thời gian tìm thấy khách từ 5 phút ──> dưới 1.5 phút.│
│ 2. Giảm tỉ lệ hủy chuyến do không tìm thấy khách từ 8% ──> │
│ dưới 2.5% tại các sảnh lớn. │
│ │
│ Quick Architecture: [ ] No AI [ ] Rule [x] LLM [ ] Agent │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3 │
│ │
│ Bài toán (1 câu): Phân loại tự động và điều hướng ticket sự │
│ cố kỹ thuật căn hộ trên App Vinhomes Resident đến đúng đội. │
│ Công ty thành viên: [ ] VinFast [ ] Xanh SM [x] Vinhomes │
│ [ ] Vinmec [ ] Khác │
│ │
│ Ai đang đau (Actor)? Nhân viên CSKH Ban Quản lý Vinhomes. │
│ │
│ Workflow thủ công hiện tại (4 bước): │
│ 1. Tiếp nhận tin nhắn báo hỏng từ cư dân trên hệ thống │
│ ──> 2. CSKH đọc nội dung, xác định loại sự cố (Điện/Nước) │
│ ──> 3. Tra cứu ca trực kỹ thuật viên của tòa nhà │
│ ──> 4. Gõ tin nhắn xác nhận lịch hẹn và chuyển giao ticket│
│ │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2 & 4 (⏱ 12 min/ticket)│
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2 & 4: LLM trích │
│ xuất mức độ khẩn cấp, loại thiết bị hỏng và draft sẵn tin. │
│ │
│ Đo thành công bằng gì (Metric có số)? │
│ 1. Giảm thời gian xử lý ban đầu từ 15 phút ──> dưới 2 phút. │
│ 2. Độ chính xác gán đúng tổ kỹ thuật đạt >= 95%. │
│ │
│ Quick Architecture: [ ] No AI [ ] Rule [x] LLM [ ] Agent │
└─────────────────────────────────────────────────────────────┘
