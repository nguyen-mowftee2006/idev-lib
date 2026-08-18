# Nhóm 1: Quản lý tài nguyên & kiểm kê mạng
## Resource & Inventory Management — IPAM + DNS/DHCP + Topology

**Gộp từ:** IPAM tự xây dựng + DNS/DHCP Manager + Network Topology Mapper

---

## 1. Tổng quan ý tưởng

Xây dựng một hệ thống trung tâm để **quản lý và kiểm kê các tài nguyên đang tồn tại trong mạng nội bộ**.

Hệ thống trả lời các câu hỏi chính:

- Mạng hiện có những subnet nào?
- IP nào đã được cấp, IP nào đang giữ trước, IP nào còn khả dụng?
- Thiết bị nào đang sử dụng địa chỉ IP nào?
- Thiết bị có những interface nào?
- Thiết bị/subnet đang thuộc VLAN nào?
- DHCP đang cấp lease cho những thiết bị nào?
- Tên miền nội bộ nào đang trỏ tới thiết bị nào?
- Các thiết bị đang có quan hệ/kết nối với nhau như thế nào?

Ý tưởng cốt lõi là xây dựng một **nguồn dữ liệu tập trung về hạ tầng mạng (network source of truth ở mức đơn giản)** thay cho việc lưu thông tin rời rạc bằng Excel, file cấu hình hoặc ghi nhớ thủ công.

Đây là nhóm có phạm vi rộng nhất trong các hướng đang cân nhắc, vì kết hợp ba bài toán có liên quan chặt chẽ:

```text
IPAM
  +
DNS / DHCP
  +
Network Topology
```

Tuy nhiên, trong **đồ án cơ sở ngành**, trọng tâm không phải làm đầy đủ cả ba phần mà là xây chắc **IPAM + Device Inventory**, sau đó chỉ bổ sung DNS/DHCP/Topology ở mức nhẹ nếu còn thời gian.

---

## 2. Vấn đề thực tế cần giải quyết

Trong mạng có từ vài chục thiết bị trở lên, việc quản lý bằng Excel hoặc ghi chép thủ công bắt đầu xuất hiện nhiều vấn đề:

- Không biết chính xác IP nào đã được cấp.
- Dễ cấp trùng IP.
- Không biết IP đang thuộc thiết bị nào.
- Không biết một thiết bị có bao nhiêu interface.
- Khó xác định subnet hoặc VLAN của thiết bị.
- DHCP lease nằm riêng trong DHCP server.
- DNS record nằm riêng trong DNS server.
- Sơ đồ mạng thường được vẽ thủ công và nhanh chóng lỗi thời.
- Khi troubleshoot phải kiểm tra thông tin từ nhiều nguồn khác nhau.
- Khi có người mới tiếp quản hệ thống, không có một nguồn dữ liệu trung tâm đáng tin cậy.

Hệ thống hướng tới việc gom các dữ liệu này về một nơi để việc quản trị và tra cứu nhất quán hơn.

---

## 3. Ứng dụng thực tế

### 3.1. Doanh nghiệp vừa và nhỏ

Có thể sử dụng để quản lý:

- Subnet.
- VLAN.
- Server.
- Router.
- Switch.
- Access Point.
- PC.
- Printer.
- Camera.
- Thiết bị IoT.
- IP tĩnh.
- DHCP lease.
- DNS record nội bộ.

### 3.2. Trường học / phòng Lab

Có thể quản lý:

- Phòng máy.
- Phòng Lab.
- Thiết bị mạng.
- Server nội bộ.
- IP cấp cho từng khu vực.
- VLAN theo phòng hoặc nhóm người dùng.

### 3.3. Homelab

Có thể quản lý:

- NAS.
- Raspberry Pi.
- ESP32.
- Virtual Machine.
- Home Server.
- Router.
- Access Point.
- Các dịch vụ nội bộ.

### 3.4. Troubleshooting

Khi xảy ra sự cố, Administrator có thể tra cứu nhanh:

```text
Device
  ↓
Interface
  ↓
IP
  ↓
Subnet
  ↓
VLAN
```

thay vì phải dò từng thiết bị hoặc từng file cấu hình riêng lẻ.

---

## 4. Giá trị chính của đề tài

Nhóm này không chỉ là một ứng dụng CRUD.

Phần có giá trị kỹ thuật nằm ở việc mô hình hóa đúng quan hệ giữa các tài nguyên mạng:

```text
Device
   |
   v
Interface
   |
   v
IP Allocation
   |
   v
Subnet
   |
   v
VLAN
```

Từ mô hình này có thể phát triển tiếp:

```text
IPAM
  ↓
DHCP / DNS
  ↓
Discovery
  ↓
Topology
  ↓
Monitoring
  ↓
Network Management System
```

Do đó đây là hướng phù hợp nếu muốn xây một project có vòng đời dài và có thể nâng cấp qua nhiều học kỳ.

---

## 5. Kiến trúc sơ bộ

```text
┌──────────────────────────────┐
│           Frontend           │
│ Dashboard / Device / IPAM    │
│ DNS/DHCP / Topology          │
└──────────────┬───────────────┘
               │ REST API
               ▼
┌──────────────────────────────┐
│            Backend           │
│                              │
│  ├── Inventory Module        │
│  ├── IPAM Module             │
│  ├── DNS/DHCP Module         │
│  ├── Discovery Module        │
│  ├── Topology Module         │
│  └── Basic Monitoring        │
└──────────────┬───────────────┘
               │
               ▼
        ┌──────────────┐
        │ PostgreSQL   │
        └──────────────┘
```

Trong đồ án cơ sở không cần tách Microservice. Có thể triển khai theo kiểu **Modular Monolith** để dễ phát triển và kiểm thử.

---

## 6. Data model sơ bộ

Phần lõi dự kiến:

```text
Device
  └── Interface
        └── IP Allocation
              └── Subnet
                    └── VLAN
```

Các module mở rộng có thể liên kết vào lõi:

```text
DHCP Lease
   └── IP / MAC / Device

DNS Record
   └── IP / Device

Topology Link
   └── Device / Interface
```

Điểm quan trọng là IPAM và Inventory phải được thiết kế đủ rõ ngay từ đầu để các module sau chỉ cần **tham chiếu hoặc bổ sung dữ liệu**, không phải tạo một hệ dữ liệu hoàn toàn khác.

---

## 7. Các module chức năng có thể có

### 7.1. IPAM

- CRUD Subnet.
- CIDR validation.
- Subnet overlap validation.
- Dynamic IP Pool.
- Reserve / Unreserve IP.
- Assign / Unassign IP.
- Tìm kiếm IP.
- Kiểm tra IP đã cấp hoặc còn trống.
- Quản lý VLAN cơ bản.

### 7.2. Device Inventory

- CRUD Device.
- CRUD Interface.
- Gắn IP vào Interface.
- Device Type.
- Location.
- MAC Address.
- Ghi chú.
- Search / Filter.

### 7.3. DNS Manager

Ở mức cơ bản:

- Xem DNS record nội bộ.
- Thêm/sửa/xóa record đơn giản.
- Liên kết hostname với IP đã có trong IPAM.

Không nhất thiết phải triển khai DNS server riêng trong đồ án cơ sở.

### 7.4. DHCP Manager

Ở mức nhẹ:

- Đọc DHCP lease.
- Hiển thị IP/MAC/hostname.
- Đối chiếu lease với Device Inventory.
- Phát hiện lease chưa có trong Inventory.

Giai đoạn cơ sở có thể chỉ đọc dữ liệu thay vì trực tiếp điều khiển DHCP server.

### 7.5. Topology

Ở mức ý tưởng:

- Phát hiện thiết bị active.
- Ghi nhận quan hệ thiết bị cơ bản.
- Vẽ graph đơn giản.

Độ chính xác của Topology phụ thuộc phương pháp Discovery, vì vậy phần này không nên là Core bắt buộc của đồ án cơ sở.

### 7.6. Basic Monitoring

- ICMP Ping.
- Online / Offline / Unknown.
- Last Check.
- Last Seen.

Monitoring chỉ hỗ trợ kiểm tra nhanh tình trạng Inventory, chưa phải NMS hoàn chỉnh.

---

## 8. Phạm vi đề xuất cho đồ án cơ sở ngành

### Nhóm A — Core bắt buộc

```text
IPAM
+
Device Inventory
+
Basic ICMP Monitoring
+
Dashboard
```

Bao gồm:

- CRUD Subnet.
- Overlap validation.
- Dynamic IP Pool.
- Reserve / Assign / Unassign IP.
- CRUD Device.
- CRUD Interface.
- Quan hệ Device → Interface → IP.
- VLAN cơ bản nếu cần.
- ICMP Monitoring.
- Search / Filter.
- Authentication Administrator.
- Dashboard.

### Nhóm B — Mở rộng nhẹ nếu còn thời gian

Chọn một hoặc hai phần:

- Đọc DHCP lease.
- CRUD DNS record nội bộ đơn giản.
- ARP/ICMP discovery nhỏ.
- Topology graph đơn giản.

### Nhóm C — Không làm trong đồ án cơ sở

- Full DHCP server management.
- SNMP polling nâng cao.
- LLDP/CDP topology đầy đủ.
- Traffic history.
- Auto-discovery liên tục.
- Alerting hoàn chỉnh.
- RBAC.
- IPv6/VRF phức tạp.

---

## 9. Kịch bản Demo cơ sở ngành

Ví dụ mạng Lab:

```text
192.168.10.0/24
        |
        +-- Router-R1
        +-- Server-01
        +-- PC-01
        +-- PC-02
        +-- IoT-01
```

Demo có thể gồm:

1. Login Administrator.
2. Tạo Subnet.
3. Hệ thống xác định Host Range.
4. Reserve một IP.
5. Tạo Device.
6. Tạo Interface.
7. Assign IP.
8. Thử cấp trùng IP để chứng minh Validation.
9. Hiển thị IP còn khả dụng.
10. Bật ICMP Monitoring.
11. Tắt một VM để thấy trạng thái Offline.
12. Bật lại VM để cập nhật Last Seen.
13. Nếu có DHCP extension: hiển thị Lease.
14. Nếu có DNS extension: tạo hostname nội bộ.
15. Nếu có Topology extension: hiển thị graph đơn giản.

---

## 10. Các quyết định cần chốt khi đi sâu

- Backend dùng Go hay Node.js.
- Cấu trúc Database chính thức.
- IP Pool lưu động hay pre-generate.
- Quy tắc Reserve / Assign / Unassign.
- Quan hệ VLAN – Subnet.
- DHCP chỉ Read-only hay có quyền ghi cấu hình.
- DNS sử dụng dịch vụ nào trong Lab.
- Discovery dùng ICMP, ARP hay SNMP.
- Topology chỉ mang tính minh họa hay phải phản ánh Layer 2.
- Monitoring Target gắn theo IP hay Device.
- Scope nào là Core và scope nào là Bonus.

Các điểm này chưa cần khóa ở giai đoạn ghi ý tưởng, nhưng phải được giải quyết trước khi bắt đầu Implementation.

---

## 11. Rủi ro và độ khó

### Độ khó tổng thể

**Trung bình → khá cao** nếu cố làm đủ IPAM + DHCP + DNS + Topology trong cùng một học kỳ.

### Rủi ro chính

- Scope dễ phình lớn.
- Topology Discovery phức tạp hơn tưởng tượng.
- DHCP/DNS integration phụ thuộc môi trường Lab.
- Data model sai từ đầu sẽ ảnh hưởng toàn hệ thống.
- Dễ biến thành nhiều module nửa hoàn thiện.

### Cách giảm rủi ro

Giữ nguyên nguyên tắc:

> Đồ án cơ sở làm chắc IPAM + Inventory trước. DNS/DHCP/Topology chỉ là extension hoặc proof of concept.

---

## 12. Nâng cấp thành đồ án chuyên ngành

Hướng nâng cấp tự nhiên nhất:

# Network Management System (NMS)

Kiến trúc có thể tiến hóa:

```text
Inventory / IPAM
       |
       +-- Discovery
       |
       +-- SNMP Monitoring
       |
       +-- DNS / DHCP Integration
       |
       +-- Topology
       |
       +-- Metrics History
       |
       +-- Alert
       |
       +-- Automation
```

Các chức năng chuyên ngành có thể gồm:

- SNMP polling.
- CPU / RAM / Uptime.
- Interface status.
- RX/TX traffic.
- Monitoring history.
- Time-series metrics.
- Auto-discovery.
- Phát hiện thiết bị mới.
- LLDP/CDP topology.
- Alert Telegram/Email.
- DHCP integration sâu hơn.
- DNS synchronization.
- Network configuration backup.
- API phục vụ automation.

---

## 13. Hướng phát triển xa hơn

Sau NMS có thể mở rộng thành:

```text
Network Source of Truth
       +
Monitoring
       +
Automation
       ↓
Network / Infrastructure Platform
```

Có thể tích hợp:

- Ansible.
- SSH Automation.
- Config Backup.
- Configuration Validation.
- Git-based Network Configuration.
- NetDevOps workflow.

Nhóm này có khả năng kết nối trực tiếp với Nhóm 6 trong giai đoạn xa hơn.

---

## 14. Giá trị đối với CV / Portfolio

Nhóm này phù hợp với các hướng:

- Network Administrator.
- System Administrator.
- Infrastructure Engineer.
- Network Engineer.
- NOC Engineer.
- NetDevOps ở giai đoạn mở rộng.

Một project hoàn chỉnh có thể chứng minh:

- Hiểu IPv4/CIDR/Subnetting.
- Hiểu mô hình Device/Interface/IP/VLAN.
- Biết thiết kế Database.
- Biết xây REST API.
- Biết xử lý Background Worker.
- Biết Monitoring.
- Biết xây Lab.
- Có khả năng thiết kế hệ thống mở rộng.

Nếu phát triển tiếp thành NMS và public project trên GitHub thì đây là hướng có tiềm năng portfolio rất tốt.

---

## 15. Đánh giá sơ bộ

| Tiêu chí | Đánh giá |
|---|---|
| Phù hợp đồ án cơ sở | Cao nếu giới hạn Core |
| Kiến thức Network | Cao |
| Backend/Database | Cao |
| Khả năng Demo | Cao |
| Độ khó | Trung bình – khá cao |
| Nguy cơ Scope Creep | Cao |
| Khả năng nâng chuyên ngành | Rất cao |
| Giá trị CV Network/Infra | Rất cao |
| Khả năng phát triển dài hạn | Rất cao |

---

## 16. Kết luận ý tưởng

Đây là hướng **cân bằng nhất giữa Networking, Backend, Database và Monitoring**.

Cấu trúc phát triển hợp lý:

```text
Đồ án cơ sở
IPAM + Inventory + Basic Monitoring

        ↓

Đồ án chuyên ngành
Full NMS

        ↓

Phát triển xa hơn
Network Source of Truth + Automation / NetDevOps
```

Nếu chọn nhóm này, điều quan trọng nhất là **không cố triển khai toàn bộ DNS/DHCP/Topology ở đồ án cơ sở**. IPAM và Inventory phải là nền móng chính; các module còn lại được thêm dần theo tiến độ.
