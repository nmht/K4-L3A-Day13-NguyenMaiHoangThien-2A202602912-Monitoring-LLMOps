# Báo cáo cá nhân — K4-L3A Day 13 Monitoring & LLMOps

> Mỗi học viên hoàn thiện một file duy nhất này. Khi dẫn evidence, dùng đường dẫn tương đối, ví dụ `evidence/07-trace-waterfall.png`.

## 1. Thông tin học viên

- **Họ và tên:** Nguyễn Mai Hoàng Thiện
- **MSSV:** 2A202602912
- **Lớp:** K4-L3A
- **Repository URL:** <https://github.com/nmht/K4-L3A-Day13-NguyenMaiHoangThien-2A202602912-Monitoring-LLMOps>
- **Commit SHA cuối:**
- **Challenge ID:**
- **Tên project Langfuse cá nhân:** `day13-k4-l3a-2a202602912`

## 2. Evidence index

Điền đúng đường dẫn tới evidence thực tế. Có thể đổi tên hoặc dùng nhiều ảnh nếu cần.

| Evidence            | Đường dẫn                             |
| ------------------- | ------------------------------------- |
| Pytest cuối         | `evidence/01-pytest.png`              |
| Log validator       | `evidence/02-log-validator.png`       |
| Dashboard validator | `evidence/03-dashboard-validator.png` |
| Structured log      | `evidence/04-structured-log.png`      |
| PII redaction       | `evidence/05-pii-redaction.png`       |
| Trace list          | `evidence/06-trace-list.png`          |
| Trace waterfall     | `evidence/07-trace-waterfall.png`     |
| Trace metadata      | `evidence/08-trace-metadata.png`      |
| Prompt versions     | `evidence/09-prompt-versions.png`     |
| Prompt rollback     | `evidence/10-prompt-rollback.png`     |
| Dashboard runtime   | `evidence/11-dashboard-overview.png`  |
| Incident metric     | `evidence/12-incident-metric.png`     |
| Incident log        | `evidence/13-incident-log.png`        |
| Incident trace      | `evidence/14-incident-trace.png`      |

## 3. Kết quả kỹ thuật

| Nội dung                | Baseline  | Kết quả cuối     | Nhận xét                                 |
| ----------------------- | --------- | ---------------- | ---------------------------------------- |
| `validate_logs.py`      | 30/100    | 100/100          | Đã fix toàn bộ lỗi missing fields và PII |
| `validate_dashboard.py` | 6/6 panel | 6/6 panel        | Dashboard config hợp lệ, đủ 6 panel      |
| `pytest`                | 22 passed | 24 passed        | Đã pass toàn bộ và thêm test PII mới     |
| Số traces hợp lệ        | 0         | > 10             | Có span tree rõ ràng từ RAG và LLM       |
| Số PII leak             | 0         | 0                | Middleware scrub hoạt động chính xác     |
| Latency P95 / TTFT P95  | N/A       | \~400ms / \~60ms | Phản hồi nhanh theo đúng chuẩn SLO       |
| Retrieval success rate  | N/A       | 100%             | Không có document nào rớt                |

## 4. Logging và PII

- **Cách tạo/nhận và truyền correlation ID:** Sử dụng ASGI middleware để lấy giá trị từ header `x-request-id`, nếu không có sẽ tự sinh `req-<hex>`. ID này được lưu vào `contextvars` để dùng xuyên suốt vòng đời request.
- **Các metadata được ghi vào structured log:** Ghi log dưới dạng JSON với các trường: `correlation_id`, `session_id`, `user_id_hash`, `env`, `model`, `feature`, `latency_ms`, `ttft_ms`, `tokens_in`, `tokens_out`, `cost_usd`.
- **Cách bảo đảm PII được scrub trước khi ghi:** Sử dụng structlog processor `scrub_event` để lọc text bằng regex cho (email, phone, CCCD, credit card) thay thế bằng `[REDACTED_...]` trước khi chuyển sang định dạng JSON.
- **Cách kiểm chứng kết quả:** Chạy script `validate_logs.py` (đạt 100/100) và kiểm chứng trực tiếp bằng mắt thông qua file `data/logs.jsonl`.

## 5. Tracing và prompt versioning

- **Cách xác nhận traces do chính tôi tạo trong project cá nhân:** Bằng cách kiểm tra dashboard trên Langfuse tương ứng với project `day13-k4-l3a-2a202602912` thông qua API Keys cá nhân.
- **Cấu trúc root/retrieval/generation observations:** Root là `LabAgent.run`, bên trong có hai child spans là `retrieval` (truy xuất tài liệu DB) và `llm_generation` (gọi mô hình sinh chữ, đính kèm token/cost).
- **Cách nối trace với log:** Ghi `correlation_id` vào thẻ metadata của root span trong quá trình tạo trace bằng `langfuse_client.update_current_span`.
- **Prompt name:** `day13-chat`
- **Version/label baseline:** v1 / `production`
- **Version/label candidate:** v2 / `production`
- **Trace ID của mỗi version:** Được lưu giữ làm bằng chứng trên Langfuse Cloud.
- **Cách promote và rollback production:** Promote/rollback bằng cách chuyển nhãn (label) `production` giữa các version prompt trên giao diện của Langfuse mà không cần khởi động lại ứng dụng.

## 6. Dashboard, SLO và alerts

- **Dashboard và sáu panel:** Gồm Latency (có P50, P95, TTFT), Errors, Traffic, Tokens, Cost, Retrieval Success.
- **SLO và lý do chọn:** Đặt SLO `fast_successful_requests` với target `latency_ms <= 1000`. Vì baseline LLM phản hồi trong khoảng 400ms nên 1000ms là đủ buffer an toàn cho network/hệ thống tải cao.
- **Cách tính error budget:** 100% trừ đi Target Percent (99.5%) bằng 0.5% (Tỷ lệ request tối đa được phép chậm/lỗi trong 28 ngày).
- **Ba alert và runbook tương ứng:** Định nghĩa trong `alert_rules.yaml` và `alerts.md` gồm HighErrorRate, HighLatencyP95, và HighCostSpike (kênh cảnh báo qua Slack).

## 7. Điều tra challenge

- **Challenge ID:** day13-k4-l3a-monitoring-llmops-v1
- **Khoảng thời gian điều tra:** 2026-09-29T16:16:00+07:00
- **Triệu chứng từ metrics:** Dashboard ghi nhận Latency tăng cao đột biến. P95 latency vượt ngưỡng SLO 1000ms (lên tới \~2700ms).
- **Log line và correlation ID liên quan:** `req-3cf68c7a`
- **Trace ID và span gây ảnh hưởng:** Trace cho thấy span `retrieval` (bước lấy tài liệu) là nguyên nhân gây chậm (mất khoảng 2.5s).
- **Root cause:** Lỗi `rag_slow` được kích hoạt làm cho hàm `retrieve` trong file `app/mock_rag.py` thực hiện lệnh `time.sleep(2.5)`.
- **Fix action:** Chạy lệnh `python scripts/inject_incident.py --disable` để tắt sự cố giả lập. Trong thực tế, cần xem xét nguyên nhân làm DB truy vấn chậm để tối ưu index hoặc nâng cấp tài nguyên.
- **Preventive measure:** Áp dụng cơ chế Timeout (ví dụ 1000ms) cho các truy vấn vào Vector DB ở bước retrieval để request không treo vô thời hạn. Cài đặt thêm Alert tự động cảnh báo lên Slack khi DB latency vượt 500ms.

## 8. Giải thích và tự đánh giá

- **Một quyết định kỹ thuật quan trọng và lý do:** Quyết định đẩy `usage_details` thủ công bằng `update_current_generation` lên Langfuse để tracking chính xác Token và Cost theo đúng nghiệp vụ.
- **Một lỗi/blocker đã gặp:** Gặp lỗi 500 Timeout khi chạy `load_test.py` ở CP2 (App không thể kết nối tới hàm Langfuse).
- **Cách tìm nguyên nhân và xử lý:** Xem `data/logs.jsonl` và phát hiện nguyên nhân do thiếu dòng import `observe` và sử dụng sai tham số `usage` của SDK v4. Xử lý bằng cách sửa lại import và đổi tên thuộc tính thành `usage_details`.
- **Cách hiểu luồng Metrics → Logs → Traces:** Metrics báo hiệu cảnh báo tổng quan, Logs giúp định vị request nào dính lỗi qua `correlation_id`, Traces giúp phóng to request đó để xác định chính xác dòng code/bước gây lỗi (root cause).
- **Vai trò của prompt version, token/cost, SLO hoặc rollback trong vận hành LLM:** Prompt version và Rollback giúp linh hoạt cứu vãn hệ thống ngay lập tức trên UI mà không cần can thiệp code. Token và Cost giúp kiểm soát tài chính. SLO là thước đo giữ vững sự uy tín.
- **Điều quan trọng nhất đã học:** Tư duy Observability và phân bổ rõ ràng chức năng của hệ thống giám sát 3 trụ cột (Metrics - Logs - Traces).
- **Hạn chế hoặc phần chưa hoàn thành, nếu có:** Không có, mọi yêu cầu bài tập đã hoàn thành 100%.

## 9. Checklist trước khi nộp

- Kết quả và evidence thuộc commit SHA cuối.
- Tất cả ảnh/output mở được bằng đường dẫn tương đối.
- Incident evidence nối đúng metric → log → trace.
- Trace/prompt evidence thuộc project Langfuse cá nhân và ảnh không lộ key/secret.
- Repository chạy lại được theo README.
- Không có secret, API key, PII thô hoặc evidence của người khác/lớp khác.
- URL repo và commit SHA cuối đã được nộp trên LMS/Codelabs.
