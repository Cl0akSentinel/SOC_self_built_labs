Trước tiên, ta mở port 9997 (mặc định) trên host để nhận logs. 

<img width="1919" height="562" alt="image" src="https://github.com/user-attachments/assets/55a75b21-123f-4ca7-a758-8567e05fcfd2" />

### Kết nối Splunk Universal Forwarder tới Splunk Host

Trong vm Window Server, ta dùng lệnh `splunk start` để khởi động Splunk Universal Forwarder. Tiếp đến, dùng ` splunk add forward-server <ip_host>:<port>` để kết nối tới host qua công 9997, thêm `splunk list forward-server` để kiểm tra kết nối, các kết nối thành công sẽ hiện ở mục Active forwards.


```powershell
PS C:\Users\Administrator> splunk add forward-server <SPLUNK_HOST>:9997
Your session is invalid.  Please login.
Splunk username: <splunk_uf_username>
Password:
<SPLUNK_HOST>:9997 forwarded-server already present
PS C:\Users\Administrator> splunk list forward-server
Active forwards:
        <SPLUNK_HOST>:9997
Configured but inactive forwards:
        None
```

### Cấu hình hai file `inputs.conf` và `outputs.conf` nhằm xác định log types cần gửi và receivers

### Ý nghĩa của hai loại log trong bài lab

**Sysmon log (Operational channel)**  
Ghi chi tiết hành vi tiến trình và kết nối mạng, như: tiến trình nào mở cổng, kết nối tới IP nào, vào thời điểm nào.  
Giúp xác định công cụ quét (ví dụ: nmap, PowerShell, netcat) và mức độ quét (số lượng cổng, tốc độ thực hiện).

**Firewall log (pfirewall.log)**  
Ghi lại các kết nối bị chặn hoặc cho phép, đặc biệt là các gói tin bị DROP khi quét đến cổng đóng.  
Giúp nhận diện tần suất quét, phạm vi port và nguồn thực hiện quét (địa chỉ IP của attacker).

**Kết hợp hai loại log:**  
- Sysmon: cho biết *ai* (process nào) thực hiện quét.  
- Firewall: cho biết *gì* (mục tiêu, cổng nào) bị quét.  

Sự kết hợp này giúp phát hiện và xác thực hành vi **port scanning** từ cả hai phía — *tiến trình thực hiện* và *mạng bị tác động*.

### Thực hiện port scanning

Trước tiên dùng lệnh

`index=* host="<victim_hostname>" 
| stats count by index, sourcetype, source`
để đảm bảo splunk host đã nhận đủ và đúng log types

<img width="1919" height="776" alt="image" src="https://github.com/user-attachments/assets/eb84aad6-a87f-44d6-a410-bd3e62e6e63f" />

## Fast SYN

“SYN” là gói TCP Synchronize, bước đầu tiên trong quá trình bắt tay 3 bước (three-way handshake) để thiết lập kết nối TCP.

Khi thấy rất nhiều gói SYN được gửi nhanh tới nhiều cổng hoặc nhiều host, nhưng không hoàn thành 3-way handshake (không có hoặc ít gói ACK/FIN trả về), điều đó gợi ý một SYN scan (half-open scan).

Đây là kiểu scan mà kẻ tấn công gửi gói SYN để dò cổng mở, nhưng không hoàn tất kết nối, giúp tránh bị log hoặc phát hiện dễ dàng.

**Demo:**

Phía attacker(ubuntu): 

`nmap -sS -p- -T4 --min-rate 1000 <target-or-target-range>`

-sS = SYN scan (half-open). -p- = quét tất cả cổng 0–65535. -T4 = timing faster. --min-rate 1000 buộc gửi tối thiểu 1000 packets/giây → tạo lượng lớn SYN packets nhanh, đúng loại traffic “fast SYN”.

Ta thử nghiệm SPL để từ đó xây dựng detection rules:

<img width="1918" height="850" alt="image" src="https://github.com/user-attachments/assets/5d9b3c15-ee9a-4aef-ad56-9818ac09a1bf" />

Ảnh cho thấy SPL thành công bắt được source IP của máy thực hiện nmap khi phát hiện src IP cố gắng kết nối đến 9 cổng khác nhau của victim và tổng 170 lần kết nối được ghi nhận đến các cổng đó. Trung bình mỗi cổng bị thử khoảng 170 / 9 (lần) -> Rất cao!





