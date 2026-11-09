## 🏗️ Phase 3 — DEEP-DIVE: Phân tích Quy trình Chuyên sâu

### 1.1. Current-State Workflow (Quy trình Vận hành Thủ công Hiện tại)

Quy trình hiện tại khi khách đặt xe tại các sảnh phức tạp (Sân bay):

|  Bước  | Hành động                                                                | Người thực hiện | Thời gian  | Ghi chú & Ký hiệu                                               |
| :----: | :----------------------------------------------------------------------- | :-------------- | :--------: | :-------------------------------------------------------------- |
| **B1** | Khách nhập điểm đón bằng văn bản tự do (Free-text note) trên App Xanh SM | Khách hàng      |  ~1 phút   | 🔄 **Handoff:** Khách hàng ──> Hệ thống                         |
| **B2** | Tài xế nhận cuốc, di chuyển xe vào khu vực sảnh đón                      | Tài xế          | 5 – 7 phút | Di chuyển vật lý                                                |
| **B3** | Tài xế đọc ghi chú, nhìn biển hiệu/cột sảnh để tìm vị trí khách          | Tài xế          |   3 phút   | 🔴 **Bottleneck:** Mô tả mập mờ ("gần cây ATM", "cột 9 tầng 2") |
| **B4** | Tài xế gọi điện cho khách hỏi đường (vừa lái xe vừa nghe máy)            | Tài xế & Khách  |   3 phút   | 🔴 **Bottleneck + Nguy cơ an toàn giao thông / Tắc sảnh**       |
| **B5** | Gặp được khách và bắt đầu cuốc xe (hoặc khách/tài xế bấm Hủy chuyến)     | Tài xế & Khách  |   1 phút   | 🔄 **Handoff:** Kết thúc đón                                    |

- 🔴 **Bottlenecks:** Bước B3 & B4 ngốn tới **6 phút**, tạo ra sự ức chế cao độ cho cả hai bên và là nguyên nhân trực tiếp khiến **8% cuốc xe tại sảnh bị hủy**.
- ⏱ **Tổng thời gian đón xe trung bình tại sảnh:** **~13 – 15 phút/cuốc** (trong đó mất 6 phút chỉ để tìm thấy nhau).
  _(Chi tiết sơ đồ trực quan xem file `04-workflow-diagram.png`)._

---

### 1.2. Problem Statement (6-field) — Chuẩn Vin Smart Future

| Trường thông tin (Field)    | Nội dung chi tiết                                                                                                                                                                                                                                                                                                                                                                                                          |
| :-------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Actor / Operator**     | Tài xế taxi Xanh SM (GSM) và Hành khách đặt xe tại sảnh lớn.                                                                                                                                                                                                                                                                                                                                                               |
| **2. Current Workflow**     | Khách nhập ghi chú tự do ──> Tài xế vào sảnh đọc ghi chú ──> Không tìm thấy ──> Gọi điện thoại hỏi vị trí nhiều lần ──> Đón khách hoặc Hủy chuyến do chờ quá lâu/tắc làn xe. 5 bước, phụ thuộc hoàn toàn vào giao tiếp thủ công.                                                                                                                                                                                           |
| **3. Bottleneck**           | Bước B3 & B4 (mất 6 phút): Văn bản mô tả của khách thiếu chuẩn hóa, định vị GPS bị nhảy/trôi do tầng hầm hoặc kết cấu mái che kim loại của sân bay, buộc tài xế phải gọi điện thoại nhiều lần.                                                                                                                                                                                                                             |
| **4. Business Impact**      | Toàn quốc có ~120 sảnh lớn (Sân bay Nội Bài, Tân Sơn Nhất, Đà Nẵng) với 40.000 cuốc/ngày. Tỉ lệ hủy cuốc do không thấy nhau là 8% (~3.200 cuốc hủy/ngày). Gây thất thoát **~176,66 Tỷ VNĐ/năm** doanh thu cước xe giá trị cao và lãng phí chi phí chìm (Sunk cost) vé vào cổng, thời gian xếp hàng của tài xế.                                                                                                             |
| **5. Success Metric**       | 1. **Hiệu suất (Efficiency):** Giảm thời gian tìm thấy nhau tại sảnh từ 6 phút xuống dưới 1.5 phút.<br>2. **Dòng tiền (Revenue Recovery):** Giảm tỉ lệ hủy chuyến tại các sảnh lớn từ 8% xuống dưới 2.5% (cứu được 2.200 cuốc/ngày).<br>3. **Chính xác (Accuracy):** Trích xuất đúng mã cột/cửa đón chuẩn đạt $\ge 95\%$.                                                                                                  |
| **6. Operational Boundary** | **ĐƯỢC LÀM:** Trích xuất thực thể (Cột số, Cửa sảnh, Tầng) từ mô tả khách, đối chiếu cơ sở dữ liệu sảnh (Venue DB), đề xuất điểm ghim đón chuẩn lên bản đồ.<br>**CẤM:** Tuyệt đối không tự ý gán điểm đón ngoài phạm vi sảnh đã quy hoạch; không tự động hủy cuốc xe của tài xế.<br>**CẦN DUYỆT (HITL):** Khách hàng phải bấm xác nhận điểm đón chuẩn được AI gợi ý trên màn hình trước khi thông tin được gửi cho tài xế. |

---

### 1.3. AI Fit Matrix & Future-State Flow

#### A. Đánh giá mức độ phù hợp AI (AI-Fit Matrix):

- **Rule / State-machine (Một phần):** Dùng để lọc tọa độ GPS thô (xác định xe và khách đang ở trong bán kính của Sân bay Nội Bài nào).
- **LLM Feature (✔ CHỌN LÀM LÕI):** Cần thiết vì ngôn ngữ mô tả của khách hàng là tiếng Việt tự nhiên, đa dạng, nhiều tiếng lóng hoặc từ viết tắt (vd: _"cột 3 sảnh đến T2", "cửa Highland", "đối diện cây rút tiền"_). Rule-based không thể bóc tách hết các biến thể này.
- **Agentic Loop (Không dùng):** Không cần Agent tự hành vì tác vụ có đầu vào - đầu ra khép kín, có cấu trúc cố định, tránh rủi ro AI tự quyết định sai.

#### B. Quy trình Tương lai (Future-State Flow):

```text
┌────────────────┐     ┌────────────────┐     ┌────────────────┐     ┌────────────────┐
│ Bước 1         │     │ Bước 2         │     │ Bước 3         │     │ Bước 4         │
│ Khách nhập     │ ──> │ 🔵 AI Trích    │ ──> │ 🟢 Khách xác   │ ──> │ 🔵 Hệ thống    │
│ ghi chú tự do  │     │ xuất & Mapping │     │ nhận điểm ghim │     │ ghim vị trí lên│
│ trên App       │     │ mã Cột/Cửa sảnh│     │ (HITL)         │     │ màn hình App Xế│
└────────────────┘     └────────────────┘     └────────────────┘     └────────────────┘
                                                       │
                                                       ▼
                                                ↩️ Fallback:
                                                Nếu AI không chắc chắn
                                                (Confidence < 85%), hệ thống
                                                hiển thị danh sách Cột để khách
                                                chọn thủ công bằng tay.
```

Dưới đây là toàn bộ nội dung được tách biệt rõ ràng, không bị lỗi lồng code block để bạn copy-paste mượt mà.

---

## 🏁 Phase 5 — EVALUATE: Đánh Giá Độ Sẵn Sàng & Quyết Định Đầu Tư

### 5.1. Bảng Quy Đổi Tác Động Ra Tiền (Financial Impact Quantification)

- **Vấn đề cốt lõi được giải quyết:** Hủy chuyến do không tìm thấy khách tại các sảnh lớn (Sân bay Nội Bài, Tân Sơn Nhất, Đà Nẵng, Phú Quốc và hơn 80 TTTM Vincom).
- **Quy mô:** ~120 sảnh lớn, phát sinh ~40.000 cuốc xe giá trị cao/ngày.

| Hạng mục tính toán                       | Giá trị định lượng      | Ghi chú & Giả định                            |
| :--------------------------------------- | :---------------------- | :-------------------------------------------- |
| **Lượng cuốc xe phát sinh tại sảnh**     | 40.000 cuốc/ngày        | Số liệu trung bình toàn mạng lưới GSM         |
| **Giá trị trung bình mỗi cuốc (AOV)**    | 220.000 VNĐ/cuốc        | Đặc thù cuốc sân bay có cước phí cao          |
| **Tỉ lệ hủy chuyến hiện tại (Baseline)** | 8,0% (3.200 cuốc/ngày)  | Do mô tả mập mờ, tắc sảnh, không thấy nhau    |
| **Tỉ lệ hủy chuyến sau khi có AI**       | 2,5% (1.000 cuốc/ngày)  | AI chuẩn hóa điểm ghim cột sảnh chính xác     |
| **Tỉ lệ cuốc xe được cứu vãn**           | 5,5%                    | Giảm 5,5% điểm phần trăm cuốc bị rò rỉ        |
| **Số cuốc xe được cứu mỗi ngày**         | 2.200 cuốc/ngày         | $40.000 \times 5,5\%$                         |
| **Doanh thu phục hồi mỗi ngày**          | 484.000.000 VNĐ/ngày    | $2.200 \times 220.000$ VNĐ                    |
| **Doanh thu phục hồi gộp mỗi năm**       | 176.660.000.000 VNĐ/năm | $484.000.000 \times 365$ ngày                 |
| **Trừ chi phí vận hành & LLM API**       | - 660.000.000 VNĐ/năm   | Chi phí token Gemini Flash + server gateway   |
| **TỔNG GIÁ TRỊ RÒNG DUY NHẤT**           | **≈ 176 TỶ VNĐ / NĂM**  | **Dòng tiền tươi phục hồi trực tiếp cho GSM** |

---

### 5.2. AI Readiness Checklist

| Tiêu chí đánh giá                                                 | Trạng thái | Bằng chứng thực tế & Biện pháp kiểm soát                                                                                                                                           |
| :---------------------------------------------------------------- | :--------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Có sẵn dữ liệu mẫu/logs sạch để test?**                      | **✔ Đạt**  | Đã có sẵn log text ghi chú của hàng triệu chuyến xe GSM tại sân bay/Vincom và bản đồ số hóa danh mục cột đón chuẩn (Venue POI Database).                                           |
| **2. Rủi ro khi AI sai nằm trong tầm kiểm soát (HITL/Fallback)?** | **✔ Đạt**  | **100% có Human-in-the-loop (HITL):** Khách hàng phải bấm xác nhận điểm ghim AI gợi ý. Có cơ chế Fallback: Nếu độ tin cậy $< 85\%$, tự động chuyển về giao diện chọn cột thủ công. |
| **3. Stakeholders sẵn sàng thay đổi quy trình?**                  | **✔ Đạt**  | Cả Khối Vận hành GSM và Tài xế đều cực kỳ ủng hộ vì trực tiếp giải tỏa tắc nghẽn sảnh, giảm cãi vã và tăng thu nhập cho tài xế.                                                    |

---

### 5.3. Quyết định

> **QUYẾT ĐỊNH: GO (TRIỂN KHAI PILOT PHẠM VI HẸP)**

- **Scope:** Thử nghiệm tính năng trích xuất điểm đón bằng AI trong 4 tuần tại **Sân bay Quốc tế Nội Bài**
- **Lý do quyết định:**
  1. **Hiệu quả tài chính vượt trội:** Tiềm năng cứu vãn dòng tiền lên tới **~176 tỷ VNĐ/năm**, vượt xa mọi bài toán tối ưu chi phí văn phòng thông thường.
  2. **Giải pháp công nghệ tinh gọn:** Dùng LLM Feature dạng API trích xuất thực thể (NER & Mapping), chi phí đầu tư rất thấp, thời gian phát triển ngắn.
  3. **Kiểm soát rủi ro triệt để:** Ranh giới vận hành rõ ràng, có con người xác nhận trước khi điều xe đến điểm đón.
- **Điều kiện tiên quyết trước khi mở rộng:**
  - Số hóa đầy đủ tọa độ GPS và tên cột đón tại 120 sảnh lớn.
  - Chạy bộ kiểm thử mở rộng với tối thiểu 50 trường hợp dữ liệu thực tế đạt độ chính xác $\ge 95\%$.
  - Thiết lập độ trễ phản hồi API của mô hình dưới 1,5 giây.

---
