# Ý tưởng 01: Quản lý tài nguyên & kiểm kê mạng
## Resource & Inventory Management — IPAM + Network Inventory + Basic Monitoring

**Định hướng:** IPAM tự xây dựng + Network Inventory + Basic ICMP Monitoring

---

## 1. Tổng quan ý tưởng

Ý tưởng 01 có thể phát triển thành một hệ thống trung tâm giúp người quản trị **biết mạng đang có gì**.

Tinh thần chính của hướng này là:

```text
KNOW THE NETWORK
```

Hệ thống dự kiến trả lời các câu hỏi cơ bản:

- Mạng hiện có những subnet nào?
- Mỗi subnet có network address, broadcast address và usable range ra sao?
- IP nào đang được sử dụng?
- IP nào được reserved?
- IP nào còn available?
- IP đang thuộc interface nào?
- Interface thuộc device nào?
- Device nào đang được quản lý?
- Device có trạng thái cơ bản là ONLINE, OFFLINE hay UNKNOWN?

Ý tưởng này không chỉ là một bảng danh sách `Device + IP`, mà là một mô hình tài nguyên mạng có quan hệ rõ ràng giữa subnet, IP, interface, device và VLAN.

---

## 2. Vấn đề thực tế cần giải quyết

Trong mạng nội bộ có từ vài chục thiết bị trở lên, cách quản lý bằng Excel, file cấu hình hoặc ghi nhớ thủ công dễ gặp vấn đề:

- Không biết chính xác IP nào đã được sử dụng.
- Không biết IP nào còn trống.
- Dễ cấp trùng IP.
- Không biết IP đang nằm trên interface nào.
- Không biết interface thuộc device nào.
- Khó quản lý nhiều subnet hoặc VLAN.
- Khi thiết bị mất kết nối phải ping thủ công.
- Dữ liệu quản lý IP và dữ liệu kiểm kê thiết bị không có quan hệ rõ ràng.
- Khi troubleshoot phải kiểm tra thông tin từ nhiều nguồn khác nhau.

Nếu lựa chọn hướng này, hệ thống có thể đóng vai trò một **network source of truth ở mức cơ bản**, giúp dữ liệu quản trị mạng được tập trung và nhất quán hơn.

---

## 3. Phạm vi định hướng

### Core có thể cân nhắc

```text
IPAM
+
Network Inventory
+
Basic ICMP Monitoring
```

Phần core nên tập trung vào:

- Quản lý subnet IPv4/CIDR.
- Tính network address, broadcast address và usable host range.
- Kiểm tra subnet overlap.
- Quản lý reserved IP và assigned IP.
- Suy ra IP available từ usable range trừ các IP đã bị chiếm dụng.
- Quản lý device và interface.
- Gắn IP allocation vào interface.
- Liên kết subnet với VLAN ở mức cơ bản nếu phù hợp.
- Theo dõi trạng thái device bằng ICMP ở mức đơn giản.

### Có thể mở rộng nếu phù hợp

Nếu core đã ổn, có thể cân nhắc một số phần nhẹ:

- Đọc DHCP lease để đối chiếu với inventory.
- Quản lý DNS record nội bộ ở mức đơn giản.
- Discovery nhỏ bằng ICMP hoặc ARP.
- Hiển thị topology graph minh họa.
- Filter hoặc dashboard tốt hơn.

### Không nên đưa vào core ban đầu

Các phần sau nên để dành cho hướng tương lai hoặc đồ án sau:

- SNMP monitoring nâng cao.
- Traffic history dài hạn.
- Time-series metrics.
- Alerting hoàn chỉnh.
- LLDP/CDP topology đầy đủ.
- DNS/DHCP management đầy đủ.
- IPv6, VRF hoặc multi-IP per interface.
- Configuration management.
- Network automation, validation và rollback.

---

## 4. Mô hình tài nguyên mạng

Mô hình lõi có thể đi theo quan hệ:

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

Trong đó:

- Một device có thể có nhiều interface.
- Một interface có thể được gắn một IP chính trong phạm vi cơ sở.
- IP allocation thuộc một subnet.
- Subnet có thể thuộc một VLAN.
- Monitoring nên nhắm tới một assigned IP cụ thể, thay vì hiểu mơ hồ rằng device chỉ có một IP duy nhất.

Cách mô hình này giúp ý tưởng có chiều sâu hơn một ứng dụng CRUD thông thường, vì nó phản ánh cấu trúc mạng thực tế: device không chỉ là một dòng có `ip_address`, mà có interface, allocation, subnet và VLAN liên quan.

---

## 5. IPAM ở mức ý tưởng

Phần IPAM có thể tập trung vào các năng lực:

- Tạo và quản lý subnet IPv4 theo CIDR.
- Tính network address.
- Tính broadcast address.
- Tính usable range.
- Kiểm tra IP có nằm trong subnet hay không.
- Phát hiện subnet overlap.
- Reserve IP cho mục đích giữ trước.
- Assign IP cho interface.
- Unreserve hoặc unassign IP khi không còn dùng.
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

Cách này giúp ý tưởng phù hợp hơn với subnet lớn, vì không cần tạo hàng nghìn hoặc hàng triệu bản ghi chỉ để biểu diễn các IP còn trống.

---

## 6. IP allocation lifecycle

Ý tưởng có thể mô tả vòng đời IP ở mức khái niệm:

```text
available → reserved
available → assigned
reserved  → assigned
reserved  → available
assigned  → available
```

Ý nghĩa:

- `available`: IP còn trống, chưa có allocation được lưu.
- `reserved`: IP được giữ trước, chưa gắn vào interface.
- `assigned`: IP đã được gắn vào interface của device.

Điểm cần chú ý là hệ thống phải tránh cấp trùng IP. Việc cấp phát IP cần bảo đảm một địa chỉ không bị gán đồng thời cho nhiều interface, kể cả khi có nhiều thao tác diễn ra gần như cùng lúc.

---

## 7. Basic Network Monitoring

Monitoring trong Ý tưởng 01 chỉ nên là phần hỗ trợ để kiểm tra nhanh trạng thái device đã có trong inventory.

Ở mức core, có thể chỉ dùng:

- ICMP monitoring.
- ONLINE.
- OFFLINE.
- UNKNOWN.
- Last Check.
- Last Seen.
- Background monitoring.

Trạng thái có thể hiểu theo hướng:

- `UNKNOWN`: chưa có đủ dữ liệu hoặc chưa kiểm tra lần nào.
- `ONLINE`: target phản hồi ICMP thành công.
- `OFFLINE`: target không phản hồi sau một số lần kiểm tra liên tiếp.

Cần lưu ý rằng `OFFLINE` chỉ có nghĩa target không phản hồi theo phương pháp kiểm tra hiện tại, không nhất thiết khẳng định thiết bị đã tắt nguồn.

---

## 8. Các bài toán kỹ thuật đáng nghiên cứu

Nếu lựa chọn hướng này, phần thú vị nằm ở các bài toán nền tảng:

- **CIDR calculation:** tính đúng network, broadcast và usable range.
- **Subnet overlap detection:** tránh tạo các subnet chồng lấn trong cùng hệ thống.
- **Subnet resize validation:** nếu đổi prefix, các IP đã reserved hoặc assigned phải vẫn hợp lệ.
- **Dynamic IP allocation:** không pre-generate toàn bộ IP pool, chỉ suy ra IP available khi cần.
- **IP allocation lifecycle:** quản lý rõ available, reserved và assigned.
- **Duplicate allocation prevention:** tránh cấp trùng IP khi nhiều thao tác xảy ra gần nhau.
- **Database integrity:** dữ liệu device, interface, subnet, VLAN và monitoring target không bị mồ côi hoặc mâu thuẫn.
- **Concurrent allocation:** cần có lớp bảo vệ cuối cùng để hai request không cùng cấp thành công một IP.
- **Background monitoring:** kiểm tra trạng thái device mà không làm nghẽn thao tác người dùng.
- **Monitoring consistency:** khi IP bị unassign hoặc target thay đổi, trạng thái monitoring phải được xử lý hợp lý.

Các điểm này đủ để Ý tưởng 01 có chiều sâu kỹ thuật, nhưng chưa cần mô tả SQL schema, API endpoint hoặc thuật toán worker chi tiết.

---

## 9. Kiến trúc dự kiến

Kiến trúc có thể giữ ở mức đơn giản:

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

Có thể triển khai theo hướng modular monolith để dễ phát triển, kiểm thử và demo.

Công nghệ nếu lựa chọn hướng này có thể dự kiến như sau, nhưng chưa phải quyết định bắt buộc:

| Thành phần | Dự kiến |
| ---------- | ------- |
| Backend | Go |
| Frontend | React |
| Database | PostgreSQL |
| API | REST |
| Monitoring | ICMP |
| Môi trường | Linux / VM / Homelab |

---

## 10. Demo có thể thực hiện

Một demo cơ sở có thể đi theo luồng:

1. Đăng nhập Administrator.
2. Tạo subnet `192.168.10.0/24`.
3. Hệ thống hiển thị network, broadcast và usable range.
4. Reserve một IP.
5. Tạo device.
6. Tạo interface cho device.
7. Assign một IP vào interface.
8. Thử assign trùng IP để chứng minh validation.
9. Hiển thị số IP còn available.
10. Bật ICMP monitoring cho assigned IP.
11. Tắt một VM hoặc target trong lab để thấy trạng thái OFFLINE.
12. Bật lại target để thấy trạng thái ONLINE và cập nhật Last Seen.
13. Search hoặc filter subnet/device/IP.

Nếu còn thời gian, demo có thể thêm discovery nhỏ hoặc đối chiếu DHCP lease, nhưng không nên để phần mở rộng lấn át core.

---

## 11. Ứng dụng thực tế

### Doanh nghiệp vừa và nhỏ

Có thể dùng để kiểm kê:

- Subnet.
- VLAN.
- Router.
- Switch.
- Access Point.
- Server.
- PC.
- Printer.
- Camera.
- IP tĩnh.

### Trường học / phòng lab

Có thể dùng để quản lý:

- Phòng máy.
- Thiết bị mạng.
- Server nội bộ.
- IP cấp cho từng khu vực.
- VLAN theo phòng hoặc vai trò.

### Homelab

Có thể dùng để quản lý:

- NAS.
- Raspberry Pi.
- ESP32.
- Virtual Machine.
- Home Server.
- Router.
- Access Point.
- Dịch vụ nội bộ.

---

## 12. Rủi ro và cách giới hạn scope

Ý tưởng 01 có khả năng mở rộng rất lớn nên rủi ro chính là scope creep.

Các phần dễ làm scope phình to:

- DNS/DHCP integration.
- Topology discovery.
- SNMP.
- Metrics history.
- Alerting.
- Network automation.

Cách giảm rủi ro:

- Làm chắc IPAM trước.
- Làm rõ quan hệ Device → Interface → IP Allocation.
- Monitoring chỉ giữ ở mức ICMP cơ bản.
- DNS/DHCP/Topology chỉ xem là extension hoặc proof of concept.
- Không claim triển khai production enterprise; chỉ kiểm thử trong lab, VM hoặc homelab.

---

## 13. Khả năng phát triển dài hạn

Nếu Ý tưởng 01 được lựa chọn và tiếp tục mở rộng, roadmap dài hạn có thể đi theo hướng:

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

Đây chỉ là khả năng phát triển dài hạn, không phải kế hoạch bắt buộc của đồ án cơ sở.

Các hướng tương lai có thể gồm:

- SNMP.
- Discovery.
- Metrics.
- History.
- Alerting.
- DNS/DHCP integration.
- Topology.
- IPv6.
- Configuration Management.
- Network Automation.
- Validation.
- Rollback.

---

## 14. Giá trị đối với đồ án và portfolio

Ý tưởng này phù hợp với các hướng:

- Network Administrator.
- System Administrator.
- Infrastructure Engineer.
- Network Engineer.
- NOC Engineer.
- NetDevOps ở giai đoạn mở rộng.

Một phiên bản hoàn chỉnh có thể chứng minh:

- Hiểu IPv4/CIDR/Subnetting.
- Hiểu mô hình Device/Interface/IP/VLAN.
- Biết thiết kế dữ liệu có quan hệ.
- Biết xây REST API và dashboard.
- Biết xử lý validation và consistency.
- Biết thiết kế background monitoring.
- Biết dựng lab nhỏ để demo.

---

## 15. Đánh giá sơ bộ

| Tiêu chí | Đánh giá |
|---|---|
| Phù hợp đồ án cơ sở | Cao nếu giới hạn core |
| Kiến thức Network | Cao |
| Backend/Database | Cao |
| Khả năng demo | Cao |
| Độ khó | Trung bình – khá cao |
| Nguy cơ scope creep | Cao |
| Khả năng nâng chuyên ngành | Rất cao |
| Giá trị CV Network/Infra | Rất cao |
| Khả năng phát triển dài hạn | Rất cao |

---

## 16. Kết luận ý tưởng

Ý tưởng 01 là hướng cân bằng giữa Networking, Backend, Database và Monitoring.

Nếu lựa chọn hướng này, trọng tâm nên là:

```text
IPAM
+
Network Inventory
+
Basic ICMP Monitoring
```

Điều quan trọng nhất là không cố triển khai toàn bộ DNS/DHCP/Topology ngay từ đầu. IPAM và Inventory nên là nền móng chính; các phần còn lại chỉ nên được thêm dần nếu phạm vi và thời gian cho phép.
