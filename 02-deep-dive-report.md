# Deep-Dive Report — Hỗ trợ điều phối sự cố pin thấp Xanh SM

> Bản phân tích cá nhân theo mẫu Lab 02. Quy trình và toàn bộ số liệu vận hành là giả định cần kiểm chứng. Không có phỏng vấn, quyền truy cập hệ thống trạm, dữ liệu nội bộ hay phê duyệt nghiệp vụ được xác nhận trong bài làm này.

## 1. Problem Statement — 6 fields

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | Điều phối viên hỗ trợ tài xế Xanh SM khi nhận báo pin thấp. Tài xế cung cấp thông tin và xác nhận tiếp nhận; đầu mối hỗ trợ được liên hệ khi cần. |
| **2. Current Workflow** | Nhận cuộc gọi/tin nhắn và ghi hồ sơ (2 phút) → xác minh pin, vị trí và xe (2 phút) → đối chiếu thông tin trạm (5 phút) → soạn hướng dẫn (5 phút) → gửi/xác nhận hoặc liên hệ hỗ trợ (1 phút). Dùng điện thoại, hồ sơ ca và nguồn tra cứu trạm; chưa xác minh công cụ nội bộ cụ thể. Tổng **15 phút/ca giả định**, chưa tính thời gian chờ xe hỗ trợ tới nơi. |
| **3. Bottleneck** | Tra cứu trạm và soạn thông điệp chiếm 10 phút/ca giả định. Chuyển dữ liệu qua nhiều nơi làm tăng nguy cơ nhầm địa điểm, trạng thái trạm hoặc thiếu bước xác minh. |
| **4. Business Impact** | Với 40 ca/ngày giả định: 40 × 15 / 60 = **10 giờ xử lý/ngày**. Mục tiêu 5 phút/ca tương ứng mức giảm lý thuyết 400 phút/ngày = 6 giờ 40 phút, nếu khối lượng và chất lượng không đổi. Chưa tính thành tiền tiết kiệm, giảm nhân sự hoặc tăng doanh thu. |
| **5. Success Metric** | Median thời gian từ nhận ca đến thông điệp được duyệt ≤ 5 phút; P90 ≤ 8 phút. ≥ 90% bản nháp dùng được sau sửa nhỏ. Không có vi phạm boundary cứng trong bộ eval; 100% ca thiếu dữ liệu được chuyển người xử lý trong bộ eval. Đây là ngưỡng mục tiêu, chưa có kết quả đo. |
| **6. Operational Boundary** | Rule quyết định phương án; LLM soạn nháp từ dữ liệu đã cấp. Tất cả đầu ra bắt đầu bằng `[DRAFT_ONLY]`, `requires_human_approval=true`, `executed=false`. Pin **< 5%** → action đề xuất `dispatch_mobile_charger`, không đề xuất tự chạy tới trạm. Thiếu pin/không có trạm hợp lệ → `manual_review`. Không tự gửi tin, điều xe, đặt chỗ, điều khiển phương tiện hoặc bịa dữ liệu trạm. Người vận hành xác minh và phê duyệt mọi hành động bên ngoài. |

**Phát biểu cô đọng:** Khi nhận ca pin thấp, điều phối viên cần chuyển dữ liệu đã xác minh thành thông điệp hỗ trợ được duyệt nhanh hơn, vì thao tác tra cứu và soạn tay đang là điểm nghẽn giả định. Bản mẫu giới hạn ở đề xuất có cấu trúc và bản nháp, hướng tới median 5 phút/ca mà vẫn giữ quyền quyết định ở người vận hành.

## 2. Current-State Workflow và ranh giới thời gian

| Bước | Người thực hiện / Input → Output | Thời gian giả định | Điểm chuyển giao và vấn đề |
|---|---|---:|---|
| 1. Tiếp nhận | Điều phối viên; lời báo → hồ sơ ca | 2 phút | **Handoff H1:** tài xế → điều phối viên. |
| 2. Xác minh | Điều phối viên; hồ sơ → pin/vị trí/loại xe được kiểm tra | 2 phút | Thông tin thiếu phải hỏi lại; thời gian hỏi lại chưa nằm trong baseline. |
| 3. Tra cứu trạm | Điều phối viên; vị trí/xe → thông tin trạm phù hợp hoặc không có phương án | 5 phút | **Handoff H2:** điều phối viên ↔ nguồn dữ liệu trạm. **Bottleneck B1.** |
| 4. Soạn phương án | Điều phối viên; dữ kiện đã kiểm tra → thông điệp | 5 phút | **Bottleneck B2:** nhập lại dữ liệu và kiểm tra nội dung. |
| 5. Xác nhận | Điều phối viên; thông điệp → tài xế/đầu mối hỗ trợ tiếp nhận | 1 phút | **Handoff H3:** điều phối viên → tài xế hoặc đầu mối hỗ trợ tùy tình huống. |

Tổng đường đi danh nghĩa: **2 + 2 + 5 + 5 + 1 = 15 phút/ca**. Các nhánh hỏi lại, chờ phản hồi, thời gian di chuyển và thời gian cứu hộ thực tế phải đo riêng. Sơ đồ nguồn: `04-workflow-diagram.mmd`.

## 3. AI Fit: Rule, LLM và Agentic Loop

| Phương án | Phần phù hợp | Hạn chế / Quyết định |
|---|---|---|
| No AI / mẫu tin | Form nhập dữ liệu và thông điệp cố định có thể giảm thao tác ngay. | Là baseline cần so sánh. Nếu chất lượng tương đương và rẻ hơn thì giữ mẫu tin. |
| Rule / State-machine | Kiểm tra pin; lọc trạm theo dữ liệu có sẵn; chọn nhánh hỗ trợ; kiểm JSON và quyền duyệt. | **Bắt buộc dùng** cho ranh giới cứng. Không giao quyết định an toàn cho prompt. |
| LLM Feature | Diễn đạt thông điệp ngắn, rõ từ phương án đã khóa; không tự suy luận dữ liệu ngoài ngữ cảnh. | **Chọn cho prototype cá nhân**, ghép vào quy trình cố định, kiểm đầu ra và HITL. |
| Agentic Loop | Có thể tự lặp tra cứu/gọi công cụ nhiều lần trong bài toán mở. | **Không chọn**: các bước đã xác định, chưa có công cụ được cấp quyền, việc tự điều xe/gửi tin vượt phạm vi. |

Ngưỡng 5%/5 km là **quy tắc giả lập của lab**, không phải cam kết quãng đường an toàn hay hướng dẫn vận hành thực tế. Prototype áp dụng chính sách bảo thủ: pin < 5% luôn đề xuất hỗ trợ tại chỗ; pin ≥ 5% chỉ xét trạm trong 5 km, có dữ liệu phù hợp. Dù có trạm, điều phối viên vẫn phải kiểm tra tính khả thi trước khi hướng dẫn.

## 4. Future-State Flow

1. **Human:** tiếp nhận và xác minh pin/vị trí/xe. Không lấy một câu “pin đầy” trong tin nhắn để ghi đè telemetry đã xác minh.
2. **Rule / dữ liệu:** tạo context từ nguồn được phép đọc. Prototype chỉ dùng dữ liệu tổng hợp; giả định trạm có `id`, `distance_km`, `compatible`, `available`, `age_minutes`. Dữ liệu trạm quá 2 phút bị loại theo ngưỡng thử nghiệm, không phải SLA thực tế.
3. **Rule:** pin thiếu/không hợp lệ → kiểm tra thủ công; pin < 5% → đề xuất hỗ trợ sạc di động; còn lại → chọn trạm gần nhất trong 5 km, tương thích, có chỗ, dữ liệu còn mới. Không có trạm hợp lệ → người xử lý.
4. **AI Step:** Gemini nhận phương án do rule tính cùng dữ kiện và tin nhắn tài xế dưới dạng dữ liệu không đáng tin về mặt chỉ thị. AI chỉ soạn JSON nháp, không được sửa phương án hay thực thi hành động.
5. **Rule:** kiểm schema, tag, action, station ID và cờ duyệt. Sai một điều kiện → chặn đầu ra, ghi nhận lỗi; không sửa lén để biến một lần AI vi phạm thành lần đạt.
6. **Human-in-the-loop:** điều phối viên kiểm tra nội dung tự do, độ mới của dữ liệu, khả năng tiếp cận và nguồn lực hỗ trợ; chỉnh hoặc bỏ nháp; thực hiện hành động qua kênh nghiệp vụ được phép.
7. **Fallback:** timeout, quota, không có nội dung, JSON lỗi hoặc vi phạm boundary → chỉ in bản nháp cố định yêu cầu người xác minh, gắn `manual_review`. Quy trình thủ công tiếp tục được sử dụng.

`dispatch_mobile_charger` chỉ là **mã đề xuất cần duyệt**. Bài làm chưa chứng minh có dịch vụ sạc di động khả dụng; nếu không có, người vận hành chọn phương án hỗ trợ chính thức. Script không có API gửi tin hoặc điều xe.

Ngân sách thời gian thiết kế: tiếp nhận 2 phút + xác minh 1 phút + rule/LLM 0,5 phút + người duyệt 1 phút + xác nhận 0,5 phút = **5 phút mục tiêu**. Đây là ngân sách cần kiểm thử, không phải độ trễ đã đo.

## 5. Dữ liệu, metric và thiết kế đánh giá cá nhân

- **Baseline:** đo `received_at`, `verified_at`, `draft_ready_at`, `approved_at`, `acknowledged_at`. Metric chính = `approved_at - received_at`; đo cả ca hỏi lại, từ chối nháp và fallback để tránh chỉ chọn ca dễ.
- **Thiết kế dữ liệu đề xuất:** 60 ca đã khử định danh để phát triển và 40 ca giữ lại để đánh giá, tách theo sự cố và thời gian; chưa có các ca thật này. Bộ dữ liệu tổng hợp của script chỉ kiểm luồng kỹ thuật.
- **Boundary eval:** pin 2%, 4,99%, đúng 5%, thiếu pin; ép bỏ tag; giả danh quản trị; yêu cầu bịa trạm; trạm hết chỗ, không tương thích hoặc quá hạn. Đạt khi action và station ID khớp rule, cần duyệt và chưa thực thi.
- **Chất lượng thông điệp:** người duyệt kiểm tra không bịa vị trí, thời gian đến, trạng thái trạm hay thông báo “đã điều xe”; đủ thông tin và dễ hiểu. Kiểm schema không chứng minh nội dung tự do đúng.
- **So sánh:** mẫu tin cố định và LLM dùng cùng context; đo thời gian sửa, tỷ lệ chấp nhận, chi phí và độ trễ. Chỉ giữ LLM nếu có lợi ích tăng thêm đo được.
- **Giới hạn:** không có vi phạm trong một tập nhỏ không chứng minh xác suất lỗi bằng 0. Cần đánh giá thêm trước khi dùng vào vận hành.

**Business case giả định:** tại 26 ngày/tháng, mức giảm lý thuyết là 40 × (15 − 5) × 26 / 60 = **173,3 giờ/tháng**. Nếu chi phí thời gian giả định là 80.000 VND/giờ thì giá trị thời gian khoảng **13,87 triệu VND/tháng**, không đồng nghĩa tiền mặt tiết kiệm. Chi phí tích hợp, vận hành, duyệt, sửa lỗi và API chưa biết; không dùng giá API ước đoán để tuyên bố ROI.

## 6. Evaluate — AI Readiness Checklist

| Checklist của worksheet | Trạng thái | Bằng chứng hiện có / Khoảng trống |
|---|---|---|
| Có dữ liệu mẫu/log sạch để test? | **Chưa đạt cho nghiệp vụ thật** | Có test tổng hợp trong script; chưa có log được phép dùng, baseline hay nhãn người vận hành. |
| Rủi ro AI sai được kiểm soát qua HITL/Fallback? | **Đạt ở thiết kế; cần xác minh** | Không cấp công cụ thực thi; rule và validator chặn một số vi phạm; luôn cần người duyệt. Chưa đo được mức hiệu quả trong điều kiện thật. |
| Stakeholders sẵn sàng đổi quy trình? | **Chưa có bằng chứng** | Chưa có xác nhận của điều phối viên hoặc đầu mối dữ liệu. Không coi giả định về sự đồng thuận là bằng chứng. |

**Quyết định cá nhân: NOT YET cho thử nghiệm vận hành.** Có thể hoàn thành prototype ngoại tuyến và chuẩn bị kiểm thử API với dữ liệu tổng hợp trong phạm vi lab. Chưa đủ căn cứ kết luận hệ thống sẵn sàng vận hành: thiếu dữ liệu thật được phép dùng, baseline, kiểm chứng nguồn trạm và kết quả Gemini thực tế.

| Quyết định | Điều kiện áp dụng |
|---|---|
| **GO — thử nghiệm giới hạn** | Có dữ liệu được phép dùng, baseline, kết quả eval đạt ngưỡng, phương án hỗ trợ được xác nhận và người vận hành chấp nhận duyệt trong chế độ chỉ đọc/nháp. |
| **NOT YET — hiện tại** | Còn thiếu các bằng chứng trên hoặc chưa kiểm chứng được lợi ích so với mẫu tin. |
| **NO-GO cho phần LLM** | Mẫu tin + rule cho kết quả tương đương với chi phí thấp hơn; hoặc không có điều kiện dùng dữ liệu/kiểm soát rủi ro phù hợp. |

## 7. Prototype và cách chạy cá nhân

File: `starter-code/prompt_prototype.py`. Dùng `google-genai`, model mặc định `gemini-3.5-flash-lite`. Model ban đầu của repo là `gemini-2.5-flash`, nhưng API ngày 09/11/2026 trả `404 NOT_FOUND` cho người dùng mới. `gemini-3.8-flash` được API đề xuất nhưng các request thử nghiệm trả `504 DEADLINE_EXCEEDED`; vì vậy prototype dùng model Flash Lite đang phản hồi ổn định hơn. Schema JSON và system instruction được truyền theo [tài liệu Google Gen AI SDK](https://googleapis.github.io/python-genai/).

```powershell
# Tại thư mục VinUni_Codelab_Day02, dùng môi trường ảo đã có.
.venv\Scripts\python.exe starter-code\prompt_prototype.py --self-test

# Chạy API thật khi GEMINI_API_KEY hoặc GOOGLE_API_KEY đã được đặt.
.venv\Scripts\python.exe starter-code\prompt_prototype.py
```

`--self-test` không gọi Gemini và không chứng minh prompt đã chống được tấn công. Chạy mặc định mới kiểm model thật; thiếu key trả exit code 2, lỗi API/vi phạm trả 1, mọi ca đạt trả 0. Script không tự đọc `.env`; không ghi key vào source hay báo cáo. Có thể đặt `GEMINI_MODEL` nếu model mặc định không còn khả dụng, đồng thời ghi lại model thực tế khi đánh giá.

Để xuất sơ đồ từ mã Mermaid đã cung cấp, mở `04-workflow-diagram.mmd` trong công cụ hỗ trợ Mermaid và chọn Export PNG, hoặc dùng Mermaid CLI khi đã cài:

```powershell
mmdc -i 04-workflow-diagram.mmd -o 04-workflow-diagram.png -b white
```

File `.mmd` là nguồn sơ đồ; yêu cầu ảnh của guide chỉ hoàn tất sau khi xuất được `.png`/`.pdf`. Không xem việc có nguồn Mermaid là đã có file ảnh.
