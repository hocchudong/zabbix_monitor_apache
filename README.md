# I. Đặt vấn đề
Trong bài viết này mình sẽ trình bày 2 phương pháp để monitor apache

Phương pháp 1: Sử dụng các thông số trả về từ
```
http://localhost/server-status?auto
``` 

<img src=http://i.imgur.com/78VAxvj.png width="80%" height="80%" border="1">

Zabbix Agent sẽ thu thập các thông số này rồi trả về cho Zabbix Server. Ý nghĩa của các tham số:

- Total Accesses: 4206 ( Tổng số truy cập)

- Total kBytes: 2777 ( Tổng lưu lượng gửi đến server)

- CPULoad: .0279888 ( CPU web service đang sử dụng)

- Uptime: 252351   (Thời gian web service đang active)

- ReqPerSec: .0166673( Số request trên giây)

- BytesPerSec: 11.2686 ( Số byte trên giây)

- BytesPerReq: 676.093 (Số byte trên request)

- BusyWorkers: 1 (Số phiên đang bận)

- IdleWorkers: 49 (Số phiên rảnh rỗi)

Đánh giá: Được apache hỗ trợ nên các số liệu trả về chính xác. Tuy nhiên Zabbix gent để lấy được các số liệu này cần truy cập vào web service vô tình tăng tổng số truy cập nên số liệu monitor trên zabbix server đã bao gồm sự truy cập của Zabbix Agent

Phương pháp 2: Lọc ra trong file log của apache

- Tổng số truy cập

- Tổng số truy cập theo thời gian

- Tổng số truy cập theo phản hổi HTTP status( 200, 404,vv...)

- Tính tổng dung lượng (theo Bye)truy cập 

Đánh giá: phương pháp này thống kê theo file log của apache nên số liệu chính xác tuy nhiên có hạn chế do file log không hỗ trợ nhiều và file log bị hệ thống xóa định kỳ

#II. Triển khai

##Phương pháp 1:


### Trên máy Zabbix Agent
Tải file script:

```sh
wget https://raw.githubusercontent.com/thanhha123/zabbix_monitor_apache/master/zapache
```

Tạo thư mục chứa script và thêm quyền file thực thi:

```sh
mkdir -p /var/lib/zabbixsrv/externalscripts/
cp zapache /var/lib/zabbixsrv/externalscripts/
chmod +x /var/lib/zabbixsrv/externalscripts/zapache
```
Chỉnh sửa file cấu hình apache:

```sh
vi /etc/apache2/mods-enables/status.conf
```

<img src=http://i.imgur.com/F7nH9jG.png width="80%" height="80%" border="1">

Test file script:

<img src=http://i.imgur.com/cBsrPAK.png width="80%" height="80%" border="1">


Chỉnh sửa file cấu hình zabbix agent, thêm UserParameter:

<img src=http://i.imgur.com/ZMvFIln.png width="80%" height="80%" border="1">

Khởi động lại zabbix Agent:

```sh
#service zabbix-agent restart
```
### Dashboard Zabbix

Khởi tạo một host mới với IP là IP của máy agent

<img src=http://i.imgur.com/17NqZk0.png width="80%" height="80%" border="1">

Khởi tạo các item với các key zapache[TotalAccesses], zapache[TotalKBytes],zapache[CPULoad],zapache[Uptime],zapache[ReqPerSec],zapache[BytesPerSec],zapache[BytesPerReq],zapache[BusyWorkers],zapache[IdleWorkers]

<img src=http://i.imgur.com/Omc1ViE.png width="80%" height="80%" border="1">

Kết quả:

<img src=http://i.imgur.com/X5GpBdS.png width="80%" height="80%" border="1">

##Phương pháp 2:

### Trên máy Zabbix Agent


Sửa trong file cấu hình zabbix agent, thêm vào các UserParameter

```sh
vi /etc/zabbix/zabbix_agentd.conf
```

<img src=http://i.imgur.com/fwIlNri.png width="80%" height="80%" border="1">

Trong đó:

- apache.totalaccess: Tổng số truy cập

- apache.totalaccess.get: Tổng số truy cập sử dụng giao thức get

- apache.totalaccess.post: Tổng số truy cập sử dụng giao thức post

- apache.totalaccess.ok: Tổng số truy cập thành công

- apache.totalaccess.fail: Tổng số truy cập thất bại

- apache.access.per.minuse: Số truy cập trong một phút

- apache.totalbyte: Tổng số truy cập

Trên Dashbroad Zabbix khởi tạo host cùng với các key trên để monitor

Kết quả

- Danh sách các item:

<img src=http://i.imgur.com/x9LIr6C.png width="80%" height="80%" border="1">

- Tổng số truy cập:


<img src=http://i.imgur.com/vfMqFym.png width="80%" height="80%" border="1">

## Tài liệu

Những chia sẻ trên có sự tham khảo từ bài viết:

[Lorf Apache](https://github.com/lorf/zapache)

