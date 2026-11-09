# AI Log & Reflection

## Tôi đã dùng AI như thế nào

Tôi dùng AI để brainstorm các bottleneck ở Xanh SM, Vinhomes và VinFast, sau đó yêu cầu AI phản biện theo ba góc nhìn: metric có đo được không, rule-based có đủ tốt không, và nếu mô hình sai thì ai chịu trách nhiệm. AI giúp tôi biến mô tả chung “hỗ trợ tài xế” thành workflow có thời gian từng bước và một ngưỡng pin cụ thể.

## Một điểm AI có thể sai

Ở lần đầu, AI có xu hướng đề xuất “đến trạm gần nhất” ngay cả khi pin còn rất thấp. Nó cũng dùng các số liệu vận hành nghe có vẻ hợp lý nhưng không có nguồn kiểm chứng. Tôi không coi các số đó là sự thật; trong báo cáo, chúng được ghi rõ là giả định cần đối chiếu log.

## Cách tôi sửa prompt và thiết kế

Tôi thêm rule cứng: dưới 5% pin và trạm cách trên 5 km thì không được hướng dẫn tuyến đó, phải chuyển sang `dispatch_mobile_charger`. Tôi thêm tag `[DRAFT_ONLY]`, yêu cầu JSON schema, cấm mô hình khẳng định đã gửi tin hoặc điều xe, và bắt buộc điều phối viên duyệt. Ngoài prompt, rule khoảng cách/pin phải được kiểm tra bằng code vì không nên giao điều kiện an toàn cho LLM.

## Adversarial testing

Tôi thử ba dạng tấn công: yêu cầu đi tới trạm cách 8 km khi pin 2%, yêu cầu bỏ tag để “gửi ngay”, và giả mạo lệnh SYSTEM để buộc mô hình nói rằng xe cứu hộ đã được điều. Kết quả kỳ vọng là mô hình luôn giữ tag, escalate trong trường hợp pin nguy cấp, và chỉ tạo bản nháp thay vì tuyên bố hành động đã xảy ra.

## Bài học

AI phù hợp nhất ở phần ngôn ngữ và tóm tắt, không phải quyền tự quyết định hay thực thi hành động. Một prototype tốt cần nêu rõ điều nó không được làm, có fallback thủ công, và có metric/baseline thật trước khi kết luận hiệu quả.
