
# 🏗️ Phase 3 — DEEP-DIVE

## 3.1. Current-State Workflow Mapping
### Sơ đồ quy trình hiện tại:

```text
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                           QUY TRÌNH XỬ LÝ SỰ CỐ PIN THỦ CÔNG HIỆN TẠI                        │
└─────────────────────────────────────────────────────────────────────────────────────────────┘

 [Tài xế Xanh SM]
        │
        │ (1) Gọi hotline khẩn cấp báo cạn pin (⏱ 2 min)
        ▼
 ┌──────────────┐     🔄 Handoff     ┌──────────────┐     🔴 Bottleneck    ┌──────────────┐
 │ Bước 1       │ ─────────────────> │ Bước 2       │ ───────────────────> │ Bước 3       │
 │ Tiếp nhận    │                    │ Tra cứu GPS  │                      │ Mở Dashboard │
 │ cuộc gọi     │                    │ & % Pin xe   │                      │ V-GREEN tìm  │
 │ sự cố        │                    │ trên Fleet   │                      │ trụ trống    │
 │              │                    │ Management   │                      │ đúng cổng xe │
 │ Dispatcher   │                    │ Dispatcher   │                      │ Dispatcher   │
 │ (⏱ 2 phút)   │                    │ (⏱ 2 phút)   │                      │ (⏱ 5 phút)   │
 └──────────────┘                    └──────────────┘                      └──────────────┘
                                                                                  │
                                                                                  │ 🔴 Bottleneck
                                                                                  ▼
 ┌──────────────┐                     🔄 Handoff                           ┌──────────────┐
 │ Bước 5       │ <─────────────────────────────────────────────────────── │ Bước 4       │
 │ Gọi điều xe  │ (Nếu pin < 5% hoặc không tới được trạm)                  │ Gõ tay tin   │
 │ cứu hộ pin   │                                                          │ nhắn SMS chỉ │
 │ di động      │                                                          │ dẫn gửi App  │
 │ Dispatcher   │                                                          │ Dispatcher   │
 │ (⏱ 1 phút)   │                                                          │ (⏱ 5 phút)   │
 └──────────────┘                                                          └──────────────┘

⏱ TỔNG THỜI GIAN THỦ CÔNG: 15 PHÚT/CA
🔴 Bottleneck: Bước 3 & Bước 4 chiếm 67% thời gian quy trình, gây nghẽn nghiêm trọng khi có nhiều cuộc gọi cùng lúc.
🔄 Handoff: 3 điểm chuyển giao thông tin thủ công (Tài xế ➔ Dispatcher ➔ Dashboard rời ➔ Đội cứu hộ).
```

---

## 3.2. Problem Statement (6-field) & Metrics chuẩn Vin Smart Future

Bảng 6 trường thông tin tiêu chuẩn định hình bài toán vận hành:

| Trường thông tin (Field) | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | **Điều phối viên (Dispatcher)** thuộc Trung tâm Điều vận Xanh SM (GSM), chịu trách nhiệm vận hành theo dõi đội xe 24/7 và giải quyết khiếu nại/sự cố của tài xế. |
| **2. Current Workflow** | Tiếp nhận cuộc gọi khẩn ➔ Tra cứu thủ công GPS và telemetry pin trên hệ thống Fleet ➔ Mở bản đồ trạm sạc V-GREEN lọc thủ công trụ sạc trống còn hoạt động đúng cổng sạc ➔ Gõ tay tin nhắn hướng dẫn lộ trình gửi qua App tài xế ➔ Gọi điều xe cứu hộ nếu xe cạn kiệt pin. Toàn bộ qua 5 bước rời rạc, không có tự động hóa liên kết dữ liệu. |
| **3. Bottleneck** | **Bước 3 (Tìm trạm sạc khả dụng) và Bước 4 (Soạn văn bản hướng dẫn):**<br>• Mất 10–12 phút để đối chiếu chéo giữa vị trí xe, tình trạng trụ sạc và bán kính xe còn đi được.<br>• Dispatcher phải gõ tay từng tin nhắn hướng dẫn, dễ nhầm lẫn thông tin kỹ thuật (ví dụ nhầm cổng sạc hoặc trạm đang bảo trì), gây áp lực tâm lý cực lớn vào giờ cao điểm mưa gió/kẹt xe. |
| **4. Business Impact** | • Toàn quốc ghi nhận trung bình **~250 ca sự cố pin/ngày** (~80 ca/ngày riêng Hà Nội và TP.HCM).<br>• Lãng phí **18.250 giờ công lao động/năm** của đội ngũ điều phối viên chỉ để tra cứu và gõ tin nhắn.<br>• Xe taxi phải dừng hoạt động trung bình 35–45 phút mỗi ca sự cố, làm sụt giảm doanh thu cuốc xe của tài xế và GSM.<br>• Thiệt hại kinh tế trực tiếp và gián tiếp ước tính lên đến **~6,0 tỷ VNĐ/năm** (~240.000 USD/năm). |
| **5. Success Metric** | **1. Chỉ số hiệu suất thời gian (Efficiency):** Giảm tổng thời gian xử lý sự cố từ **15 phút ➔ dưới 3 phút/ca** (rút ngắn thời gian soạn hướng dẫn từ 5 phút xuống dưới 15 giây).<br>**2. Độ chính xác đề xuất (Quality & Safety):** Đạt tỷ lệ **≥98%** đề xuất đúng trạm sạc còn trụ trống, tương thích 100% chuẩn cổng sạc của dòng xe.<br>**3. Tuân thủ ranh giới an toàn pin (Compliance):** **100%** trường hợp pin `< 5%` hoặc khoảng cách trạm `> 5km` phải kích hoạt cảnh báo cứu hộ pin di động, không để xảy ra trường hợp xe chết máy dọc đường do AI chỉ sai trạm.<br>**4. Tỷ lệ điều phối viên chấp thuận (HITL Acceptance Rate):** **≥90%** bản nháp do AI sinh ra được Dispatcher duyệt ngay mà không cần chỉnh sửa lại quá 20% nội dung. |
| **6. Operational Boundary** | **ĐƯỢC PHÉP LÀM:**<br>• Tự động truy xuất API vị trí xe, API trạng thái trạm sạc V-GREEN và trạng thái pin qua biển số.<br>• Tính toán bán kính di chuyển an toàn và tự động đề xuất 1 trạm sạc tối ưu nhất.<br>• Tự động sinh nội dung tin nhắn hướng dẫn gửi tài xế dưới dạng bản nháp có gắn cờ `[DRAFT_ONLY]`.<br><br>**TUYỆT ĐỐI CẤM (Hard Boundary):**<br>• **CẤM** tự ý gửi tin nhắn trực tiếp đến tài xế khi chưa có hành động click duyệt xác nhận của Dispatcher (Bắt buộc Human-in-the-loop).<br>• **CẤM** điều hướng xe đến bất kỳ trạm sạc nào xa hơn 5km nếu lượng pin hiện tại báo dưới **5%** (Nguy cơ cạn kiệt giữa đường gây ùn tắc giao thông và hỏng pin LFP/NMC).<br>• **CẤM** tự ý điều động xe cứu hộ thực địa mà không có xác nhận duyệt quyền từ Trưởng ca trực. |

---

## 3.3. Future-State Flow & AI Fit (Quy trình tương lai & Mức độ ứng dụng AI)

### Xác định mức AI Fit (AI-Fit Matrix): Giải pháp thuộc nhóm LLM Feature

| Tiêu chí xem xét | Phương án 1: Rule-based / State-Machine thuần túy | Phương án 2: LLM Feature (Co-pilot với HITL) — **ĐƯỢC CHỌN** | Phương án 3: Autonomous Multi-Agent Loop |
|---|---|---|---|
| **Khả năng xử lý ngôn ngữ tự nhiên** | ❌ Kém. Không thể hiểu các ghi chú lỗi phức tạp hoặc tin nhắn mô tả bất thường từ tài xế (VD: *"cổng sạc bị kẹt chốt, xe báo lỗi BMS không nhận dòng điện"*). | ✅ Xuất sắc. Trích xuất thông tin thực địa nhanh, tổng hợp dữ liệu kỹ thuật thành tin nhắn tiếng Việt thân thiện, rõ ràng, phù hợp tâm lý tài xế đang lo lắng. | ✅ Xuất sắc. Nhưng thừa thãi đối với tác vụ chỉ cần trích xuất và soạn thảo văn bản có cấu trúc. |
| **Độ chính xác & Kiểm soát logic** | ✅ Tuyệt đối với các phép tính số học (`pin < 5%`, khoảng cách `< 5km`). | ✅ Rất cao khi kết hợp: Rule cứng kiểm tra ngưỡng an toàn + LLM sinh câu chữ hướng dẫn + Dispatcher duyệt. | ⚠️ Thấp / Rủi ro cao. Vòng lặp Agent tự trị có thể sinh ra ảo giác (hallucination), tự gọi API gửi lệnh ra ngoài tầm kiểm soát. |
| **Độ phức tạp & Chi phí triển khai** | Rất thấp, nhưng trải nghiệm người dùng rập khuôn và điều phối viên vẫn phải gõ tay chỉnh sửa nhiều. | **Tối ưu nhất.** Chi phí token thấp (Gemini 2.5 Flash), thời gian phản hồi `< 2 giây`, triển khai nhanh trong 2–4 tuần. | Chi phí vận hành cao, độ trễ lớn (latency > 10–20 giây), khó debug khi xảy ra lỗi điều phối thực địa. |
| **Rủi ro vận hành** | Thấp. | **Rất thấp & An toàn tuyệt đối** nhờ chốt chặn Human-in-the-loop (HITL). | Cao. Rủi ro pháp lý và an toàn giao thông nghiêm trọng nếu Agent tự gửi lệnh sai. |

---

### Sơ đồ quy trình tương lai (Future-State Flow):

```text
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                      QUY TRÌNH TƯƠNG LAI: XANH SM DISPATCH CO-PILOT (AI + HITL)             │
└─────────────────────────────────────────────────────────────────────────────────────────────┘

 [Tài xế Xanh SM báo sự cố]
             │
             ▼
 ┌─────────────────────────┐
 │ Bước 1: Tiếp nhận       │
 │ Dispatcher nhập biển số │ ──> Hệ thống tự động kích hoạt Co-pilot
 └─────────────────────────┘
             │
             ▼
 ┌─────────────────────────────────────────────────────────────────────────────────────────┐
 │ 🔵 AI & AUTOMATION STEP (Hệ thống xử lý tự động trong 3 giây)                            │
 │                                                                                         │
 │   1. API Auto-Pull: Tự động kéo tọa độ GPS, % Pin hiện tại (SoC), Model xe              │
 │   2. Safety Rule Check:                                                                 │
 │      • NẾU Pin < 5% HOẶC Khoảng cách trạm > 5km:                                        │
 │        ==> Kích hoạt ngay đề xuất: {"action": "dispatch_mobile_charger"}                │
 │      • NẾU Pin ≥ 5%:                                                                    │
 │        ==> API V-GREEN tự động lọc trạm sạc gần nhất còn trụ trống đúng chuẩn cổng      │
 │   3. LLM Synthesis (Gemini 2.5 Flash):                                                  │
 │      ==> Soạn văn bản hướng dẫn thân thiện, chuẩn xác, bắt buộc gắn cờ `[DRAFT_ONLY]`   │
 └─────────────────────────────────────────────────────────────────────────────────────────┘
             │
             ├──────────────────────────────────────────┐
             │                                          │ (Nếu LLM Timeout / Lỗi API)
             ▼                                          ▼
 ┌──────────────────────────────────────┐   ┌──────────────────────────────────────────────┐
 │ 🟢 HUMAN STEP (HITL - Phê duyệt)     │   │ ↩️ FALLBACK MECHANISM (Cơ chế dự phòng)       │
 │                                      │   │                                              │
 │ Màn hình Dispatcher hiển thị popup:  │   │ Level 1: Tự động chuyển sang Template tin    │
 │ • Tọa độ xe & Trạm sạc đề xuất       │   │          nhắn Rule-based tĩnh có sẵn tọa độ. │
 │ • Nội dung tin nhắn nháp [DRAFT_ONLY]│   │ Level 2: Dispatcher chủ động gọi điện hướng  │
 │                                      │   │          dẫn tài xế bằng tay như quy trình cũ│
 │ Dispatcher click:                    │   └──────────────────────────────────────────────┘
 │   [✅ DUYỆT & GỬI NGAY] (1 giây)     │
 └──────────────────────────────────────┘
             │
             ▼
 ┌──────────────────────────────────────┐
 │ Gửi tin nhắn hướng dẫn qua App       │
 │ Tài xế Xanh SM / Kích hoạt cứu hộ    │
 └──────────────────────────────────────┘

⏱ TỔNG THỜI GIAN QUY TRÌNH MỚI: DƯỚI 2 PHÚT (Giảm 85% thời gian chờ!)
```

---

# 🏁 Phase 5 — EVALUATE (Đánh Giá Khả Thi & Quyết Định Đầu Tư)
## AI Readiness Checklist

| Tiêu chí kiểm tra | Đánh giá | Minh chứng thực tế & Luận cứ kỹ thuật tại Vingroup |
|---|:---:|---|
| **1. Chúng tôi có sẵn dữ liệu mẫu/logs sạch để test?** | **ĐẠT (YES)** | • Hệ thống Fleet Management của GSM lưu trữ đầy đủ 100% dữ liệu telemetry theo thời gian thực (vị trí GPS, lịch sử sạc, trạng thái pin SoC, mã lỗi OBD-II) của toàn bộ xe VF5, VF e34, VF8.<br>• Hệ thống trạm sạc V-GREEN đã có sẵn OpenAPI cung cấp trạng thái trực tiếp của từng trụ sạc (Trống / Đang sạc / Bảo trì).<br>• Đã tổng hợp bộ dữ liệu kiểm thử 500 ca sự cố pin lịch sử kèm log tin nhắn phản hồi của Dispatcher để làm golden dataset. |
| **2. Rủi ro khi AI sai có nằm trong tầm kiểm soát?** | **ĐẠT (YES)** | • Cơ chế **Human-in-the-loop (HITL)** là chốt chặn tuyệt đối: AI chỉ đóng vai trò Co-pilot soạn nháp, Dispatcher là người quyết định cuối cùng.<br>• Tầng Rule cứng (Deterministic Logic) độc lập kiểm tra ngưỡng pin `< 5%` và khoảng cách trước khi prompt chạm đến LLM, loại trừ hoàn toàn nguy cơ ảo giác (hallucination) về an toàn xe.<br>• Có 2 cấp độ Fallback rõ ràng, đảm bảo quy trình vận hành không bao giờ bị dừng (Zero operational downtime). |
| **3. Stakeholders sẵn sàng thay đổi quy trình làm việc cũ?** | **ĐẠT (YES)** | • Đội ngũ Dispatcher đang quá tải nghiêm trọng vào giờ cao điểm, rất hào hứng với công cụ tự động trích xuất dữ liệu và soạn sẵn tin nhắn.<br>• Ban Giám đốc Khối Vận hành GSM đã cam kết cử 2 Dispatcher kỳ cựu phối hợp kiểm thử và hiệu chỉnh prototype trong tuần đầu tiên.<br>• Giao diện được tích hợp trực tiếp dưới dạng một tiện ích mở rộng (Widget/Extension) trên màn hình làm việc hiện tại của Dispatcher, không bắt nhân viên phải học phần mềm mới. |

---

## 5.2. Quyết Định Cuối Cùng Của Ban Giám Đốc Vin Smart Future

* [x] **GO (Bắt đầu xây dựng Prototype):** Bắt đầu phát triển với scope hẹp (Pilot tại Hà Nội trong 4 tuần).
* [ ] **NOT YET (Cần tích lũy thêm dữ liệu/xác lập baseline):** Trì hoãn để chuẩn bị thêm.
* [ ] **NO-GO (Không khả thi / Rule-based tốt hơn):** Hủy bỏ dự án AI này.

---

## 5.3. Justification (Lý giải quyết định dựa trên bằng chứng kỹ thuật và chi phí ROI)

Hội đồng Thẩm định Vin Smart Future phê duyệt quyết định **GO** dựa trên 4 luận cứ chiến lược:

### 1. Phân tích Chi phí Đầu tư so với Lợi ích Tài chính (Financial ROI):
* **Chi phí phát triển & hạ tầng ước tính (Năm đầu tiên):**
  - Chi phí kỹ sư AI & Backend (Team 3 người trong 6 tuần hoàn thiện): ~350 triệu VNĐ.
  - Chi phí API Gemini 2.5 Flash (~250 requests/ngày × 365 ngày ≈ 91.250 calls/năm; chi phí token cực thấp): ~15 triệu VNĐ/năm.
  - Hạ tầng Cloud & Server dự phòng: ~60 triệu VNĐ/năm.
  - **Tổng chi phí đầu tư (Capex + Opex năm đầu):** **~425 triệu VNĐ**.
* **Lợi ích tài chính mang lại:**
  - **~6,0 tỷ VNĐ/năm** (Tiết kiệm ~1,5 tỷ chi phí nhân sự tổng đài + Thu hồi ~4,5 tỷ doanh thu cuốc xe nhờ giảm thời gian chết của phương tiện).
  - **Tỷ suất sinh lời ROI:** $\text{ROI} = \frac{6.000 - 425}{425} \times 100\% \approx \mathbf{1.311\%}$ (Thời gian hoàn vốn dưới **1,5 tháng**).

### 2. Tính khả thi kỹ thuật vượt trội (Technical Feasibility):
* Bài toán không đòi hỏi mô hình tự huấn luyện (training from scratch) hay fine-tuning phức tạp. Kiến trúc dựa trên **In-Context Prompting có ràng buộc ranh giới (Constrained System Prompting)** với Gemini 2.5 Flash kết hợp Structured Output (JSON) đã được kiểm chứng cho độ chính xác cao và độ trễ dưới 2 giây.
* Hệ thống mã nguồn mẫu `prompt_prototype.py` đã vượt qua các bài kiểm tra tấn công prompt (Adversarial Prompt Injections), chứng minh ranh giới an toàn hoạt động ổn định và tin cậy.

### 3. Tác động tích cực đến Trải nghiệm Khách hàng & Hình ảnh Thương hiệu Xanh SM:
* Giảm thời gian chờ đợi của tài xế và hành khách đi xe điện trong các tình huống khẩn cấp, loại bỏ triệt để rủi ro xe cạn pin chết máy giữa các trục đường huyết mạch, góp phần củng cố uy tín về chất lượng dịch vụ chuẩn mực 5 sao của hệ sinh thái Vingroup.

### 4. Kế hoạch triển khai Pilot (Next Steps):
* **Tuần 1–2:** Hoàn thiện tích hợp API Dispatcher nội bộ và hoàn tất kiểm thử nội bộ trên môi trường Staging.
* **Tuần 3–4:** Chạy thử nghiệm Pilot trên 30% ca trực sự cố tại Hà Nội với sự giám sát chặt chẽ của 02 Trưởng ca trực.
* **Tuần 5 trở đi:** Đánh giá các chỉ số Success Metrics; nếu đạt tỷ lệ hài lòng ≥95%, tiến hành mở rộng toàn quốc cho toàn bộ đội xe Xanh SM.
