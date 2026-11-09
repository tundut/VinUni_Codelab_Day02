# 🏗️ Phase 3 — DEEP-DIVE: Xanh SM Customer Support Copilot

## Use case được chọn

**Problem Card #1 — Xử lý khiếu nại và yêu cầu hỗ trợ khách hàng Xanh SM.**

Mục tiêu là hỗ trợ nhân viên chăm sóc khách hàng kiểm tra thông tin, phân loại sự
cố và soạn phản hồi nhanh hơn. Hệ thống chỉ tạo đề xuất/bản nháp; nhân viên vẫn
chịu trách nhiệm kiểm tra và phê duyệt trước khi gửi.

## 3.1. Current-State Workflow Mapping (25 min)

```text
[1. Tiếp nhận khiếu nại]
    Khách hàng gọi tổng đài hoặc gửi yêu cầu trên ứng dụng
    ⏱ 1 phút
    🔄 Handoff: khách hàng → nhân viên CSKH
                         ↓
[2. Kiểm tra thông tin khách hàng] 🔴 BOTTLENECK
    Tra cứu mã chuyến, tài khoản, lịch sử thanh toán và sự cố
    ⏱ 2 phút
    🔄 Handoff: CSKH → hệ thống dữ liệu
                         ↓
[3. Phân loại sự cố] 🔴 BOTTLENECK
    Xác định nhóm vấn đề, mức độ ưu tiên và bộ phận xử lý
    ⏱ 1 phút
    🔄 Handoff: CSKH → bộ phận vận hành liên quan
                         ↓
[4. Phản hồi khách hàng]
    Soạn và gửi câu trả lời theo chính sách hỗ trợ
    ⏱ 1 phút
    🔄 Handoff: CSKH → khách hàng

Tổng cộng: khoảng 5 phút/lượt
```

### Bottleneck và rủi ro

- Bước 2 mất nhiều thời gian vì nhân viên phải tra cứu nhiều nguồn dữ liệu.
- Bước 3 có thể phân loại sai nếu nội dung khách hàng viết không rõ hoặc dùng
  từ ngữ đời thường.
- Phản hồi có thể chậm hoặc rập khuôn nếu nhân viên phải tự viết lại từ đầu.
- Số liệu thời gian là baseline giả định để scoping, cần xác nhận bằng log thực tế.

## 3.2. Problem Statement (6-field) & Metrics (15 min)

| Field | Nội dung |
|---|---|
| **1. Actor / Operator** | Nhân viên chăm sóc khách hàng Xanh SM tiếp nhận và xử lý yêu cầu; khách hàng cung cấp mã chuyến, nội dung khiếu nại và thông tin liên quan. |
| **2. Current Workflow** | Nhân viên nhận yêu cầu qua tổng đài/app, tra cứu mã chuyến và lịch sử khách hàng, phân loại sự cố, chuyển bộ phận liên quan rồi tự soạn phản hồi theo chính sách. |
| **3. Bottleneck** | Kiểm tra thông tin và phân loại sự cố mất khoảng 3 phút/lượt; nội dung tiếng Việt không theo mẫu làm tăng nguy cơ chuyển sai bộ phận. |
| **4. Business Impact** | Mỗi yêu cầu mất khoảng 5 phút xử lý, làm tăng thời gian chờ của khách hàng, tạo tải cho CSKH và có thể ảnh hưởng SLA phản hồi. |
| **5. Success Metric** | Giảm thời gian xử lý trung bình từ 5 phút xuống dưới 2 phút; ít nhất 85% yêu cầu được phân loại trong 30 giây; tỷ lệ chuyển sai bộ phận dưới 5%; 100% phản hồi do AI tạo có nhân viên duyệt. |
| **6. Operational Boundary** | AI được phép trích xuất thông tin, gợi ý nhóm sự cố, mức ưu tiên và tạo bản nháp phản hồi dựa trên chính sách đã cung cấp. AI không được tự hoàn tiền, tự nhận lỗi pháp lý, tự cam kết bồi thường, tự gửi phản hồi hoặc xử lý yêu cầu thiếu dữ liệu nhạy cảm. Các trường hợp tranh chấp, an toàn, tai nạn, thanh toán hoặc dữ liệu không chắc chắn phải chuyển nhân viên cấp cao review. |

## 3.3. Future-State Flow & AI Fit (25 min)

### AI-Fit Matrix

| Phương án | Vai trò | Đánh giá |
|---|---|---|
| **Rule / State-Machine** | Kiểm tra trường bắt buộc, nhóm sự cố cố định, mức ưu tiên và quyền thực hiện hành động | **Bắt buộc** làm lớp kiểm soát |
| **LLM Feature** | Đọc nội dung tự do bằng tiếng Việt, tóm tắt vụ việc, gợi ý phân loại và soạn bản nháp | **Phù hợp** với phạm vi prototype |
| **Agentic Loop** | Tự gọi hệ thống thanh toán, tự hoàn tiền hoặc tự gửi phản hồi | **Không chọn** vì có quyền tác động và rủi ro cao |

### Future-State Flow

```text
Yêu cầu từ app/tổng đài
        ↓
🔵 AI: trích xuất mã chuyến, vấn đề, thời gian và mong muốn của khách hàng
        ↓
Rule: kiểm tra trường bắt buộc và phát hiện nhóm rủi ro cao
        ├── Thiếu dữ liệu / tai nạn / tranh chấp / thanh toán
        │       ↓
        │  ↩️ Fallback: chuyển hàng đợi CSKH chuyên trách
        │       ↓
        │  🟢 HITL: nhân viên cấp cao xử lý thủ công
        │
        └── Yêu cầu thông thường
                ↓
           🔵 AI: gợi ý nhóm sự cố, mức ưu tiên và bộ phận nhận ticket
                ↓
           🔵 AI: tạo draft phản hồi theo policy được phê duyệt
                ↓
           🟢 HITL: nhân viên CSKH kiểm tra dữ liệu, tone và chính sách
                ↓
           Nhân viên duyệt và gửi qua hệ thống hiện có

↩️ Nếu JSON sai schema, model không chắc chắn, policy không phù hợp hoặc API
timeout: không gửi tự động; giữ ticket ở hàng đợi thủ công và ghi nhận lỗi để review.
```

### Operational boundary

1. Mọi phản hồi từ AI phải mang trạng thái **DRAFT_ONLY** cho đến khi nhân viên
   CSKH duyệt.
2. AI không được tự thực hiện giao dịch, hoàn tiền, bồi thường hoặc thay đổi
   trạng thái chuyến đi.
3. AI không được suy đoán thông tin khách hàng khi dữ liệu bị thiếu.
4. Các vụ việc liên quan đến tai nạn, an toàn, pháp lý, thanh toán hoặc khiếu nại
   nghiêm trọng phải được chuyển người có thẩm quyền.

## 🏁 Phase 5 — EVALUATE

### AI Readiness Checklist

- [ ] Có dữ liệu ticket đã ẩn thông tin cá nhân, gồm nội dung, nhóm xử lý, thời gian
  và kết quả cuối để đo baseline.
- [x] Rủi ro có thể kiểm soát bằng rule, human-in-the-loop và fallback thủ công.
- [ ] Nhân viên CSKH và quản lý đã xác nhận quy trình mới, policy và quyền phê duyệt.

### Quyết định: NOT YET

Chọn **NOT YET** cho triển khai thật, nhưng cho phép xây dựng prototype trong
môi trường sandbox. LLM phù hợp với việc đọc nội dung tiếng Việt, phân loại và
soạn bản nháp; tuy nhiên rule-based vẫn phải kiểm soát trường bắt buộc và các
nhóm rủi ro cao.

Nhóm chưa có log sạch để xác nhận baseline 5 phút/lượt, tỷ lệ phân loại sai và
SLA hiện tại. Trước khi pilot cần thu thập tối thiểu hai tuần dữ liệu đã ẩn danh,
đánh giá trên 100 ticket đại diện, xác nhận policy với CSKH và đo các metric:
thời gian xử lý dưới 2 phút, 85% phân loại trong 30 giây, chuyển sai dưới 5%.
Chỉ chuyển sang **GO** khi toàn bộ phản hồi vẫn có người duyệt và không có trường
hợp rủi ro cao bị tự động xử lý.
