# Nhóm 2: Giám sát & quan sát hệ thống
## Monitoring & Observability — Network + Service + Bandwidth + Power Actions

**Gộp từ:** Network Monitoring + Service Uptime Monitor + Bandwidth Tracker + Wake-on-LAN Dashboard

---

## 1. Tổng quan ý tưởng

Xây dựng một hệ thống trung tâm để **theo dõi tình trạng hoạt động của thiết bị, dịch vụ và một số chỉ số hạ tầng** trong mạng nội bộ.

Hệ thống trả lời các câu hỏi:

- Thiết bị nào đang Online/Offline?
- Dịch vụ Web/SSH/API có còn hoạt động không?
- Lần kiểm tra gần nhất là khi nào?
- Lần cuối thiết bị phản hồi là khi nào?
- Một dịch vụ đã Down trong bao lâu?
- Có thiết bị nào đang có dấu hiệu bất thường?
- Băng thông hoặc Traffic đang thay đổi như thế nào?
- Có cần gửi cảnh báo cho Administrator không?
- Có thể thực hiện một số hành động quản trị đơn giản như Wake-on-LAN không?

Điểm cốt lõi của nhóm này là mô hình:

```text
Target
   ↓
Periodic Check
   ↓
Result / Metric
   ↓
Status
   ↓
Dashboard / Alert
```

So với Nhóm 1, nhóm này không tập trung vào việc "mạng có những tài nguyên nào" mà tập trung vào câu hỏi:

> **Những tài nguyên và dịch vụ đó hiện đang hoạt động như thế nào?**

---

## 2. Vấn đề thực tế cần giải quyết

Nếu không có hệ thống Monitoring, Administrator thường phải:

- Ping thiết bị thủ công.
- SSH vào từng Server để kiểm tra.
- Mở từng Website/API để xem còn chạy không.
- Chỉ biết sự cố khi người dùng báo.
- Không biết thiết bị đã Down từ lúc nào.
- Không có lịch sử để phân tích.
- Không biết lỗi là tạm thời hay kéo dài.
- Khó theo dõi nhiều thiết bị cùng lúc.

Khi số lượng thiết bị và dịch vụ tăng lên, kiểm tra thủ công không còn hiệu quả.

Hệ thống Monitoring tự động thực hiện các Check định kỳ và tập trung kết quả về Dashboard.

---

## 3. Ứng dụng thực tế

### 3.1. System Administration

Theo dõi:

- Linux Server.
- Windows Server.
- NAS.
- VM.
- Container Host.
- Database Server.

### 3.2. Network Administration

Theo dõi:

- Router.
- Switch.
- Access Point.
- Gateway.
- IoT Device.

### 3.3. Service Monitoring

Theo dõi:

- HTTP/HTTPS.
- SSH.
- TCP Port.
- API Endpoint.
- Internal Service.

### 3.4. Homelab

Theo dõi:

- Home Server.
- NAS.
- Raspberry Pi.
- ESP32.
- Self-hosted Service.

---

## 4. Giá trị chính của đề tài

Nhóm này có kiến trúc rất rõ:

```text
Scheduler
   ↓
Check Queue
   ↓
Worker
   ↓
Target
   ↓
Result
   ↓
Database
   ↓
Dashboard
```

Do nhiều loại Monitoring đều dùng chung kiến trúc trên, hệ thống có thể bắt đầu rất nhỏ với:

```text
ICMP
+
TCP / HTTP
```

rồi mở rộng dần sang:

```text
Metrics
+
History
+
Alert
+
SNMP
+
Bandwidth
+
Observability
```

Đây là ưu điểm lớn nhất của nhóm.

---

## 5. Kiến trúc sơ bộ

```text
┌──────────────────────────────┐
│          Frontend            │
│ Dashboard / Target / Alert   │
└──────────────┬───────────────┘
               │ REST API
               ▼
┌──────────────────────────────┐
│            Backend           │
│                              │
│ ├── Target Management        │
│ ├── Monitoring Check         │
│ ├── Scheduler                │
│ ├── Worker Pool              │
│ ├── Status Engine            │
│ └── Alert Module (optional)  │
└──────────────┬───────────────┘
               │
               ▼
        ┌──────────────┐
        │ PostgreSQL   │
        └──────────────┘
```

Ở đồ án chuyên ngành có thể bổ sung Time-Series Database hoặc tích hợp Prometheus/Grafana.

---

## 6. Mô hình dữ liệu sơ bộ

Có thể mô hình hóa:

```text
Target
  |
  └── Monitoring Check
          |
          └── Monitoring Result
```

Ví dụ:

```text
Server-01
├── ICMP Check
├── TCP:22 Check
└── HTTP:8080 Check
```

Trong đồ án cơ sở có thể giữ dữ liệu đơn giản:

```text
Target
Check Type
Target Address
Status
Last Check
Last Seen
Consecutive Failures
```

Chưa cần lưu toàn bộ Metrics lịch sử nếu muốn giữ Scope nhỏ.

---

## 7. Các loại Check có thể hỗ trợ

### 7.1. ICMP Check

Dùng để kiểm tra thiết bị có phản hồi mạng hay không.

```text
Target
  ↓
ICMP Ping
  ↓
ONLINE / OFFLINE / UNKNOWN
```

### 7.2. TCP Check

Kiểm tra một Port có mở/đáp ứng hay không.

Ví dụ:

```text
22
80
443
5432
```

### 7.3. HTTP Check

Kiểm tra:

- Endpoint có phản hồi hay không.
- HTTP Status.
- Response Time cơ bản.

### 7.4. Bandwidth / Traffic

Trong đồ án cơ sở chỉ nên xem là Extension.

Có thể lấy dữ liệu từ:

- SNMP.
- System Agent.
- Host Metrics.

Phần này thích hợp hơn ở đồ án chuyên ngành vì cần lưu dữ liệu theo thời gian.

### 7.5. Wake-on-LAN

Không phải Monitoring Core.

Có thể là một Action nhỏ:

```text
Device Offline
      |
      v
Administrator chọn Wake
      |
      v
Send WoL Packet
```

Phù hợp làm chức năng phụ cho Dashboard nếu môi trường Lab hỗ trợ.

---

## 8. Monitoring State

Trạng thái cơ bản:

```text
UNKNOWN
ONLINE
OFFLINE
```

Có thể áp dụng cho Device.

Đối với Service:

```text
UNKNOWN
UP
DOWN
```

Có thể thống nhất thành một Status Model chung ở mức Backend nếu thiết kế phù hợp.

---

## 9. False Positive và Retry

Không nên coi một lần thất bại là Down ngay.

Ví dụ:

```text
Check #1 FAIL
Check #2 FAIL
Check #3 FAIL
        ↓
OFFLINE / DOWN
```

Nếu thành công:

```text
Status = ONLINE / UP
Failure Count = 0
Last Seen = Current Time
```

Việc này giúp giảm cảnh báo sai khi mạng chập chờn hoặc có Packet Loss tạm thời.

---

## 10. Worker Pool và Concurrency

Nếu kiểm tra hàng trăm Target tuần tự thì Monitoring Cycle có thể kéo dài.

Do đó có thể sử dụng:

```text
Scheduler
   ↓
Job Queue
   ↓
Worker Pool
```

Số Worker được giới hạn bằng cấu hình.

Ví dụ:

```text
MONITOR_INTERVAL=60s
CHECK_TIMEOUT=2s
OFFLINE_THRESHOLD=3
MONITOR_WORKERS=20
```

Không cần Distributed Scheduler ở đồ án cơ sở.

---

## 11. Phạm vi đề xuất cho đồ án cơ sở ngành

### Nhóm A — Core bắt buộc

- CRUD Target.
- ICMP Monitoring.
- TCP hoặc HTTP Service Check.
- Scheduler.
- Worker Pool.
- Retry Threshold.
- Online/Offline/Unknown.
- Up/Down/Unknown.
- Last Check.
- Last Seen.
- Dashboard.
- Search / Filter.
- Authentication Administrator.
- Testing.

### Nhóm B — Mở rộng nhẹ nếu còn thời gian

- HTTP Response Time.
- Basic Alert Telegram/Email.
- Wake-on-LAN.
- Lưu một lượng History nhỏ.
- Basic Service Availability.
- Một chỉ số Traffic đơn giản.

### Nhóm C — Không làm trong đồ án cơ sở

- Full Prometheus replacement.
- Full Grafana replacement.
- Distributed Collector.
- High-availability Monitoring.
- Long-term Time-Series Metrics.
- Full SNMP Performance Monitoring.
- Complex Alert Rule Engine.
- Log Aggregation.
- Distributed Tracing.

---

## 12. Dashboard cơ sở ngành

Dashboard có thể hiển thị:

```text
Targets       : 25
Online        : 21
Offline       : 3
Unknown       : 1

Services      : 18
Up            : 16
Down          : 2
```

Danh sách:

| Target | Check | Status | Last Check | Last Seen |
|---|---|---|---|---|
| Server-01 | ICMP | Online | 10:32 | 10:32 |
| Server-01 | TCP/22 | Up | 10:32 | 10:32 |
| Web-01 | HTTP | Down | 10:32 | 10:28 |

Có thể Filter theo:

- Target.
- Status.
- Check Type.

---

## 13. Kịch bản Demo cơ sở ngành

Ví dụ:

```text
Router-R1
Server-01
Web-01
NAS-01
```

Các Check:

```text
Router-R1 → ICMP
Server-01 → ICMP + TCP/22
Web-01    → ICMP + HTTP
NAS-01    → ICMP
```

Demo:

1. Login.
2. Thêm Target.
3. Tạo ICMP Check.
4. Tạo HTTP/TCP Check.
5. Dashboard hiển thị trạng thái.
6. Tắt một VM.
7. Sau số lần Fail quy định, Target chuyển Offline.
8. Stop Web Service nhưng giữ VM Online.
9. Device vẫn Online nhưng HTTP Service chuyển Down.
10. Start lại Service.
11. Dashboard cập nhật lại.
12. Nếu có WoL: gửi Wake Packet cho Device hỗ trợ.

Điểm demo #8-9 rất quan trọng vì chứng minh hệ thống phân biệt:

```text
Host Reachability
khác
Service Availability
```

---

## 14. Điểm cần quyết định khi đi sâu

- Target là Device hay Endpoint tổng quát.
- Một Target có bao nhiêu Check.
- ICMP implementation.
- TCP/HTTP timeout.
- Scheduler interval.
- Worker Pool size.
- Retry Threshold.
- Có lưu History ở đồ án cơ sở hay không.
- Alert nằm trong Core hay Bonus.
- WoL có cần hỗ trợ không.
- Frontend dùng React hay Server-render.
- Database chỉ PostgreSQL hay thêm Time-Series DB ở kỳ sau.

---

## 15. Rủi ro và độ khó

### Độ khó tổng thể

**Trung bình** nếu chỉ làm ICMP + TCP/HTTP + Dashboard.

### Rủi ro

- Dễ mở rộng quá nhanh sang Full Observability.
- Nếu lưu mọi Check Result thì Database tăng nhanh.
- Alert có thể gây Spam nếu State Transition không rõ.
- Monitoring nhiều Target cần xử lý Concurrency.
- HTTP Monitoring có nhiều Edge Case nếu cố hỗ trợ quá sâu.
- Bandwidth Monitoring có thể kéo theo SNMP/Time-Series.

### Cách giảm rủi ro

Core chỉ cần:

```text
Current Status
+
Last Check
+
Last Seen
+
ICMP
+
TCP/HTTP
```

History, Traffic và Alert nâng cao để dành cho đồ án chuyên ngành.

---

## 16. Nâng cấp thành đồ án chuyên ngành

Hướng nâng cấp:

# Monitoring & Observability Platform

Kiến trúc:

```text
Targets
   |
   v
Collectors / Checks
   |
   v
Metrics / Events
   |
   v
Time-Series Storage
   |
   +-- Dashboard
   +-- Alert
   +-- History
   +-- Analytics
```

Các module có thể bổ sung:

### 16.1. Metrics

- CPU.
- RAM.
- Disk.
- Network Traffic.
- Latency.
- Packet Loss.
- Service Response Time.

### 16.2. SNMP

Theo dõi:

- Router.
- Switch.
- Interface.
- Uptime.
- Traffic.

### 16.3. Time-Series

Lưu dữ liệu theo thời gian để:

- Vẽ biểu đồ.
- So sánh.
- Tính Availability.
- Phân tích xu hướng.

### 16.4. Alert

- Telegram.
- Email.
- Webhook.
- Alert Threshold.
- Alert Recovery.

### 16.5. Dashboard nâng cao

- Traffic chart.
- Availability.
- Device Health.
- Service Health.
- Historical Timeline.

### 16.6. Integration

Có thể tích hợp hoặc học theo các công cụ thật:

- Prometheus.
- Grafana.
- InfluxDB.

Mục tiêu không nhất thiết là tự viết lại toàn bộ các công cụ này, mà có thể tự xây Core để hiểu nguyên lý rồi tích hợp công nghệ ngành dùng thật ở giai đoạn chuyên ngành.

---

## 17. Hướng phát triển xa hơn

Có thể mở rộng thành:

```text
Monitoring
   +
Metrics
   +
Logs
   +
Alert
   ↓
Observability Platform
```

Hoặc kết hợp với Nhóm 1:

```text
Inventory / IPAM
       +
Monitoring
       ↓
Network Management System
```

Do đó Nhóm 2 có khả năng hội tụ tự nhiên với Nhóm 1 nếu sau này muốn xây một NMS lớn.

---

## 18. Giá trị đối với CV / Portfolio

Phù hợp với:

- System Administrator.
- Infrastructure Engineer.
- DevOps Engineer.
- Site Reliability Engineer.
- NOC Engineer.
- Platform Engineer ở giai đoạn xa hơn.

Có thể chứng minh:

- Background Worker.
- Scheduler.
- Concurrency.
- Network Check.
- Service Check.
- State Machine.
- Database.
- REST API.
- Dashboard.
- Alerting.
- Time-Series khi nâng cấp.

Đây là hướng có giá trị CV tốt đặc biệt nếu sau này kết hợp với Prometheus/Grafana và có một Lab thực tế.

---

## 19. Đánh giá sơ bộ

| Tiêu chí | Đánh giá |
|---|---|
| Phù hợp đồ án cơ sở | Rất cao |
| Kiến thức Network | Trung bình – cao |
| System/Backend | Cao |
| Concurrency/Worker | Cao |
| Khả năng Demo | Rất cao |
| Độ khó | Trung bình |
| Nguy cơ Scope Creep | Trung bình |
| Khả năng nâng chuyên ngành | Rất cao |
| Giá trị CV DevOps/SRE | Rất cao |
| Giá trị CV Network thuần | Cao |

---

## 20. Kết luận ý tưởng

Cấu trúc phát triển:

```text
Đồ án cơ sở
Basic Monitoring
ICMP + TCP/HTTP + Dashboard

        ↓

Đồ án chuyên ngành
Monitoring & Observability Platform

        ↓

Phát triển xa hơn
SRE / DevOps Observability Stack
```

Nhóm này có ưu điểm là **Scope đồ án cơ sở dễ kiểm soát hơn Nhóm 1**, nhưng vẫn có đường nâng cấp rất dài.

Điểm quan trọng là không biến đồ án cơ sở thành một phiên bản Prometheus/Grafana tự viết. Phần cơ sở chỉ cần chứng minh hệ thống Monitoring có Scheduler, Worker, State và Dashboard hoạt động đúng; các phần Metrics/History/Alert nâng cao để dành cho đồ án chuyên ngành.
