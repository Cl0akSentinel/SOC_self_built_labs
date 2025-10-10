Trước tiên, ta mở port 9997 (mặc định) trên host để nhận logs. 

<img width="1919" height="562" alt="image" src="https://github.com/user-attachments/assets/55a75b21-123f-4ca7-a758-8567e05fcfd2" />

Trong vm window server, ta dùng lệnh `splunk start` để khởi động Splunk Universal Forwarder. Tiếp đến, dùng ` splunk add forward-server <ip_host>:<port>`, thêm `splunk list forward-server` để kiểm tra kết nối, các kết nối thành công sẽ hiện ở mục Active forwards.

Giờ chúng ta cần cấu hình file inputs.conf để quyết định loại logs nào sẽ được gửi đi. Trong bài lab này, ta sẽ dùng firewall log và Sysmon event log.

Firewall cho ta network-level (src_ip → dst_port, action allow/drop, packet info).

Sysmon cho ta host-level (Process Create — EventID 1; Network Connection — EventID 3; file/registry persistence, hashes...) → giúp xác định process nào thực hiện các kết nối hoặc scan.

Kết hợp cả hai cho phép correlate: ví dụ: nhiều dst_port từ 1 src_ip (firewall) + cùng thời điểm một tiến trình lạ tạo nhiều kết nối (Sysmon) → xác thực port scan.


