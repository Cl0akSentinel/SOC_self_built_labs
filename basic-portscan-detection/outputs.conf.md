# Cấu hình `outputs.conf` cho Splunk Universal Forwarder

File `outputs.conf` xác định nơi Splunk Universal Forwarder gửi dữ liệu (Splunk indexer / host nhận).  
Trong ví dụ dưới, Splunk forwarder gửi đến `<host_ip>:9997` (mặc định port nhận logs là 9997).

**Lưu ý:** Trong repository công khai, giữ `<host_ip>` dưới dạng placeholder. Đưa file cấu hình thật `outputs.conf` vào `.gitignore` nếu nó chứa thông tin nhạy cảm.

---

## Nội dung mẫu `outputs.conf`

```ini
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = <host_ip>:9997

[tcpout-server://<host_ip>:9997]
