# Ý tưởng 01: Quản lý tài nguyên & kiểm kê mạng

> Hướng xây dựng một hệ thống giúp người quản trị biết mạng đang có gì: subnet, IP, device, interface, VLAN và trạng thái cơ bản của thiết bị.

## 1. Tổng quan & bài toán

Ý tưởng 01 xoay quanh tinh thần:

```text
KNOW THE NETWORK
```

Nếu lựa chọn hướng này, hệ thống có thể đóng vai trò một **network source of truth ở mức cơ bản** cho mạng nội bộ. Thay vì quản lý bằng Excel, file cấu hình hoặc ghi nhớ thủ công, người quản trị có thể tra cứu tập trung:

- Mạng có những subnet nào.
- Subnet có network address, broadcast address và usable range ra sao.
- IP nào đang available, reserved hoặc assigned.
- IP đang thuộc interface nào, interface thuộc device nào.
- Device nào đang được quản lý và trạng thái cơ bản là ONLINE, OFFLINE hay UNKNOWN.

Bài toán thực tế nằm ở việc dữ liệu IP và dữ liệu thiết bị thường rời rạc. Khi số lượng thiết bị tăng, rất dễ cấp trùng IP, khó biết thiết bị nằm ở subnet/VLAN nào và mất thời gian khi troubleshooting.

## 2. Mục tiêu

Core của ý tưởng có thể tập trung vào:

```text
IPAM
+
Network Inventory
+
Basic Monitoring
```

Mục tiêu là làm rõ quan hệ giữa tài nguyên mạng, không chỉ tạo một danh sách `Device + IP`. Một mô hình tối giản nhưng có chiều sâu có thể là:

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

Từ mô hình này, hệ thống có thể hỗ trợ quản lý subnet IPv4/CIDR, cấp phát IP, kiểm kê device/interface và kiểm tra trạng thái thiết bị bằng ICMP ở mức cơ bản.

## 3. Hướng triển khai

Phần IPAM có thể xử lý các năng lực chính:

- Tạo và quản lý subnet IPv4 theo CIDR.
- Tính network address, broadcast address và usable host range.
- Kiểm tra subnet overlap.
- Reserve IP, assign IP cho interface, unreserve hoặc unassign khi không còn dùng.
- Hiển thị IP available mà không cần tạo sẵn toàn bộ danh sách IP trong database.

Một hướng hợp lý là chỉ lưu các IP đã thật sự bị chiếm dụng:

```text
reserved
assigned
```

Còn `available` có thể được suy ra:

```text
Available IP
=
Usable Host Range
-
Reserved IP
-
Assigned IP
```

Vòng đời IP có thể giữ ở mức khái niệm:

```text
available → reserved
available → assigned
reserved  → assigned
reserved  → available
assigned  → available
```

Phần inventory có thể quản lý device, interface, VLAN và quan hệ gắn IP vào interface. Monitoring chỉ nên hỗ trợ kiểm tra nhanh tình trạng device đã có trong inventory, chưa phải NMS hoàn chỉnh.

## 4. Kỹ thuật & công nghệ

Các bài toán kỹ thuật đáng giữ trong ý tưởng:

- **CIDR calculation:** tính đúng network, broadcast và usable range.
- **Subnet overlap detection:** tránh tạo các subnet chồng lấn.
- **Subnet resize validation:** khi đổi prefix, IP đã reserved hoặc assigned vẫn phải hợp lệ.
- **Duplicate IP prevention:** tránh cấp trùng IP, kể cả khi nhiều thao tác xảy ra gần nhau.
- **Database integrity:** dữ liệu device, interface, subnet, VLAN và monitoring target không bị mâu thuẫn.
- **Basic ICMP monitoring:** trạng thái `UNKNOWN`, `ONLINE`, `OFFLINE`, `Last Check`, `Last Seen`.

Kiến trúc dự kiến có thể giữ đơn giản:

```text
Web UI
   ↓
Backend API
   ↓
Business Logic
   ↓
Database

      +

Background Monitoring
   ↓
Network Devices
```

Công nghệ nếu lựa chọn hướng này có thể dự kiến là Go, React, PostgreSQL, REST API, ICMP và môi trường Linux/VM/Homelab. Đây chỉ là lựa chọn dự kiến, chưa phải quyết định bắt buộc.

## 5. Phạm vi

### Core

- IPAM cho IPv4/CIDR.
- Subnet, usable range, overlap validation.
- Reserved/assigned/available IP.
- Device, Interface, IP Allocation, VLAN cơ bản.
- Duplicate allocation prevention.
- Basic ICMP monitoring với `ONLINE / OFFLINE / UNKNOWN`.
- Dashboard và search/filter ở mức đủ demo.

### Mở rộng

- Đọc DHCP lease để đối chiếu với inventory.
- DNS record nội bộ ở mức đơn giản.
- Discovery nhỏ bằng ICMP hoặc ARP.
- Topology graph minh họa.
- SNMP, metrics, history, alerting, IPv6, configuration management và network automation nên để dành cho giai đoạn sau.

## 6. Demo dự kiến

Một demo gọn có thể đi theo luồng:

1. Tạo subnet `192.168.10.0/24` và hiển thị network/broadcast/usable range.
2. Reserve một IP, tạo device và interface.
3. Assign IP vào interface.
4. Thử assign trùng IP để chứng minh validation.
5. Hiển thị số IP còn available.
6. Bật ICMP monitoring cho assigned IP.
7. Tắt/bật target trong lab để thấy `OFFLINE`, `ONLINE` và `Last Seen` thay đổi.

Demo nên chứng minh được cả IPAM, quan hệ inventory và monitoring cơ bản.

## 7. Hướng phát triển

Nếu Ý tưởng 01 được lựa chọn và tiếp tục mở rộng, roadmap dài hạn có thể là:

```text
KNOW
  ↓
OBSERVE
  ↓
CONTROL
```

Tương ứng:

```text
IPAM & Inventory
        ↓
Network Management System
        ↓
Network Management & Automation Platform
```

Đây chỉ là khả năng phát triển dài hạn. Core ban đầu nên làm chắc IPAM + Inventory trước, không đưa toàn bộ DNS/DHCP/Topology hoặc automation vào từ đầu.

## 8. Đánh giá

| Tiêu chí | Đánh giá |
| --- | --- |
| Độ khó | Trung bình – khá cao |
| Networking | Cao |
| System / Software | Cao |
| Khả năng demo | Cao |
| Khả năng mở rộng | Rất cao |
| Giá trị portfolio | Rất cao |
| Rủi ro chính | Scope creep nếu ôm DNS/DHCP/Topology/NMS quá sớm |
