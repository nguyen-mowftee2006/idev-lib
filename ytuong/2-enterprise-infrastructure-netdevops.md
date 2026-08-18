# Ý tưởng 02: Mô phỏng hạ tầng doanh nghiệp & NetDevOps

> Hướng xây dựng một mô hình mạng doanh nghiệp thu nhỏ trong lab, sau đó có thể nâng cấp dần sang Network Automation và NetDevOps.

## 1. Tổng quan & bài toán

Ý tưởng 02 tập trung vào **chính hạ tầng mạng** thay vì xây một ứng dụng quản lý ngay từ đầu. Nếu lựa chọn hướng này, đồ án có thể mô phỏng một mạng doanh nghiệp nhỏ với VLAN, routing, DHCP, ACL, dịch vụ nội bộ và một số tình huống vận hành.

Bài toán thực tế là khi hệ thống mạng tăng số lượng thiết bị và vùng mạng, việc cấu hình thủ công dễ sai, khó đồng bộ, khó rollback và khó biết cấu hình nào đang chạy ở đâu.

Hướng phát triển tự nhiên của ý tưởng là:

```text
Enterprise Network
        ↓
Network Automation
        ↓
NetDevOps
```

## 2. Mục tiêu

Mục tiêu ban đầu là thiết kế một lab đủ nhỏ để kiểm thử nhưng đủ nhiều thành phần để thể hiện tư duy Network Engineer:

- Chia VLAN theo vai trò hoặc phòng ban.
- Thiết kế IP Plan.
- Cấu hình trunk và inter-VLAN routing.
- Cấp IP bằng DHCP.
- Kiểm soát lưu lượng bằng ACL.
- Có server/network services nội bộ.
- Kiểm thử connectivity, routing, policy và troubleshooting.

Sau khi hạ tầng ổn định, ý tưởng có thể mở rộng sang automation bằng Ansible, Git, configuration management, validation, backup và rollback.

## 3. Hướng triển khai

Một mô hình lab cơ bản có thể gồm:

```text
Internet / WAN
      ↓
Edge Router / Firewall
      ↓
Core / Distribution
      ↓
Access Switches
      ↓
Management / Staff / Server / Guest VLAN
```

Policy có thể mô tả đơn giản:

- Management được quản trị thiết bị.
- Staff truy cập các dịch vụ cần thiết.
- Guest chỉ ra Internet.
- Server được bảo vệ bằng ACL hoặc firewall rule.

Phần automation nếu có chỉ nên ở mức PoC trong đồ án cơ sở, ví dụ backup cấu hình, đẩy một thay đổi nhỏ hoặc kiểm tra trạng thái sau deploy.

## 4. Kỹ thuật & công nghệ

Các năng lực kỹ thuật nên giữ trong ý tưởng:

- VLAN, 802.1Q trunk, inter-VLAN routing.
- Static routing hoặc OSPF tùy quy mô lab.
- DHCP, DNS cơ bản, NAT nếu phù hợp.
- ACL, firewall rule, STP/RSTP, redundancy.
- Linux server, web service nội bộ, monitoring hoặc management host.
- GNS3, EVE-NG, VM, container hoặc thiết bị thật trong homelab.
- Network Automation với Ansible, Git workflow, configuration template, validation, config backup và rollback ở giai đoạn mở rộng.

Điểm kỹ thuật đặc trưng của ý tưởng là chứng minh được luồng packet, phân đoạn mạng, policy và khả năng vận hành hạ tầng, không chỉ vẽ sơ đồ.

## 5. Phạm vi

### Core

- Sơ đồ mạng doanh nghiệp nhỏ.
- IP Plan.
- VLAN, trunk, inter-VLAN routing.
- DHCP.
- Static routing hoặc OSPF.
- ACL cơ bản.
- Một số server/network services nội bộ.
- Kiểm thử connectivity và troubleshooting.
- Tài liệu cấu hình và sơ đồ mạng.

### Mở rộng

- Redundancy, STP/RSTP, NAT, firewall rule.
- Config backup.
- Một playbook automation nhỏ.
- Dashboard trạng thái lab.
- NetDevOps sâu hơn: Git workflow, validation, automated backup/rollback, CI/CD cho network configuration.

Không nên đưa vào core các phần quá lớn như full SD-WAN, BGP đa site phức tạp, EVPN/VXLAN hoặc một network automation platform hoàn chỉnh.

## 6. Demo dự kiến

Một demo gọn có thể dùng các VLAN:

```text
VLAN 10 - Management
VLAN 20 - Staff
VLAN 30 - Server
VLAN 40 - Guest
```

Luồng demo:

1. Trình bày topology và IP Plan.
2. Client ở Staff nhận IP từ DHCP.
3. Staff truy cập server nội bộ qua routing đúng thiết kế.
4. Guest bị chặn truy cập Management VLAN.
5. Admin trong Management VLAN quản trị thiết bị.
6. Kiểm tra ACL/routing bằng ping, traceroute hoặc lưu lượng mẫu.
7. Nếu có redundancy, tắt một đường kết nối và quan sát hành vi dự kiến.
8. Nếu có automation PoC, chạy một playbook thay đổi hoặc backup cấu hình.

Demo nên tập trung vào luồng packet và chính sách mạng.

## 7. Hướng phát triển

Nếu tiếp tục phát triển sau lab cơ sở, hướng nâng cấp có thể là:

```text
Enterprise Infrastructure
        ↓
Network Automation
        ↓
NetDevOps
```

Ở giai đoạn sau, lab có thể được gắn với source of truth, config templates, automation engine, validation, config backup và rollback:

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

Ý tưởng này cũng có thể hội tụ với Ý tưởng 01 nếu cần một nền tảng vừa có inventory/IPAM vừa có automation.

## 8. Đánh giá

| Tiêu chí | Đánh giá |
| --- | --- |
| Độ khó | Trung bình – khá cao |
| Networking | Rất cao |
| System / Software | Thấp – trung bình ở core, cao hơn khi mở rộng automation |
| Khả năng demo | Rất cao |
| Khả năng mở rộng | Rất cao |
| Giá trị portfolio | Rất cao cho Network Engineer / NetDevOps |
| Rủi ro chính | Lab quá lớn, tốn thời gian troubleshoot, thiếu software artifact nếu không nâng cấp automation |
