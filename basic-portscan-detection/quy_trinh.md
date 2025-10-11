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

