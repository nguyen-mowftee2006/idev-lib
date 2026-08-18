# HỒ SƠ ĐỒ ÁN — IPAM & NETWORK INVENTORY → NMS → NETWORK AUTOMATION

> **Mục đích của file**
>
> Đây là tài liệu bàn giao đầy đủ về hướng đồ án đã chốt. File được viết để có thể mang sang một phiên ChatGPT mới ngay cả khi toàn bộ lịch sử chat cũ đã bị xóa.
>
> Tài liệu chỉ tập trung vào **đồ án này**, gồm: mục tiêu, phạm vi, kiến trúc, mô hình dữ liệu, business rules, monitoring, bảo mật, testing, roadmap và hướng nâng cấp.
>
> Đây là **bản ghi thiết kế/ý tưởng hiện tại**, chưa phải báo cáo chính thức nộp giảng viên và chưa phải đặc tả implementation cuối cùng.

---

# 1. Bối cảnh và định hướng dài hạn

Đồ án được định hướng phát triển xuyên suốt ba giai đoạn:

```text
Năm 3 - Học kỳ 1
Đồ án cơ sở ngành
        ↓
Năm 3 - Học kỳ 2
Đồ án chuyên ngành
        ↓
Năm 4
Thực tập + Đồ án tốt nghiệp
```

Mục tiêu không phải làm ba project rời nhau.

Mỗi giai đoạn phải kế thừa nền tảng của giai đoạn trước:

```text
KNOW THE NETWORK
        ↓
OBSERVE THE NETWORK
        ↓
CONTROL THE NETWORK
```

Tương ứng:

```text
IPAM & Network Inventory
        ↓
Network Management System (NMS)
        ↓
Network Management & Automation Platform
```

Triết lý thiết kế xuyên suốt:

- Core phải đúng trước khi mở rộng.
- Không thêm chức năng chỉ để đề tài trông lớn.
- Không over-engineer đồ án cơ sở ngành.
- Database và business rules phải được thiết kế đủ sạch để nâng cấp về sau.
- Đồ án chuyên ngành phải kế thừa cơ sở, không viết lại từ đầu.
- Đồ án tốt nghiệp phải là bước nâng cấp về năng lực hệ thống, không chỉ thêm giao diện hoặc vài biểu đồ.
- Project cuối cùng phải có giá trị học tập và portfolio/CV thực tế.

---

# 2. Đề tài đồ án cơ sở ngành

## Tên đề xuất

### Tiếng Việt

**Hệ thống quản lý địa chỉ IP và kiểm kê tài nguyên mạng**

Nếu cần thể hiện monitoring trong tên:

**Hệ thống quản lý địa chỉ IP, kiểm kê và giám sát cơ bản thiết bị mạng**

### Tiếng Anh

**IPAM & Network Inventory Management System**

---

# 3. Bài toán thực tế

Trong một mạng nội bộ có nhiều thiết bị, việc quản lý bằng Excel hoặc ghi chép thủ công dễ phát sinh:

- Không biết IP nào đã được sử dụng.
- Không biết IP nào còn trống.
- Không biết IP đang được cấp cho thiết bị/interface nào.
- Cấp trùng IP.
- Khó quản lý nhiều subnet.
- Khó kiểm kê thiết bị.
- Khi thiết bị mất kết nối phải ping thủ công.
- Dữ liệu quản lý IP và dữ liệu thiết bị không có quan hệ rõ ràng.

Hệ thống giải quyết bài toán bằng cách tạo một nguồn dữ liệu tập trung:

```text
Device
   ↓
Interface
   ↓
IP Allocation
   ↓
Subnet
   ↓
VLAN
```

kết hợp với monitoring ICMP cơ bản.

---

# 4. Mục tiêu của đồ án cơ sở

Đồ án cơ sở tập trung vào ba khối:

```text
IPAM
+
Network Inventory
+
Basic ICMP Monitoring
```

Mục tiêu:

> Xây dựng một hệ thống trung tâm quản lý địa chỉ IPv4, subnet và thông tin thiết bị/interface trong mạng Lab, đồng thời kiểm tra trạng thái kết nối cơ bản của thiết bị bằng ICMP.

Câu hỏi mà hệ thống phải trả lời được:

- Có những subnet nào?
- Một subnet có bao nhiêu địa chỉ usable?
- IP nào đang available?
- IP nào reserved?
- IP nào assigned?
- IP được assigned cho interface nào?
- Interface thuộc thiết bị nào?
- Thiết bị đang ONLINE/OFFLINE/UNKNOWN?
- Lần kiểm tra gần nhất là khi nào?
- Lần cuối target phản hồi ICMP là khi nào?

---

# 5. Phạm vi đồ án cơ sở

## 5.1. Core bắt buộc

### IPAM

- CRUD Subnet.
- IPv4 CIDR validation.
- Tính Network Address.
- Tính Broadcast Address.
- Tính usable host range.
- Tính số usable IP.
- Kiểm tra subnet overlap.
- Validate khi thay đổi CIDR/prefix.
- Dynamic IP Pool.
- Reserve IP.
- Unreserve IP.
- Assign IP.
- Unassign IP.
- Ngăn cấp trùng IP.
- Database constraints và transaction.

### Network Inventory

- CRUD Device.
- CRUD Interface.
- Device có nhiều Interface.
- Một Interface tối đa một assigned IPv4 trong scope cơ sở.
- IP Allocation liên kết với Interface.
- Có thể lưu Device Type, Location, Description.
- Thiết kế quan hệ VLAN/Subnet.

### Monitoring

- ICMP Ping.
- Monitoring Check riêng.
- Monitoring Target là một assigned IP Allocation cụ thể.
- Trạng thái UNKNOWN / ONLINE / OFFLINE.
- Retry threshold.
- Last Check.
- Last Seen.
- Background scheduler.
- Bounded worker pool.
- Job deduplication.
- Stale-result protection.

### System

- REST API.
- PostgreSQL.
- Authentication một Administrator.
- Dashboard.
- Search/filter.
- Testing.

---

## 5.2. Làm nếu còn thời gian

- VLAN CRUD/UI hoàn chỉnh.
- Filter nâng cao.
- Basic ICMP Discovery.
- Unknown active IP detection.
- Cải thiện UI.

---

## 5.3. Không làm ở đồ án cơ sở

Các phần sau **cố ý để dành**:

- SNMP monitoring.
- Traffic history.
- Time-series metrics dài hạn.
- Advanced network discovery.
- LLDP/CDP.
- Automatic topology.
- DNS Management.
- DHCP Management.
- Full Alert System.
- RBAC/multi-user.
- IPv6.
- VRF.
- Multi-IP per Interface.
- Ansible.
- SSH automation.
- Network configuration deployment.
- Configuration backup/versioning.
- Rollback.

Lý do: đồ án cơ sở phải làm chắc IPAM/Inventory thay vì nhồi quá nhiều module.

---

# 6. Stack công nghệ định hướng

```text
Backend      : Go
Frontend     : React
Database     : PostgreSQL
API          : REST
Architecture : Modular Monolith
Versioning   : Git / GitHub
Environment  : Lab / Homelab / VM / GNS3 / EVE-NG
```

Không dùng Microservices ở đồ án cơ sở.

Kiến trúc backend nên chia domain:

```text
Backend
├── Auth
├── Inventory
├── IPAM
└── Monitoring
```

Business logic không dồn vào REST controller.

---

# 7. Mô hình tài nguyên mạng

Quan hệ lõi:

```text
Device
   │
   ├── Interface 1
   │      └── IP Allocation
   │
   ├── Interface 2
   │      └── IP Allocation
   │
   └── ...
```

IP Allocation thuộc:

```text
Subnet
```

Subnet có thể thuộc:

```text
VLAN
```

Tổng thể:

```text
Device
   ↓ 1:N
Interface
   ↓ 0:1 trong scope cơ sở
Assigned IP Allocation
   ↓
Subnet
   ↓ optional
VLAN
```

Không thiết kế kiểu:

```text
Device.ip_address
```

vì thiết bị thực tế có thể có nhiều interface.

---

# 8. Conceptual database schema

Schema dưới đây là mô hình conceptual, chưa phải migration cuối cùng.

## users

```text
id
username
password_hash
created_at
```

## devices

```text
id
name
type
location
description
created_at
updated_at
```

## device_interfaces

```text
id
device_id
name
mac_address
created_at
updated_at
```

## vlans

```text
id
vlan_id
name
description
```

## subnets

```text
id
vlan_id
network
prefix_length
description
created_at
updated_at
```

## ip_allocations

```text
id
subnet_id
address
status
interface_id
description
created_at
updated_at
```

Persisted status:

```text
reserved
assigned
```

`available` không cần lưu thành row.

## monitoring_checks

Conceptual:

```text
id
device_id
target_ip_allocation_id
type
enabled
status
last_check
last_seen
consecutive_failures
created_at
updated_at
```

Trong đồ án cơ sở:

```text
type = ICMP
```

Một Device tối đa một main ICMP Monitoring Check.

---

# 9. Dynamic IP Pool

Đây là quyết định kiến trúc quan trọng.

## Không pre-generate toàn bộ IP

Ví dụ subnet:

```text
10.0.0.0/8
```

có số lượng địa chỉ rất lớn.

Không được tạo hàng triệu record chỉ để đánh dấu:

```text
available
```

Database chỉ lưu IP đã thực sự bị chiếm dụng:

```text
reserved
assigned
```

`available` là trạng thái suy ra.

Công thức:

```text
Available IP
=
Usable Host Range
-
Reserved IP
-
Assigned IP
```

Ví dụ:

```text
Subnet: 192.168.10.0/24

Network   : 192.168.10.0
Broadcast : 192.168.10.255

Usable:
192.168.10.1
...
192.168.10.254
```

Nếu DB chỉ có:

```text
192.168.10.10 reserved
192.168.10.20 assigned
```

thì các host address usable còn lại được xem là available.

---

# 10. IP lifecycle

State machine:

```text
available ── reserve ───> reserved
available ── assign ────> assigned

reserved  ── unreserve ─> available
reserved  ── assign ────> assigned

assigned  ── unassign ──> available
```

## Available

Không có allocation row persisted.

## Reserved

Có allocation row:

```text
status = reserved
interface_id = NULL
```

## Assigned

Có allocation row:

```text
status = assigned
interface_id != NULL
```

---

# 11. Reserved → Assigned

Reserved IP có thể được assign trực tiếp cho Interface.

Không bắt buộc:

```text
reserved
→ available
→ assigned
```

Cho phép:

```text
reserved
→ assigned
```

Khi đó reservation được xem là đã tiêu thụ.

Nếu sau đó Unassign:

```text
assigned
→ available
```

không tự quay lại `reserved`.

---

# 12. Validation khi Reserve và Assign

Reserve và Assign phải dùng chung validation IP cơ bản.

IP phải:

1. Thuộc subnet.
2. Nằm trong usable host range.
3. Không phải Network Address.
4. Không phải Broadcast Address.
5. Chưa bị allocation theo cách gây conflict.
6. Không vi phạm DB constraint.

Sau validation chung mới kiểm tra state transition cụ thể.

Không được xảy ra:

```text
Reserve Network Address
Reserve Broadcast Address
Assign Network Address
Assign Broadcast Address
```

---

# 13. IPv4 scope

Đồ án cơ sở định hướng:

```text
IPv4
Prefix /1 đến /30
```

Không hỗ trợ trong kỳ cơ sở:

```text
/31
/32
IPv6
VRF
```

`/31` và `/32` có semantics đặc biệt và không cần thiết cho scope ban đầu.

IPv6 và VRF để dành cho giai đoạn nâng cấp.

---

# 14. Subnet overlap

Vì chưa hỗ trợ VRF/routing domain:

> Hai subnet IPv4 không được overlap trong toàn hệ thống.

Ví dụ đã có:

```text
192.168.10.0/24
```

phải reject:

```text
192.168.10.0/25
192.168.10.128/25
192.168.10.0/23
```

Cho phép:

```text
192.168.11.0/24
```

Overlap phải được kiểm tra cả khi:

```text
CREATE Subnet
```

và:

```text
UPDATE/Resize Subnet
```

---

# 15. Resize Subnet

Ví dụ:

```text
Old:
192.168.10.0/24
```

Admin đổi thành:

```text
192.168.10.0/25
```

Trước khi update:

1. Validate CIDR mới.
2. Re-check overlap với các subnet khác.
3. Kiểm tra toàn bộ assigned/reserved allocations.
4. Tất cả allocation phải còn nằm trong usable range mới.
5. Nếu có conflict → reject.

Ví dụ đang có:

```text
192.168.10.200 assigned
```

thì không được shrink xuống `/25`.

Có thể trả:

```text
409 Conflict
```

---

# 16. Device và Interface

Một Device:

```text
1 Device
→ N Interface
```

Ví dụ:

```text
Router-R1
├── eth0
└── eth1
```

Scope cơ sở:

```text
1 Interface
→ tối đa 1 assigned IPv4
```

Đây là **scope simplification**, không phải giới hạn dài hạn.

Sau này có thể nâng:

```text
1 Interface
→ N IP Address
→ IPv4 + IPv6
```

---

# 17. VLAN

Quan hệ đề xuất:

```text
1 VLAN
→ N Subnet

1 Subnet
→ 0 hoặc 1 VLAN
```

VLAN có thể chứa:

```text
vlan_id
name
description
```

VLAN là module network hữu ích nhưng ưu tiên thấp hơn IPAM core.

Nếu thiếu thời gian:

> Hoàn thiện Subnet/IP Allocation/Device/Interface trước VLAN UI.

---

# 18. Delete rules

Không dùng cascade delete một cách tùy tiện.

## Delete Device

Trong transaction:

```text
1. Xử lý Monitoring Check
2. Release assigned IP của Interfaces
3. Delete Interfaces
4. Delete Device
```

## Delete Interface

Nếu Interface đang có assigned IP:

```text
1. Nếu IP là Monitoring Target → clear/reset monitoring
2. Unassign/release IP
3. Delete Interface
```

## Delete Subnet

Nếu còn:

```text
reserved allocation
hoặc
assigned allocation
```

→ block delete.

Admin phải release/unreserve trước.

Có thể trả:

```text
409 Conflict
```

## Delete VLAN

Nếu còn Subnet reference VLAN:

```text
409 Conflict
```

## Quan hệ xóa Subnet/VLAN

Có chủ ý bất đối xứng:

```text
Delete Subnet
→ VLAN không phải lý do block

Delete VLAN
→ block nếu Subnet còn reference
```

---

# 19. Database integrity

Không chỉ dựa vào backend validation.

Mô hình:

```text
Application Validation
        ↓
Database Transaction
        ↓
UNIQUE / CHECK / FK
        ↓
Commit
```

Các constraint cần xem xét:

- `users.username` UNIQUE.
- Không có duplicate allocated IP.
- Một Interface tối đa một assigned IPv4 trong scope cơ sở.
- `reserved` → `interface_id IS NULL`.
- `assigned` → `interface_id IS NOT NULL`.
- Foreign Key đầy đủ.
- Không để dangling monitoring target.

---

# 20. Race condition khi Assign IP

Case:

```text
Request A → check IP X → available
Request B → check IP X → available
```

sau đó cả hai cùng insert.

Nếu chỉ dùng application validation có thể cấp trùng IP.

Do đó:

```text
Transaction
+
Database Constraint
```

phải là lớp bảo vệ cuối cùng.

Kết quả mong muốn:

```text
A → success
B → conflict
```

hoặc ngược lại.

Không bao giờ cả hai success.

---

# 21. Basic Monitoring

Đồ án cơ sở **chỉ dùng ICMP**.

Không SNMP.

Monitoring là module phụ để chứng minh hệ thống tương tác với mạng thật, không chỉ quản lý dữ liệu tĩnh.

---

# 22. Monitoring Target

Thiết kế cũ kiểu “Primary Interface” không được dùng.

Lý do:

```text
Interface
```

không nhất thiết đồng nghĩa với một IP duy nhất trong kiến trúc dài hạn.

Thiết kế đúng:

```text
Device
   ↓
Monitoring Check
   ↓
Target IP Allocation
```

Trong đồ án cơ sở:

```text
1 Device
→ tối đa 1 main ICMP Monitoring Check
→ target đúng 1 assigned IPv4
```

---

# 23. Monitoring Target invariants

Target phải:

1. Là IP Allocation tồn tại.
2. `status = assigned`.
3. Có `interface_id`.
4. Interface đó thuộc chính Device của Monitoring Check.

Không cho:

```text
Device A
→ Monitoring Check
→ IP của Device B
```

Reserved IP không được làm Monitoring Target.

---

# 24. Monitoring states

```text
UNKNOWN
ONLINE
OFFLINE
```

## UNKNOWN

Có thể xảy ra khi:

- Chưa monitor lần nào.
- Chưa có target.
- Target vừa thay đổi.
- Target vừa bị xóa/unassign.
- Monitoring chưa đủ dữ liệu.

## ONLINE

Target trả lời ICMP thành công.

## OFFLINE

Target không trả lời đủ số lần liên tiếp theo threshold.

Quan trọng:

> OFFLINE chỉ có nghĩa ICMP target không phản hồi/reachable theo phương pháp check hiện tại. Không tuyệt đối khẳng định thiết bị đã tắt nguồn vì ICMP có thể bị firewall/filter.

---

# 25. Monitoring fields

```text
status
last_check
last_seen
consecutive_failures
```

## last_check

Thời điểm monitoring attempt gần nhất.

## last_seen

Thời điểm gần nhất ICMP thành công.

Không diễn giải `last_seen` là “lần cuối hệ thống thấy bất kỳ traffic nào của thiết bị”.

---

# 26. Retry / False Positive

Không mark OFFLINE chỉ sau một failure.

Default đề xuất:

```text
OFFLINE_THRESHOLD=3
```

Success:

```text
status = ONLINE
consecutive_failures = 0
last_check = now
last_seen = now
```

Failure:

```text
consecutive_failures += 1
last_check = now
```

Nếu:

```text
consecutive_failures >= OFFLINE_THRESHOLD
```

thì:

```text
status = OFFLINE
```

---

# 27. Monitoring configuration

Global config:

```text
MONITOR_INTERVAL=60s
PING_TIMEOUT=2s
OFFLINE_THRESHOLD=3
MONITOR_WORKERS=20
```

Các giá trị nên lấy từ config/environment.

Không hardcode trong logic.

Per-device interval để dành cho giai đoạn sau.

---

# 28. Monitoring architecture

Không ping trong REST request.

Không:

```text
GET /devices
→ ping tất cả
```

Kiến trúc:

```text
Scheduler
   ↓
Job Queue
   ↓
Bounded Worker Pool
   ↓
ICMP Target
   ↓
Monitoring Result
   ↓
Database
```

Lý do:

Nếu:

```text
200 devices × timeout 2s
```

và ping sequential thì cycle có thể kéo dài hàng trăm giây.

Do đó phải có concurrency có giới hạn.

---

# 29. Cycle overlap

Rule đơn giản cho đồ án cơ sở:

> Không bắt đầu monitoring cycle mới nếu cycle trước vẫn chưa xử lý xong.

Không cần distributed scheduler/lock.

---

# 30. Immediate Check

Không trigger immediate check sau mọi lần Assign IP.

Chỉ cần khi:

- Tạo Monitoring Target.
- Thay đổi Monitoring Target.

Immediate Check và Scheduled Check dùng chung queue.

---

# 31. Job deduplication

Một Monitoring Check chỉ có tối đa:

```text
1 active job
```

tại một thời điểm.

Nếu scheduled cycle và immediate check cùng muốn enqueue một check:

```text
job already active
→ skip duplicate
```

Không cần distributed lock trong đồ án cơ sở.

---

# 32. Stale Monitoring Result

Case:

```text
Worker bắt đầu ping IP A
        ↓
Admin đổi target sang IP B
        ↓
Ping A trả kết quả
```

Không được ghi kết quả của A vào state hiện tại của B.

Job snapshot cần đủ thông tin, ví dụ:

```text
device_id
check_id
target_allocation_id
target_ip
```

Trước khi persist:

```text
Current Target == Job Target ?
```

Nếu không:

```text
discard stale result
```

Không update:

- status.
- last_seen.
- consecutive_failures.

---

# 33. Khi Target bị Unassign/Delete

Nếu assigned IP đang được Monitoring Check dùng làm target:

Trước khi release:

```text
1. Clear target
2. Reset Monitoring State
3. Release allocation
```

Reset:

```text
status = UNKNOWN
consecutive_failures = 0
last_check = NULL
last_seen = NULL
```

Không để FK/dangling reference.

---

# 34. Authentication

Scope cơ sở:

```text
1 Administrator
```

Không:

- Public registration.
- Multi-user.
- RBAC.

Ưu tiên:

```text
Session Authentication
+
HttpOnly Cookie
```

thay vì JWT access/refresh phức tạp không cần thiết.

---

# 35. Password và Bootstrap

Password chỉ lưu:

```text
password_hash
```

Không plaintext.

Admin đầu tiên tạo bằng:

- Seed command.
- Bootstrap command.
- Environment/config.

Bootstrap phải idempotent:

```text
if admin exists
    skip
else
    hash password
    create admin
```

DB có:

```text
UNIQUE(username)
```

---

# 36. CSRF

Vì dùng cookie-based session, các request thay đổi state:

```text
POST
PUT
PATCH
DELETE
```

phải có CSRF protection.

Ví dụ:

```text
X-CSRF-Token
```

Cookie cần cân nhắc:

- HttpOnly.
- SameSite.
- Secure khi HTTPS.

---

# 37. REST API conceptual

Có thể tổ chức:

```text
/auth
/devices
/interfaces
/vlans
/subnets
/ip-allocations
/monitoring
/dashboard
```

Exact contract chưa khóa.

Điểm quan trọng:

> API phải phân biệt persisted allocation và computed available address.

Không giả vờ toàn bộ IP available đều là row trong database.

---

# 38. Error format

Có thể chuẩn hóa:

```json
{
  "error": {
    "code": "IP_ALREADY_ALLOCATED",
    "message": "The IP address is already allocated.",
    "details": {}
  }
}
```

Possible codes:

```text
INVALID_CIDR
SUBNET_OVERLAP
SUBNET_RESIZE_CONFLICT
SUBNET_HAS_ALLOCATIONS
IP_OUTSIDE_SUBNET
IP_ALREADY_ALLOCATED
IP_NOT_ASSIGNABLE
VLAN_HAS_SUBNETS
DEVICE_NOT_FOUND
INTERFACE_NOT_FOUND
MONITORING_TARGET_INVALID
AUTH_INVALID_CREDENTIALS
CSRF_INVALID_TOKEN
```

HTTP status thường dùng:

```text
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
500 Internal Server Error
```

---

# 39. Frontend

React.

Các trang tối thiểu:

```text
Login
Dashboard
Subnets
IP Allocations / Addresses
Devices
Interfaces
Monitoring
VLAN (nếu module hoàn thành)
```

UI chỉ cần:

- Rõ.
- Sạch.
- Dễ demo.
- Không lỗi.

Không dành quá nhiều thời gian cho animation hoặc UI polish không phục vụ chức năng.

---

# 40. Dashboard

Ví dụ:

```text
Subnets       : 5
Usable IP     : 1260
Assigned      : 48
Reserved      : 12
Available     : 1200

Devices       : 35
Online        : 30
Offline       : 4
Unknown       : 1
```

Search/filter có thể theo:

- Device Name.
- IP.
- Status.
- Subnet.
- VLAN.

---

# 41. Basic Discovery — Bonus

Nếu core hoàn thành sớm, extension đáng ưu tiên nhất:

```text
Subnet
   ↓
ICMP Sweep
   ↓
Active IP
   ↓
Compare Inventory
```

Ví dụ:

```text
192.168.10.20 → Registered
192.168.10.21 → Registered
192.168.10.35 → Unknown
```

Ý nghĩa:

> Bắt đầu nối dữ liệu khai báo trong Inventory với dữ liệu quan sát trực tiếp từ network.

Nhưng:

**Basic Discovery không phải chức năng bắt buộc của đồ án cơ sở.**

---

# 42. Testing

Testing là phần bắt buộc của thiết kế, không để đến cuối mới nghĩ.

## Unit Tests

### CIDR

Input:

```text
192.168.1.0/24
```

Expected:

```text
Network   = 192.168.1.0
Broadcast = 192.168.1.255
Usable    = 254
```

### IP Validation

```text
192.168.1.20  → valid
192.168.1.0   → network address → reject
192.168.1.255 → broadcast → reject
10.0.0.1      → outside subnet → reject
```

### Overlap

Existing:

```text
192.168.1.0/24
```

Tests:

```text
192.168.1.0/25   → reject
192.168.1.128/25 → reject
192.168.2.0/24   → accept
```

---

# 43. Integration Tests

- Create Subnet.
- Reserve IP.
- Unreserve IP.
- Create Device.
- Create Interface.
- Assign IP.
- Unassign IP.
- Assign duplicate IP.
- Resize subnet.
- Delete Interface đang có IP.
- Delete Subnet đang có allocation.
- Delete VLAN đang có Subnet.
- Create Monitoring Check.
- Change Monitoring Target.
- Remove Monitoring Target.
- Login/logout.
- CSRF.
- DB constraint violation.

---

# 44. Monitoring System Tests

```text
VM running
→ ICMP success
→ ONLINE
```

Shutdown:

```text
failure 1
failure 2
failure 3
→ OFFLINE
```

Start lại:

```text
ICMP success
→ ONLINE
→ last_seen updated
```

---

# 45. Concurrency Tests

## Duplicate IP Assignment

Hai requests cùng assign một IP:

```text
A ─┐
   ├── 192.168.10.20
B ─┘
```

Expected:

```text
1 success
1 conflict
```

## Monitoring duplicate job

```text
Scheduled Check
+
Immediate Check
```

Expected:

```text
chỉ 1 active job/check
```

## Stale result

```text
Ping target A
→ target đổi B
→ A trả kết quả
```

Expected:

```text
A result discarded
```

---

# 46. Non-functional scope

Môi trường:

```text
Lab / Private Network
```

Demo target:

```text
5–10 subnets
20–50 devices/endpoints
```

Design target ban đầu:

```text
khoảng 100–200 devices
```

Đây chỉ là **design target**, không claim benchmark nếu chưa đo.

Không được viết:

```text
Response time < 1s
```

nếu chưa benchmark.

---

# 47. Lab

Không kết nối vào mạng doanh nghiệp thật.

Có thể test bằng:

- Homelab.
- Linux VM.
- Windows VM.
- GNS3.
- EVE-NG.
- Thiết bị mạng thật nếu có.

Cách trình bày:

> Hệ thống được thiết kế hướng tới mạng doanh nghiệp/tổ chức nhưng được kiểm thử trong môi trường mạng Lab quy mô nhỏ.

Không claim production enterprise deployment.

---

# 48. Kịch bản demo đồ án cơ sở

Một demo có thể đi theo thứ tự:

```text
1. Login Admin
2. Create Subnet 192.168.10.0/24
3. Hệ thống tính Network/Broadcast/Usable Range
4. Reserve một IP
5. Create Device
6. Create Interface
7. Assign một IP cho Interface
8. Thử assign IP trùng → reject
9. Xem số IP available
10. Create ICMP Monitoring Check
11. Chọn assigned IP làm target
12. Device → ONLINE
13. Shutdown VM
14. Sau threshold → OFFLINE
15. Start VM
16. Device → ONLINE
17. last_seen update
18. Unassign IP
19. IP trở lại derived available
20. Monitoring target được clear/reset đúng rule
21. Search/filter
```

Nếu có Discovery:

```text
22. Scan subnet
23. Hiển thị Registered / Unknown active IP
```

---

# 49. Giá trị kỹ thuật khi bảo vệ

Không nên tập trung nói:

```text
Em dùng Go.
Em dùng React.
Em dùng PostgreSQL.
```

Đó chỉ là technology stack.

Các bài toán kỹ thuật cần nhấn mạnh:

```text
Network Resource Modeling
CIDR/Subnet Calculation
Subnet Overlap Detection
IP Allocation Lifecycle
Database Integrity
Transaction & Concurrency
Background Monitoring
Monitoring State Consistency
```

Đây là những phần chứng minh đề tài không chỉ là CRUD web.

---

# 50. Roadmap cơ sở ngành gợi ý

Nếu có khoảng 14 tuần:

## Tuần 1

- Requirements.
- Scope.
- Use Cases.

## Tuần 2

- Business Rules.
- State Machines.
- ERD.
- DB Constraints.
- API Contract.

## Tuần 3

- Subnet CRUD.
- CIDR.
- Overlap.

## Tuần 4

- Dynamic IP Pool.
- Reserve/Unreserve.

## Tuần 5

- Device.
- Interface.

## Tuần 6

- Assign/Unassign.
- Transaction.
- Constraints.

## Tuần 7

- Auth.
- Session.
- CSRF.
- Bootstrap.

## Tuần 8

- Monitoring Check.
- ICMP.

## Tuần 9

- Worker Pool.
- Retry.
- Job dedupe.
- Stale result.

## Tuần 10

- VLAN hoặc hoàn thiện core.

## Tuần 11

- Dashboard/UI.

## Tuần 12

- Frontend/backend integration.

## Tuần 13

- Testing.
- Bug fixing.

## Tuần 14

- Report.
- Slides.
- Demo.

Frontend cơ bản có thể phát triển song song từ sớm.

---

# 51. Đồ án chuyên ngành — Năm 3 kỳ 2

Đồ án chuyên ngành kế thừa toàn bộ IPAM/Inventory core.

Tên định hướng:

# Network Management System (NMS)

hoặc:

**Hệ thống quản lý và giám sát mạng**

Mục tiêu chuyển từ:

```text
KNOW THE NETWORK
```

sang:

```text
OBSERVE THE NETWORK
```

---

# 52. Những gì được kế thừa

Không viết lại:

```text
Device
Interface
Subnet
VLAN
IP Allocation
Authentication
REST API Core
Database Core
```

IPAM/Inventory trở thành:

> **Network Source of Truth / Inventory Core**

cho NMS.

---

# 53. Core chuyên ngành

Đề xuất:

```text
IPAM / Inventory
        +
Network Discovery
        +
SNMP Monitoring
        +
Metrics
        +
Monitoring History
        +
Alert
        +
Advanced Dashboard
```

Topology là extension mạnh nếu còn thời gian.

---

# 54. Network Discovery

Từ Basic ICMP Discovery nâng thành:

```text
Discovery Engine
├── ICMP
├── ARP / MAC
├── SNMP
└── Device Identification
```

Phân loại:

```text
Known Device
New Device
Unknown Device
Missing Device
```

Ví dụ:

```text
192.168.10.35 active
        ↓
Không tồn tại trong Inventory
        ↓
UNKNOWN DEVICE
```

---

# 55. SNMP Monitoring

Đây nên là một trọng tâm kỹ thuật của chuyên ngành.

Có thể thu:

```text
sysName
sysDescr
sysUpTime
Interface Status
Traffic RX
Traffic TX
```

Nếu thiết bị hỗ trợ:

```text
CPU
RAM
```

Luồng:

```text
Inventory
   ↓
SNMP Poller
   ↓
Metrics
   ↓
Storage
   ↓
Dashboard
```

---

# 56. Monitoring History

Cơ sở:

```text
Current State
```

Chuyên ngành:

```text
09:00 ONLINE
09:05 ONLINE
09:10 OFFLINE
09:15 OFFLINE
09:20 ONLINE
```

Từ đó tính:

- Availability.
- Downtime.
- Latency history.
- Traffic history.

---

# 57. Metrics storage

Inventory/config data tiếp tục dùng:

```text
PostgreSQL
```

Metrics dài hạn có thể cân nhắc:

```text
Prometheus
hoặc
InfluxDB
```

Không cần tự xây Time-Series Database.

---

# 58. Alert

Có thể alert:

```text
Device Offline
Interface Down
Unknown Device
Threshold exceeded
```

Notification:

```text
Telegram
Email
Webhook
```

Nếu scope cho phép:

```text
OPEN
ACKNOWLEDGED
RESOLVED
```

Không cần full enterprise alert engine.

---

# 59. Topology

Nếu làm:

Không chỉ dựa vào ARP để suy ra Layer 2 topology.

Hướng tốt hơn:

```text
SNMP
+
LLDP/CDP
   ↓
Neighbor / Interface Relationship
   ↓
Topology Graph
```

Ví dụ:

```text
Router
  |
Core Switch
 /        \
SW-01    SW-02
 |         |
PCs      Servers
```

Topology có thể là bonus nếu core NMS chưa hoàn thiện.

---

# 60. DHCP/DNS

Không phải core cơ sở.

Nếu sau này tích hợp, mục tiêu nên là integration thật:

```text
IPAM
├── Static Allocation
├── DHCP Lease
└── DNS Record
```

Ví dụ:

```text
IPAM:
IP A → Server01

DHCP:
IP A → MAC khác

→ Conflict / Anomaly
```

Không nên thêm DNS/DHCP chỉ để có thêm CRUD.

---

# 61. Scope chuyên ngành đề xuất

## Core

- IPAM/Inventory Core.
- Discovery.
- SNMP.
- Metrics.
- History.
- Alert.
- Advanced Dashboard.

## Bonus

Ưu tiên chọn một:

```text
Topology
```

hoặc:

```text
DHCP/DNS Integration
```

Không cố làm cả hai nếu core chưa chắc.

---

# 62. Hình dạng hệ thống sau chuyên ngành

```text
Network Management System

├── Inventory
│   ├── Device
│   ├── Interface
│   ├── VLAN
│   └── Location
│
├── IPAM
│   ├── Subnet
│   └── IP Allocation
│
├── Discovery
│   ├── ICMP
│   ├── ARP
│   └── SNMP
│
├── Monitoring
│   ├── ICMP
│   └── SNMP
│
├── Metrics
│   ├── Uptime
│   ├── Interface
│   └── Traffic
│
├── History
├── Alert
├── Topology (optional)
└── Dashboard
```

---

# 63. Năm 4 — Thực tập

Thực tập được dùng để đối chiếu project với thực tế doanh nghiệp.

Các vấn đề nên quan sát:

- Doanh nghiệp quản lý IP bằng gì?
- Excel, NetBox, phpIPAM hay hệ thống nội bộ?
- Monitoring dùng Zabbix/LibreNMS/Prometheus hay giải pháp nào?
- Quản lý config thiết bị thế nào?
- Backup config ra sao?
- Network change có approval không?
- Có Git/Ansible không?
- Alert được xử lý theo quy trình nào?
- Có Source of Truth không?
- Automation được áp dụng đến đâu?

Những bài toán thực tế này sẽ quyết định scope tốt nghiệp chính xác hơn.

---

# 64. Đồ án tốt nghiệp — hướng dài hạn

Không chỉ làm:

```text
NMS v2 + thêm chart
```

Hướng đề xuất:

# **Network Management & Automation Platform**

Mục tiêu:

```text
OBSERVE
+
MANAGE
+
AUTOMATE
```

Tức chuyển từ:

```text
Mạng đang thế nào?
```

sang:

```text
Làm thế nào quản lý và thay đổi mạng tự động, có kiểm soát và có rollback?
```

---

# 65. Kiến trúc dài hạn

```text
                     Network Platform
                            |
        ┌───────────────────┼───────────────────┐
        │                   │                   │
    Source of Truth      Monitoring         Automation
        │                   │                   │
      IPAM                SNMP                Ansible
     Device              Metrics               SSH
     VLAN                History             Templates
  Interfaces              Alert               Deploy
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                         Network
```

---

# 66. Configuration Management

Có thể thêm:

```text
Network Device
      ↓
Backup Running Config
      ↓
Versioning
      ↓
Config Diff
      ↓
Restore / Rollback
```

---

# 67. Network Automation

Ví dụ:

```text
Admin:
"Tạo VLAN 30 trên 5 switch"
```

Flow:

```text
Source of Truth
      ↓
Generate Configuration
      ↓
Automation Engine
      ↓
Deploy
      ↓
Validate
```

Automation có thể dùng:

```text
Ansible
SSH
Netmiko/NAPALM hoặc công cụ phù hợp
```

Việc chọn công cụ cuối cùng để sau.

---

# 68. Validation và Rollback

Sau deployment:

```text
Ping
Interface State
Routing
VLAN existence
Expected configuration
```

Nếu:

```text
Validation Fail
```

thì:

```text
Rollback
```

Đây là bước biến project từ monitoring thành management/automation platform.

---

# 69. Git / NetDevOps

Hướng dài hạn:

```text
Change Request
     ↓
Config / Template
     ↓
Git
     ↓
Review
     ↓
Deploy
     ↓
Validate
```

Có thể phát triển thêm Configuration as Code.

---

# 70. Compliance / Configuration Drift

Ví dụ policy:

```text
Tất cả switch:
- phải có NTP
- phải dùng SSH
- không dùng Telnet
- phải có đúng Management VLAN
```

System có thể:

```text
Expected State
      vs
Actual State
      ↓
Drift Detection
```

Đây là hướng tốt cho đồ án tốt nghiệp nếu scope phù hợp.

---

# 71. Ba giai đoạn cuối cùng

## Giai đoạn 1 — Cơ sở ngành

# KNOW THE NETWORK

```text
IPAM
+
Inventory
+
Basic ICMP Monitoring
```

Câu hỏi:

> Mạng của tôi có gì?

---

## Giai đoạn 2 — Chuyên ngành

# OBSERVE THE NETWORK

```text
NMS
+
Discovery
+
SNMP
+
Metrics
+
History
+
Alert
```

Câu hỏi:

> Mạng của tôi đang hoạt động như thế nào?

---

## Giai đoạn 3 — Tốt nghiệp

# CONTROL THE NETWORK

```text
NMS
+
Configuration Management
+
Automation
+
Validation
+
Rollback
+
NetDevOps
```

Câu hỏi:

> Tôi có thể quản lý và thay đổi mạng một cách tự động, an toàn và có kiểm soát như thế nào?

---

# 72. Giá trị CV/Portfolio

Nếu hoàn thành đúng roadmap:

## Sau đồ án cơ sở

Có thể chứng minh:

- IPv4/CIDR.
- Subnetting.
- IPAM.
- Network Inventory.
- REST API.
- PostgreSQL.
- Transactions.
- Background workers.
- ICMP monitoring.

## Sau chuyên ngành

Thêm:

- SNMP.
- Network Discovery.
- NMS architecture.
- Metrics.
- Monitoring history.
- Alert.
- Topology nếu có.

## Sau tốt nghiệp

Thêm:

- Network Automation.
- Configuration Management.
- Git workflow.
- Ansible/SSH automation.
- Validation.
- Rollback.
- NetDevOps.

Hướng nghề nghiệp phù hợp:

```text
Network Administrator
System Administrator
Infrastructure Engineer
NOC Engineer
Network Engineer
Network Automation Engineer
NetDevOps
```

---

# 73. Các quyết định thiết kế đã loại bỏ

Khi tiếp tục project, không tự quay lại các thiết kế này nếu không có lý do mới rõ ràng.

## Không pre-generate IP Pool

Sai hướng:

```text
Subnet /16
→ tạo 65k IP rows
```

Đã thay bằng Dynamic IP Pool.

## Không dùng Primary Interface làm Monitoring Target

Sai vì Interface về dài hạn có thể có nhiều IP.

Đã thay bằng:

```text
Monitoring Check
→ Target IP Allocation
```

## Không SNMP ở cơ sở

SNMP thuộc chuyên ngành.

## Không nhồi DNS/DHCP/Topology vào cơ sở

Để Future Work/chuyên ngành.

## Không Microservices

Modular Monolith phù hợp hơn.

## Không ping trực tiếp trong REST request

Monitoring chạy background.

## Không mark OFFLINE sau một failure

Dùng retry threshold.

## Không dựa hoàn toàn vào backend validation

Dùng DB constraints + transaction.

---

# 74. Các điểm CHƯA khóa hoàn toàn

Các mục sau cần bàn tiếp trước khi implementation cuối cùng.

## 74.1. Canonical subnet input

Nếu Admin nhập:

```text
192.168.1.10/24
```

chọn một:

### Option A — Normalize

```text
192.168.1.0/24
```

### Option B — Reject

Yêu cầu host bits = 0.

Chưa chốt.

---

## 74.2. Concurrent Subnet Creation

Hai request tạo subnet overlap cùng lúc có thể cùng pass application check.

Cần chọn chiến lược PostgreSQL phù hợp:

- Transaction locking.
- Advisory lock.
- Serialized subnet write.
- Exclusion strategy.
- Giải pháp khác phù hợp.

Chưa khóa.

---

## 74.3. Available IP API cho subnet cực lớn

Dynamic Pool giải quyết storage nhưng UI vẫn không thể enumerate:

```text
/1
```

Cần thiết kế:

- Pagination.
- Range query.
- Search exact IP.
- On-demand candidate generation.

Không render toàn bộ subnet.

---

## 74.4. Disabled Monitoring semantics

Nếu:

```text
enabled = false
```

cần quyết định Dashboard hiển thị:

```text
UNKNOWN
```

hay:

```text
DISABLED
```

hoặc không tính vào Online/Offline.

Chưa khóa.

---

## 74.5. Monitoring Target removal semantics

Hướng hiện tại:

```text
keep Monitoring Check
clear target
reset UNKNOWN
```

nhưng cần khóa trong đặc tả chính thức.

---

## 74.6. Interface transfer

Có cho phép:

```text
Interface
Device A → Device B
```

hay không?

Phương án đơn giản đang được ưu tiên:

> Không cho move Interface trực tiếp. Nếu cần thì workflow riêng hoặc recreate.

---

## 74.7. VLAN detail

Cần chốt:

- VLAN ID validation.
- Exact DB constraints.
- VLAN có bắt buộc trong bản cơ sở cuối hay chỉ Group B.

---

## 74.8. ICMP implementation trên Linux

Cần chọn cách triển khai Go phù hợp với:

- Raw socket.
- Unprivileged ping.
- CAP_NET_RAW.
- Container deployment.

Chưa chọn library/cơ chế cuối.

---

## 74.9. React + Session Cookie

Khi frontend/backend chạy khác origin trong development cần xử lý:

- CORS.
- Credentials.
- SameSite.
- CSRF.

Nếu deployment có thể same-origin thì đơn giản hơn.

---

## 74.10. Exact API / ERD

Concept đã tương đối rõ nhưng:

- Exact endpoint.
- Request/response DTO.
- SQL type.
- Index.
- Migration.
- Transaction boundary.

chưa khóa.

---

# 75. Thứ tự thiết kế trước khi code

Phải ưu tiên:

```text
Requirements
      ↓
Business Rules
      ↓
State Machines
      ↓
Use Cases
      ↓
ERD
      ↓
Database Constraints
      ↓
API Contract
      ↓
Implementation
      ↓
Testing
```

Không vội code trước khi các quan hệ chính được chốt.

---

# 76. Nguyên tắc review trong các phiên chat sau

Khi đề xuất thay đổi kiến trúc, phải trả lời bốn câu:

```text
1. Thiết kế hiện tại có vấn đề gì?
2. Thay đổi giải quyết vấn đề đó như thế nào?
3. Nó có làm đồ án cơ sở quá tải không?
4. Nó ảnh hưởng thế nào tới chuyên ngành/tốt nghiệp?
```

Không thay đổi chỉ vì:

```text
"công nghệ này mới hơn"
"kiến trúc này enterprise hơn"
"thêm cái này nhìn ngầu hơn"
```

Ưu tiên:

```text
Correctness
Scope Discipline
Maintainability
Upgrade Path
Demo Value
Learning Value
```

---

# 77. Phiên bản scope cơ sở hiện được khuyến nghị

```text
IPAM & Network Inventory Management System

CORE
│
├── IPAM
│   ├── Subnet CRUD
│   ├── CIDR
│   ├── Network/Broadcast
│   ├── Usable Range
│   ├── Overlap Detection
│   ├── Resize Validation
│   ├── Dynamic IP Pool
│   ├── Reserve / Unreserve
│   └── Assign / Unassign
│
├── Inventory
│   ├── Device
│   ├── Interface
│   └── VLAN relationship
│
├── Monitoring
│   ├── ICMP
│   ├── Monitoring Check
│   ├── Target IP Allocation
│   ├── UNKNOWN / ONLINE / OFFLINE
│   ├── Retry Threshold
│   ├── Worker Pool
│   ├── Job Deduplication
│   ├── Stale Result Protection
│   └── Last Check / Last Seen
│
├── Security
│   ├── Single Admin
│   ├── Session
│   ├── HttpOnly Cookie
│   ├── CSRF
│   └── Password Hashing
│
└── System
    ├── Go REST API
    ├── PostgreSQL
    ├── React
    ├── Dashboard
    ├── Search / Filter
    └── Testing
```

Bonus ưu tiên:

```text
Basic ICMP Discovery
```

---

# 78. Tóm tắt một đoạn để AI mới hiểu ngay

Đây là một project được thiết kế phát triển xuyên suốt nhiều học kỳ. Giai đoạn cơ sở ngành xây **IPAM & Network Inventory Management System** bằng Go + React + PostgreSQL theo Modular Monolith. Core gồm quản lý IPv4 subnet/CIDR, dynamic IP pool, reserve/assign lifecycle, Device→Interface→IP Allocation→Subnet, VLAN relationship và ICMP monitoring cơ bản bằng Monitoring Check target một assigned IP cụ thể. Hệ thống phải xử lý đúng overlap, network/broadcast, resize subnet, transaction, database constraints, monitoring concurrency, retry, job dedupe và stale result. Không làm SNMP, DNS/DHCP, topology hay automation ở cơ sở. Sang chuyên ngành, core này trở thành Source of Truth cho NMS với Discovery + SNMP + Metrics + History + Alert, có thể thêm Topology. Sang tốt nghiệp, NMS được nâng thành Network Management & Automation Platform với configuration management, Ansible/SSH automation, validation, rollback và NetDevOps workflow. Triết lý xuyên suốt là **Know → Observe → Control**, ưu tiên correctness và scope discipline, không over-engineering.

---

# 79. Prompt để tiếp tục trong một phiên ChatGPT mới

> Hãy đọc toàn bộ file Markdown đính kèm trước khi trả lời. Đây là toàn bộ trạng thái hiện tại của đồ án IPAM/NMS của tôi và có thể coi lịch sử chat cũ không còn tồn tại.
>
> Hãy xem các phần “đã chốt” trong file là baseline hiện tại. Không tự ý đưa SNMP, DNS/DHCP, Topology, Microservices hoặc Automation trở lại đồ án cơ sở nếu tôi chưa yêu cầu.
>
> Nếu phát hiện lỗi logic, hãy chỉ rõ lỗi và ảnh hưởng. Nếu muốn thay đổi thiết kế, hãy giải thích:
>
> 1. Vấn đề của thiết kế hiện tại.
> 2. Giải pháp mới.
> 3. Lợi ích và trade-off.
> 4. Có làm scope cơ sở ngành quá lớn không.
> 5. Có giúp hay cản trở việc nâng lên chuyên ngành/tốt nghiệp không.
>
> Ưu tiên của project là: **chính xác, có kỷ luật, đủ chiều sâu ngành mạng, hoàn thành được trong một học kỳ, và có đường nâng cấp sạch thành NMS rồi Network Automation Platform.**
