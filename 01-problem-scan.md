# Problem Scan & Quick Problem Cards — Bài làm cá nhân

## 1. Phạm vi và tình trạng bằng chứng

Bài làm sử dụng bối cảnh doanh nghiệp trong `README.md`, `01-worksheet.md` và `03-inspiration-kit.md` của Lab 02. Đây là các bài toán vận hành đề xuất để khảo sát, không phải kết luận từ phỏng vấn hay dữ liệu nội bộ. Mọi khối lượng, thời gian, tỷ lệ và chi phí dưới đây là **giả định phục vụ scoping**; mục tiêu là ngưỡng thử nghiệm đề xuất, chưa phải kết quả đã đạt. Tên công ty được dùng theo bối cảnh đề bài, không khẳng định cơ cấu sở hữu hiện hành.

## 2. Bảng quét cơ hội — SCAN

Sử dụng đủ bốn lenses: Repetitive, Time-consuming, AI-upgrade và Stakeholder Pain.

| ID | Công ty theo đề bài | Lens | Actor và bài toán cụ thể | Quy trình/bottleneck cần khảo sát | Giả định định lượng và hướng can thiệp |
|---|---|---|---|---|---|
| P1 | Xanh SM | Time-consuming; Stakeholder Pain | Điều phối viên mất thời gian hỗ trợ tài xế báo pin thấp ngoài đường. | Nhận thông tin → xác minh pin/vị trí → tìm trạm phù hợp → soạn hướng dẫn → xác nhận phương án hỗ trợ. Nghẽn ở tra cứu và soạn thông điệp. | 40 ca/ngày × 15 phút = 600 phút/ngày. Rule quyết định phương án; LLM chỉ soạn nháp; người duyệt. |
| P2 | Vinhomes | Repetitive; Time-consuming | Nhân viên ban quản lý đọc và chuyển ticket phản ánh cư dân nhiều lần khi nội dung thiếu rõ ràng. | Nhận ticket → đọc/xác minh → phân loại → gán bộ phận → xác nhận tiếp nhận. Nghẽn ở hiểu mô tả và xác định nơi nhận. | 150 ticket/ngày × 5 phút = 750 phút/ngày. Thử LLM phân loại có nhãn chuẩn, rule cho tín hiệu khẩn cấp, người duyệt nơi nhận. |
| P3 | Vinpearl | AI-upgrade; Time-consuming | Nhân viên đặt phòng phải đọc email đoàn dài, tách ngày lưu trú, số phòng và yêu cầu đặc biệt trước khi kiểm tra chỗ. | Nhận email → trích trường → hỏi thiếu dữ liệu → tra quỹ phòng → soạn trả lời. Nghẽn ở đọc email và nhập lại dữ liệu. | 25 email/ngày × 12 phút = 300 phút/ngày. LLM trích xuất có dẫn chứng; rule kiểm tra ngày/số lượng; nhân viên kiểm tra giá và tồn phòng. |
| P4 | VinFast | Repetitive | Nhân viên kế toán rà soát chênh lệch giữa dữ liệu phiên sạc và hóa đơn đối tác. | Nhận hai bảng → chuẩn hóa mã → đối soát → tra chênh lệch → lập danh sách xử lý. Nghẽn ở dữ liệu thiếu mã hoặc khác định dạng. | 300 dòng/ngày × 1 phút = 300 phút/ngày nếu rà từng dòng. Ưu tiên SQL/rule; chỉ cân nhắc LLM cho ghi chú phi cấu trúc. |
| P5 | Vinmec | Stakeholder Pain; Repetitive | Nhân viên tiếp đón trả lời lặp lại câu hỏi hành chính trước lịch hẹn. | Nhận câu hỏi → nhận diện cơ sở/dịch vụ → tra hướng dẫn → trả lời → chuyển chuyên viên khi vượt phạm vi. | 100 lượt/ngày × 4 phút = 400 phút/ngày. Bắt đầu bằng FAQ/rule; chỉ dùng LLM để diễn đạt từ tài liệu đã duyệt. Loại trừ tư vấn/chẩn đoán y khoa. |

Các tích trên là thời gian xử lý giả định, không phải thời gian tiết kiệm chắc chắn. Không cộng các công ty thành một business case chung.

## 3. Quick Problem Card #1 — P1: Hỗ trợ sự cố pin thấp Xanh SM

**Bài toán một câu:** Khi tài xế báo pin thấp, điều phối viên cần nhanh chóng xác minh dữ liệu và chuẩn bị phương án hỗ trợ phù hợp để tránh kéo dài thời gian chờ và gửi hướng dẫn thiếu căn cứ.

| Trường | Nội dung |
|---|---|
| Công ty / Actor | Xanh SM; điều phối viên là người vận hành, tài xế là người chịu ảnh hưởng trực tiếp. |
| Trigger / Input | Cuộc gọi hoặc tin nhắn báo pin thấp; cần pin hiện tại, thời điểm đo, vị trí, loại xe và thông tin trạm đã xác minh. |
| Current workflow — 5 bước | (1) Tiếp nhận, lập hồ sơ: 2 phút → (2) xác minh pin/vị trí: 2 phút → (3) tra cứu và đối chiếu trạm: 5 phút → (4) soạn hướng dẫn: 5 phút → (5) gửi thông tin đã kiểm tra và xác nhận tiếp nhận hoặc liên hệ hỗ trợ: 1 phút. |
| Bottleneck | Bước 3–4 mất 10/15 phút giả định; phải chuyển qua nhiều màn hình và viết lại thông tin từ dữ liệu tra cứu. |
| Handoff | Tài xế → điều phối viên; điều phối viên ↔ nguồn thông tin trạm; điều phối viên → tài xế/đầu mối hỗ trợ. |
| Business impact giả định | 40 ca/ngày × 15 phút = 10 giờ xử lý/ngày. Chưa có dữ liệu để quy đổi thành doanh thu mất đi hoặc tỷ lệ giảm hủy chuyến. |
| AI hỗ trợ ở đâu? | Bước 4: soạn nháp từ phương án đã được rule xác định và dữ liệu được cấp. Không dùng LLM để tính quãng đường hay suy đoán trạm còn trống. |
| Success metric đề xuất | Median thời gian từ nhận ca tới thông điệp được duyệt ≤ 5 phút; P90 ≤ 8 phút. Không có vi phạm cứng trong bộ eval; ≥ 90% bản nháp được người duyệt chấp nhận chỉ với sửa nhỏ. |
| Quick Architecture | **Rule + LLM Feature + HITL**; không cần Agentic Loop. |
| Boundary / Fallback | Mọi đầu ra là nháp. Pin < 5% → đề xuất hỗ trợ sạc di động theo giả định lab, không hướng dẫn tự chạy tới trạm. Thiếu dữ liệu → người xác minh. Chưa xác nhận có nguồn lực sạc di động → điều phối viên chọn phương án hỗ trợ đã được đơn vị cho phép. |
| Bằng chứng còn thiếu | Log thời gian, khả năng truy cập nguồn trạm, quy trình hỗ trợ chính thức, mức sẵn sàng của điều phối viên. |

## 4. Quick Problem Card #2 — P2: Phân loại phản ánh cư dân Vinhomes

**Bài toán một câu:** Nhân viên ban quản lý cần chuyển phản ánh đến đúng bộ phận ngay lần đầu, dù cư dân mô tả tự do hoặc gộp nhiều vấn đề trong một ticket.

| Trường | Nội dung |
|---|---|
| Công ty / Actor | Vinhomes; nhân viên CSKH/ban quản lý, đầu mối kỹ thuật và cư dân. |
| Trigger / Input | Ticket dạng chữ; mã tòa và vị trí sự cố được xác minh. Prototype chỉ xử lý text, không nhận dạng ảnh/camera. |
| Current workflow — 5 bước | (1) Tiếp nhận: 0,5 phút → (2) đọc và xác minh vị trí: 1,5 phút → (3) phân loại/chọn nơi nhận: 1,5 phút → (4) nhập và chuyển: 1 phút → (5) xác nhận tiếp nhận: 0,5 phút. Tổng 5 phút/ticket giả định. |
| Bottleneck | Bước 2–3; nội dung như “nước tràn ngoài hành lang, đèn chớp” có thể cần nhiều nơi nhận và kiểm tra mức khẩn cấp. |
| Handoff | Cư dân → CSKH → đầu mối kỹ thuật; ticket có thể bị trả lại khi thiếu vị trí. |
| Business impact giả định | 150 × 5 = 750 phút/ngày, tức 12,5 giờ xử lý. Tỷ lệ chuyển sai chưa biết, phải đo riêng. |
| AI hỗ trợ ở đâu? | Tóm tắt, đề xuất nhãn và câu hỏi bổ sung; không tự gán mức độ an toàn chỉ dựa vào cảm xúc của văn bản. |
| Success metric đề xuất | Median thời gian định tuyến được duyệt ≤ 2 phút; macro-F1 phân loại ≥ 0,90 trên tập giữ lại; tỷ lệ chuyển lại ≤ 5%; các ví dụ nguy hiểm trong bộ eval đều được chuyển người xử lý. |
| Quick Architecture | **Rule + LLM Feature**. Dùng form bắt buộc nếu thiếu trường là nguyên nhân chính. |
| Boundary / Fallback | Không tự đóng ticket, hứa thời hạn sửa, xác nhận trách nhiệm hay tiết lộ dữ liệu căn hộ. Ca nguy hiểm/mơ hồ → đầu mối trực; mô hình lỗi → phân loại thủ công. |
| Bằng chứng còn thiếu | Danh mục bộ phận, nhãn chuẩn, ticket đã khử định danh, SLA hiện hành và tỷ lệ chuyển sai. |

## 5. Quick Problem Card #3 — P3: Đọc email đặt phòng đoàn Vinpearl

**Bài toán một câu:** Nhân viên đặt phòng cần chuyển email tự do thành yêu cầu có cấu trúc mà không bỏ sót ngày, số lượng hoặc điều kiện chưa xác nhận.

| Trường | Nội dung |
|---|---|
| Công ty / Actor | Vinpearl; nhân viên đặt phòng đoàn và đơn vị lữ hành. |
| Trigger / Input | Một email hỏi phòng; có thể thiếu năm, có hai phương án ngày hoặc nhiều loại phòng. |
| Current workflow — 5 bước | (1) Nhận/đọc email: 3 phút → (2) nhập trường: 4 phút → (3) ghi câu hỏi bổ sung: 2 phút → (4) kiểm tra quỹ phòng: 2 phút → (5) soạn trả lời: 1 phút. Tổng 12 phút/email giả định. |
| Bottleneck | Bước 1–2; phải diễn giải nhiều cách ghi ngày và gõ lại số lượng từ văn bản. |
| Handoff | Đại lý → nhân viên đặt phòng → nguồn quỹ phòng → nhân viên → đại lý. |
| Business impact giả định | 25 × 12 = 300 phút/ngày. Chưa biết chi phí của lỗi nhập liệu hoặc doanh thu tác động. |
| AI hỗ trợ ở đâu? | Trích xuất JSON: ngày, số phòng, số khách, loại phòng, trường còn thiếu; mỗi trường có đoạn dẫn chứng từ email. |
| Success metric đề xuất | Median xử lý ≤ 6 phút/email; độ chính xác từng trường bắt buộc ≥ 95% trên tập giữ lại; không tự điền trường thiếu trong bộ eval. |
| Quick Architecture | **LLM Feature + Rule kiểm tra + HITL**; đối chiếu với form đặt đoàn chuẩn để đo lợi ích tăng thêm. |
| Boundary / Fallback | Không tự báo giá, giữ phòng, xác nhận booking hay suy đoán phòng trống. Ngày mơ hồ → null và hỏi lại; lỗi API → nhập thủ công. |
| Bằng chứng còn thiếu | Email được phép sử dụng, mẫu trường chuẩn, tỷ lệ thiếu dữ liệu, quyền đọc quỹ phòng và quy trình xác nhận. |

## 6. Lựa chọn cá nhân

Tôi chọn **P1 — Xanh SM** để phân tích sâu vì có đầu vào/đầu ra rõ, tách được quyết định bằng rule khỏi tác vụ soạn nháp và khớp với hợp đồng kỹ thuật của starter code. Đây là lựa chọn để học và kiểm thử ranh giới, không khẳng định P1 có ROI cao nhất ngoài thực tế.

P2 và P3 vẫn là ứng viên tốt nhưng cần bộ nhãn và dữ liệu nghiệp vụ riêng. P4 nên thử đối soát bằng rule trước; P5 cần giới hạn rất chặt giữa thông tin hành chính và nội dung y tế. Kết luận triển khai P1 phải chờ dữ liệu và kiểm thử, được ghi trong `02-deep-dive-report.md`.
