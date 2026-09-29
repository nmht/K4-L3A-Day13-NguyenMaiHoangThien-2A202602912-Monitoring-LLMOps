# Template Alert và Runbook

Mỗi alert phải dựa trên triệu chứng người dùng hoặc SLO, không dựa trực tiếp vào tên implementation nội bộ.

## High-Error-Rate

- Tên: HighErrorRate
- Severity: Critical
- Duration: 5m
- Kênh thông báo: Slack
- SLI/SLO liên quan: fast_successful_requests (Error Budget)
- Điều kiện và thời gian duy trì: error_rate_pct > 2 trong 5 phút
- Ảnh hưởng tới người dùng: Người dùng liên tục nhận được lỗi khi gọi API chat.
- Ba bước kiểm tra đầu tiên: Check dashboard để xem lỗi nào tăng đột biến, check Langfuse để xem trace, check logs `data/logs.jsonl` để lấy correlation_id.
- Mitigation tạm thời: Rollback prompt version, hoặc restart service, hoặc disable feature nếu cần.
- Owner: devops-team

## High-Latency-P95

- Tên: HighLatencyP95
- Severity: Warning
- Duration: 10m
- Kênh thông báo: Slack
- SLI/SLO liên quan: fast_successful_requests
- Điều kiện và thời gian duy trì: latency_ms_p95 > 1000 trong 10 phút
- Ảnh hưởng tới người dùng: Người dùng phải đợi lâu để nhận được câu trả lời.
- Ba bước kiểm tra đầu tiên: Check panel Latency, xem TTFT có cao không, check Langfuse span để xem RAG hay LLM bị chậm.
- Mitigation tạm thời: Tắt tính năng RAG phụ, hoặc tăng cường tài nguyên.
- Owner: devops-team

## High-Cost-Spike

- Tên: HighCostSpike
- Severity: Critical
- Duration: 1m
- Kênh thông báo: Slack
- SLI/SLO liên quan: guardrails daily_cost_usd_max
- Điều kiện và thời gian duy trì: Tổng cost USD > 1.0 trong 1 phút
- Ảnh hưởng tới người dùng: Không trực tiếp ảnh hưởng người dùng nhưng ảnh hưởng chi phí vận hành.
- Ba bước kiểm tra đầu tiên: Check panel cost và tokens, xem tokens_out có bất thường không, check Langfuse trace xem có bị loop prompt không.
- Mitigation tạm thời: Tạm thời rate limit hoặc fallback sang model rẻ hơn.
- Owner: devops-team
