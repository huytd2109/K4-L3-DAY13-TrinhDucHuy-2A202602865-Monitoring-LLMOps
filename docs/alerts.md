# Alert và Runbook

Mỗi alert phải dựa trên triệu chứng người dùng hoặc SLO, không dựa trực tiếp vào tên implementation nội bộ.

## Alert 1

- Tên: High user-facing latency
- Severity: critical
- Duration: 5m
- Kênh thông báo: Slack
- SLI/SLO liên quan: Fast successful requests, latency không quá 3000 ms.
- Điều kiện và thời gian duy trì: P95 latency lớn hơn 3000 ms liên tục 5 phút.
- Ảnh hưởng tới người dùng: Phản hồi chat chậm, có nguy cơ timeout.
- Ba bước kiểm tra đầu tiên: (1) xác định khoảng tăng P95 trên panel latency; (2) lọc log `response_sent` chậm và lấy `correlation_id`; (3) mở trace cùng ID, so sánh duration của retrieval và generation.
- Mitigation tạm thời: Giảm concurrency hoặc chuyển prompt về bản production ổn định; nếu retrieval chậm, bật fallback tài liệu và cô lập dependency lỗi.
- Owner: api-oncall

## Alert 2

- Tên: Elevated request error rate
- Severity: critical
- Duration: 5m
- Kênh thông báo: Slack
- SLI/SLO liên quan: Error rate guardrail tối đa 2%.
- Điều kiện và thời gian duy trì: Tỷ lệ `request_failed` trên `request_received` lớn hơn 2% liên tục 5 phút.
- Ảnh hưởng tới người dùng: Request chat trả lỗi thay vì câu trả lời.
- Ba bước kiểm tra đầu tiên: (1) kiểm tra error rate và nhóm theo `error_type`; (2) chọn log lỗi có `correlation_id`; (3) mở trace cùng ID để xác định child span lỗi.
- Mitigation tạm thời: Tắt incident practice nếu đang bật; cô lập dependency lỗi và dùng fallback an toàn trong khi khôi phục dịch vụ.
- Owner: api-oncall

## Alert 3

- Tên: Low retrieval success rate
- Severity: warning
- Duration: 10m
- Kênh thông báo: Slack
- SLI/SLO liên quan: Retrieval success guardrail tối thiểu 90%.
- Điều kiện và thời gian duy trì: Tỷ lệ `tool_success=true` của retrieval thấp hơn 90% liên tục 10 phút.
- Ảnh hưởng tới người dùng: Câu trả lời thiếu ngữ cảnh hoặc request thất bại.
- Ba bước kiểm tra đầu tiên: (1) kiểm tra panel errors/retrieval; (2) lọc log `tool_name=retrieval` và `tool_success=false`; (3) đối chiếu trace cùng `correlation_id` để xem lỗi hoặc timeout retrieval.
- Mitigation tạm thời: Chuyển sang corpus/fallback đã kiểm chứng, giới hạn retry và khôi phục vector store trước khi trả traffic đầy đủ.
- Owner: rag-oncall
