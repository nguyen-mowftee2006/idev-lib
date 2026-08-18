# Ý tưởng 03: Giám sát & quan sát hệ thống

> Hướng xây dựng một hệ thống theo dõi trạng thái thiết bị, dịch vụ và một số chỉ số hạ tầng trong mạng nội bộ.

## 1. Tổng quan & bài toán

Ý tưởng 03 tập trung vào câu hỏi:

> Những tài nguyên và dịch vụ trong mạng hiện đang hoạt động như thế nào?

Khác với Ý tưởng 01, trọng tâm không phải là kiểm kê mạng có gì, mà là kiểm tra trạng thái hoạt động của device, service và một số metric cơ bản. Mô hình cốt lõi có thể là:

```text
Target
   ↓
Check
   ↓
Result / Metric
   ↓
Status
   ↓
Dashboard / Alert
```

Bài toán thực tế là nếu không có monitoring, administrator thường phải ping thủ công, SSH vào từng server, mở từng web/API để kiểm tra và chỉ biết sự cố khi người dùng báo.

## 2. Mục tiêu

Ý tưởng có thể bắt đầu nhỏ với:

```text
ICMP
+
TCP / HTTP
+
Dashboard
```

Mục tiêu là xây được một hệ thống tự động kiểm tra định kỳ, phân biệt trạng thái host và service, lưu trạng thái gần nhất và hiển thị trên dashboard.

Các câu hỏi cần trả lời:

- Thiết bị nào đang ONLINE, OFFLINE hoặc UNKNOWN?
- Dịch vụ nào đang UP, DOWN hoặc UNKNOWN?
- Lần kiểm tra gần nhất là khi nào?
- Lần cuối target phản hồi thành công là khi nào?
- Host có thể online nhưng service vẫn down hay không?

## 3. Hướng triển khai

Kiến trúc tổng quát có thể đi theo luồng:

```text
Scheduler
   ↓
Job Queue
   ↓
Worker Pool
   ↓
Target
   ↓
Result
   ↓
Database
   ↓
Dashboard
```

Mô hình dữ liệu có thể giữ đơn giản:

```text
Target
  ↓
Monitoring Check
  ↓
Monitoring Result
```

Một target có thể có nhiều check, ví dụ:

```text
Server-01
├── ICMP Check
├── TCP:22 Check
└── HTTP:8080 Check
```

Core chỉ cần lưu trạng thái hiện tại, `Last Check`, `Last Seen` và số lần fail liên tiếp. History dài hạn có thể để dành cho giai đoạn mở rộng.

## 4. Kỹ thuật & công nghệ

Các kỹ thuật quan trọng:

- **ICMP check:** kiểm tra host reachability.
- **TCP check:** kiểm tra port như 22, 80, 443, 5432.
- **HTTP check:** kiểm tra endpoint, HTTP status và response time cơ bản nếu phù hợp.
- **Status model:** `ONLINE / OFFLINE / UNKNOWN` cho host, `UP / DOWN / UNKNOWN` cho service.
- **Retry threshold:** không đánh dấu down chỉ sau một lần fail.
- **Scheduler và worker pool:** kiểm tra nhiều target định kỳ mà không chạy tuần tự quá lâu.
- **Dashboard:** hiển thị target, check type, status, last check và last seen.

Điểm cần giữ rõ là **Host Reachability** khác **Service Availability**. Một VM có thể vẫn ping được nhưng HTTP service bên trong đã down.

## 5. Phạm vi

### Core

- CRUD Target.
- ICMP monitoring.
- TCP hoặc HTTP service check.
- Scheduler.
- Worker pool.
- Retry threshold.
- `ONLINE / OFFLINE / UNKNOWN` và `UP / DOWN / UNKNOWN`.
- `Last Check`, `Last Seen`.
- Dashboard, search/filter.
- Authentication administrator ở mức đơn giản nếu cần.

### Mở rộng

- HTTP response time.
- Alert Telegram/Email ở mức cơ bản.
- Wake-on-LAN.
- Lưu một lượng history nhỏ.
- Traffic hoặc bandwidth đơn giản.
- SNMP, time-series metrics, distributed collector, log aggregation và tracing nên để dành cho giai đoạn sau.

Không nên biến core thành một phiên bản Prometheus/Grafana tự viết.

## 6. Demo dự kiến

Một demo gọn có thể dùng:

```text
Router-R1 → ICMP
Server-01 → ICMP + TCP/22
Web-01    → ICMP + HTTP
NAS-01    → ICMP
```

Luồng demo:

1. Thêm target và tạo ICMP/TCP/HTTP check.
2. Dashboard hiển thị trạng thái ban đầu.
3. Tắt một VM để sau retry threshold target chuyển `OFFLINE`.
4. Stop web service nhưng giữ VM online.
5. Chứng minh host vẫn `ONLINE` nhưng HTTP service `DOWN`.
6. Start lại service hoặc VM để trạng thái phục hồi.
7. Nếu có alert hoặc WoL, demo như một chức năng phụ.

Demo nên làm rõ scheduler, worker, state transition và dashboard.

## 7. Hướng phát triển

Roadmap có thể đi theo hướng:

```text
Monitoring
    ↓
Metrics + History + Alert
    ↓
Observability
```

Ở giai đoạn chuyên ngành, có thể mở rộng sang SNMP, time-series storage, availability chart, traffic metrics, alert recovery và tích hợp Prometheus/Grafana/InfluxDB nếu phù hợp.

Ý tưởng này cũng có thể hội tụ với Ý tưởng 01:

```text
Inventory / IPAM
       +
Monitoring
       ↓
Network Management System
```

## 8. Đánh giá

| Tiêu chí | Đánh giá |
| --- | --- |
| Độ khó | Trung bình |
| Networking | Trung bình – cao |
| System / Software | Cao |
| Khả năng demo | Rất cao |
| Khả năng mở rộng | Rất cao |
| Giá trị portfolio | Rất cao cho DevOps/SRE/NOC, cao cho Network |
| Rủi ro chính | Mở rộng quá nhanh sang full observability, history, alerting hoặc SNMP |
