# Nhóm 6: Mô phỏng hạ tầng doanh nghiệp & NetDevOps
## Enterprise Infrastructure Lab → Network Automation / NetDevOps

**Nguồn ý tưởng:** Mô phỏng hạ tầng doanh nghiệp, sau đó nâng cấp bằng Network Automation

---

## 1. Tổng quan ý tưởng

Xây dựng một **mô hình mạng doanh nghiệp thu nhỏ** trên môi trường Lab, tập trung vào thiết kế, cấu hình và vận hành hạ tầng mạng thay vì xây một ứng dụng quản lý ngay từ đầu.

Đồ án cơ sở ngành tập trung trả lời:

- Doanh nghiệp nên chia mạng thành những VLAN nào?
- Các VLAN giao tiếp với nhau như thế nào?
- Routing được thiết kế ra sao?
- DHCP cấp IP ở đâu?
- ACL kiểm soát lưu lượng giữa các vùng như thế nào?
- Server nội bộ được đặt ở đâu?
- Khi một liên kết hoặc thiết bị gặp lỗi thì mạng xử lý ra sao?
- Cấu hình hệ thống được tổ chức và tài liệu hóa như thế nào?

Sau khi phần hạ tầng ổn định, đồ án chuyên ngành có thể phát triển theo hướng:

```text
Enterprise Network
       ↓
Network Automation
       ↓
NetDevOps
```

Tức là từ việc **cấu hình mạng thủ công** chuyển sang **quản lý cấu hình bằng Code, Git và Automation**.

---

## 2. Vấn đề thực tế cần giải quyết

Trong hệ thống mạng doanh nghiệp, quản trị viên thường phải xử lý:

- Phân chia mạng theo phòng ban.
- Tách Broadcast Domain.
- Quản lý VLAN.
- Routing giữa các VLAN.
- Cấp phát IP.
- Kiểm soát truy cập.
- Kết nối server nội bộ.
- Dự phòng khi liên kết hoặc thiết bị lỗi.
- Ghi chép cấu hình.
- Thay đổi cấu hình trên nhiều thiết bị.

Nếu số lượng thiết bị tăng, cấu hình thủ công từng router/switch sẽ:

- Tốn thời gian.
- Dễ sai.
- Khó đồng bộ.
- Khó biết cấu hình nào đang chạy ở đâu.
- Khó rollback khi thay đổi lỗi.

Đồ án được thiết kế theo hai tầng:

```text
Tầng 1: Xây dựng hạ tầng đúng
Tầng 2: Tự động hóa cách quản lý hạ tầng
```

---

## 3. Ứng dụng thực tế

### 3.1. Doanh nghiệp vừa và nhỏ

Mô phỏng:

- Office Network.
- Server Network.
- Management Network.
- Guest Network.
- Wi-Fi.
- Internal Services.

### 3.2. Trường học

Có thể chia:

- VLAN giảng viên.
- VLAN sinh viên.
- VLAN phòng Lab.
- VLAN Server.
- VLAN quản trị.

### 3.3. Homelab

Có thể mô phỏng:

- Router.
- Layer 2 / Layer 3 Switch.
- Firewall.
- Server.
- Client.
- Management Host.

### 3.4. Network Engineering Lab

Dùng để thực hành:

- VLAN.
- Trunk.
- Routing.
- ACL.
- OSPF.
- DHCP.
- NAT.
- Redundancy.
- Troubleshooting.

---

## 4. Giá trị chính của đề tài

Khác với các nhóm xây ứng dụng chạy trên một mạng có sẵn, nhóm này tập trung vào **chính hạ tầng mạng**.

Đồ án chứng minh hai lớp năng lực:

```text
Networking Fundamentals
          +
Infrastructure Design
```

Sau đó nâng lên:

```text
Automation
+
Version Control
+
Configuration Management
```

Đây là hướng phù hợp nếu muốn tập trung nhiều vào Network Engineer và sau này chuyển sang NetDevOps.

---

## 5. Mô hình Lab sơ bộ

Ví dụ:

```text
                      Internet
                         |
                       Router
                         |
                  Core / L3 Switch
          ┌──────────────┼──────────────┐
          │              │              │
       VLAN 10        VLAN 20        VLAN 30
     Management        Staff          Server
          │              │              │
       Admin-PC        Clients        Services
```

Có thể mở rộng:

```text
                         WAN
                          |
                    Edge Router
                          |
                     Firewall
                          |
                 Distribution/Core
                   /            \
              Access SW1      Access SW2
                |                |
             VLANs            VLANs
```

Không cần mô phỏng một doanh nghiệp quá lớn. Mục tiêu là tạo một Lab đủ nhỏ để kiểm thử nhưng đủ nhiều thành phần để thể hiện thiết kế mạng.

---

## 6. Các thành phần kỹ thuật có thể sử dụng

### Network

- VLAN.
- 802.1Q Trunk.
- Inter-VLAN Routing.
- Static Routing.
- OSPF.
- DHCP.
- DNS cơ bản.
- NAT.
- ACL.
- STP/RSTP.
- Link redundancy.
- Default Gateway.

### Server / System

- Linux Server.
- DHCP/DNS service.
- Web service nội bộ.
- Monitoring hoặc Management Host.

### Lab Platform

Có thể sử dụng:

- GNS3.
- EVE-NG.
- Virtual Machine.
- Container.
- Thiết bị thật trong Homelab nếu có.

---

## 7. Kiến trúc logic dự kiến

Có thể chia hạ tầng thành các Zone:

```text
Internet / WAN
      |
      v
Edge
      |
      v
Core / Distribution
      |
      +-- Management VLAN
      +-- Staff VLAN
      +-- Server VLAN
      +-- Guest VLAN
```

Ví dụ policy:

```text
Management
   → được quản trị thiết bị

Staff
   → truy cập Server cần thiết

Guest
   → chỉ ra Internet

Server
   → hạn chế truy cập theo ACL
```

Mục đích là chứng minh khả năng phân đoạn mạng và kiểm soát luồng dữ liệu.

---

## 8. Phạm vi đề xuất cho đồ án cơ sở ngành

### Nhóm A — Core bắt buộc

- Thiết kế sơ đồ mạng doanh nghiệp nhỏ.
- Lập IP Plan.
- Chia VLAN.
- Cấu hình Trunk.
- Inter-VLAN Routing.
- DHCP.
- Routing giữa các phần mạng.
- ACL cơ bản.
- Một số dịch vụ Server nội bộ.
- Kiểm thử Connectivity.
- Viết tài liệu cấu hình và sơ đồ mạng.

Có thể chọn:

```text
Static Routing
hoặc
OSPF
```

tùy quy mô Lab.

### Nhóm B — Tăng chiều sâu nếu còn thời gian

- OSPF nếu Core đang dùng Static Routing.
- Redundancy.
- STP/RSTP.
- NAT.
- Firewall rule.
- Config Backup.
- Một Playbook Automation nhỏ.
- Dashboard trạng thái Lab.

### Nhóm C — Chưa làm ở đồ án cơ sở

- Full SD-WAN.
- BGP đa site phức tạp.
- EVPN/VXLAN.
- Full Network Automation Platform.
- CI/CD hoàn chỉnh cho Network Config.
- Automated Rollback.
- Dynamic Inventory lớn.
- Multi-vendor automation sâu.

---

## 9. Kịch bản Demo cơ sở ngành

Ví dụ doanh nghiệp có:

```text
VLAN 10 - Management
VLAN 20 - Staff
VLAN 30 - Server
VLAN 40 - Guest
```

Demo:

1. Trình bày sơ đồ mạng.
2. Trình bày IP Plan.
3. Client VLAN Staff nhận IP từ DHCP.
4. Staff truy cập Server nội bộ.
5. Guest không được truy cập Management VLAN.
6. Admin trong Management VLAN truy cập thiết bị quản trị.
7. Kiểm tra Routing.
8. Kiểm tra ACL.
9. Tắt một đường kết nối nếu có Redundancy.
10. Chứng minh hệ thống vẫn hoạt động hoặc mô tả hành vi dự kiến.
11. Nếu có Automation PoC: chạy một Playbook thay đổi cấu hình nhỏ trên nhiều thiết bị.

Demo của nhóm này nên tập trung vào **luồng Packet và chính sách mạng**, không chỉ vào giao diện.

---

## 10. Điểm cần quyết định khi đi sâu

- Lab dùng GNS3 hay EVE-NG.
- Vendor/device image nào được sử dụng.
- Dùng Static Routing hay OSPF.
- Có cần Firewall riêng hay dùng ACL trên Router/L3 Switch.
- Có làm Redundancy ở đồ án cơ sở không.
- DHCP chạy trên Router hay Linux Server.
- Có sử dụng DNS nội bộ không.
- Mức độ Automation PoC ở kỳ cơ sở.
- Network diagram và IP Plan được quản lý theo cách nào.
- Cần bao nhiêu Site/Branch để không làm Scope quá lớn.

---

## 11. Rủi ro và độ khó

### Độ khó tổng thể

**Trung bình – khá cao**, phụ thuộc số protocol và số thiết bị mô phỏng.

### Rủi ro

- Dễ nhét quá nhiều công nghệ.
- Tốn thời gian troubleshoot Lab.
- Image thiết bị có thể nặng.
- Lab lớn gây tốn RAM/CPU.
- Nếu chỉ cấu hình thủ công thì project dễ giống một bài thực hành CCNA mở rộng.
- Có thể thiếu Software Artifact rõ ràng để đưa lên GitHub.

### Cách giảm rủi ro

Giới hạn Lab:

```text
1 Core
2 Access
1 Router/Firewall
1-2 Server
Một số Client
```

và chỉ chọn các protocol thật sự phục vụ kiến trúc.

---

## 12. Điểm yếu nếu chỉ dừng ở đồ án cơ sở

Nếu chỉ có:

```text
GNS3/EVE-NG
+
VLAN
+
OSPF
+
ACL
```

thì giá trị Networking tốt nhưng project có thể trông giống một **Lab lớn** hơn là một sản phẩm phần mềm.

Do đó khi muốn xây Portfolio lâu dài, đồ án chuyên ngành nên bổ sung:

```text
Automation
Source of Truth
Git Workflow
Validation
Backup / Rollback
```

Đây là bước biến Lab thành một project NetDevOps thực sự.

---

## 13. Nâng cấp thành đồ án chuyên ngành

Hướng đề xuất:

# Network Automation / NetDevOps Platform

Luồng:

```text
Network Source of Truth
        |
        v
Config Templates
        |
        v
Automation Engine
        |
        v
Network Devices
        |
        v
Validation
        |
        v
Backup / Rollback
```

Các module có thể bổ sung:

### 13.1. Inventory

- Device list.
- IP Management.
- Interface.
- VLAN.
- Credentials reference.
- Site/Role.

### 13.2. Configuration Template

- Template cấu hình VLAN.
- Interface.
- Routing.
- ACL.

### 13.3. Automation

- Ansible.
- SSH Automation.
- Deploy config nhiều thiết bị.
- Scheduled task.

### 13.4. Validation

- Kiểm tra cấu hình sau Deploy.
- Ping test.
- Routing test.
- Interface status.

### 13.5. Configuration Backup

- Backup Running Config.
- Versioning.
- Compare Diff.
- Restore/Rollback.

### 13.6. Git-based Workflow

Có thể phát triển:

```text
Git
 ↓
Review Config
 ↓
Automation
 ↓
Deploy
 ↓
Validate
```

---

## 14. Hướng phát triển xa hơn

Sau NetDevOps có thể mở rộng:

- CI/CD cho Network Configuration.
- Change Approval.
- Automated Compliance.
- Policy as Code.
- Multi-vendor Automation.
- SD-WAN Lab.
- Network Digital Twin.
- Integration với NMS/IPAM.

Nhóm này có khả năng hội tụ với Nhóm 1:

```text
Resource & Inventory
        +
NetDevOps
        ↓
Network Automation Platform
```

---

## 15. Giá trị đối với CV / Portfolio

Phù hợp với:

- Network Engineer.
- Network Administrator.
- Infrastructure Engineer.
- NetDevOps Engineer.
- Network Automation Engineer.

Có thể chứng minh:

- Hiểu VLAN/Trunk.
- Hiểu Routing.
- Hiểu DHCP.
- Hiểu ACL.
- Hiểu Redundancy.
- Biết thiết kế Network.
- Biết Troubleshooting.
- Biết Linux/Git.
- Biết Ansible/Automation khi nâng cấp.

Đây là hướng có giá trị CV rất cao nếu project chuyên ngành thực sự chuyển được từ cấu hình thủ công sang Automation.

---

## 16. Đánh giá sơ bộ

| Tiêu chí | Đánh giá |
|---|---|
| Phù hợp đồ án cơ sở | Cao |
| Kiến thức Network Core | Rất cao |
| Backend/Software ở kỳ cơ sở | Thấp – trung bình |
| Khả năng Demo | Rất cao |
| Độ khó | Trung bình – khá cao |
| Nguy cơ Scope Creep | Trung bình |
| Khả năng nâng chuyên ngành | Rất cao |
| Giá trị CV Network Engineer | Rất cao |
| Giá trị CV NetDevOps sau nâng cấp | Rất cao |

---

## 17. Kết luận ý tưởng

Cấu trúc phát triển:

```text
Đồ án cơ sở
Enterprise Network Infrastructure Lab

        ↓

Đồ án chuyên ngành
Network Automation / NetDevOps

        ↓

Phát triển xa hơn
Network Platform / Configuration as Code
```

Đây là hướng phù hợp nếu muốn ưu tiên **Networking Core trước**, sau đó bổ sung Automation để tạo ra một Portfolio mạnh hơn.

Điều quan trọng là không biến đồ án cơ sở thành một Lab quá lớn. Phần cơ sở phải chứng minh kiến trúc mạng đúng, còn phần chuyên ngành mới tập trung biến việc vận hành mạng thành một quy trình tự động và có kiểm soát.
