# Minimal `inputs.conf` — Phát hiện port scanning (low-volume)

Tệp này chứa một cấu hình `inputs.conf` tối giản, tối ưu cho việc phát hiện port scanning low-volume trên Windows bằng Sysmon và Windows Firewall.

## Mục đích

* Thu thập Sysmon EventID=3 (NetworkConnect) để có event-level network connection từ process.
* Thu thập log Windows Firewall (pfirewall.log) để ghi lại các packet bị drop.
* Giữ noise ở mức thấp bằng `followTail=1` và chỉ bật các nguồn cần thiết.

---

## Hướng dẫn nhanh

1. Đặt `inputs.conf` vào thư mục `C:\Program Files\SplunkUniversalForwarder\etc\system\local` trên Universal Forwarder.
2. Trên máy Windows, bật:

   * Sysmon với cấu hình chỉ ghi `EventID 3` (tùy chọn: giữ `EventID 1` để track process create).
   * Windows Firewall logging -> chỉ bật `Log dropped packets` để giảm noise.
3. Khởi động lại Splunk UF nếu cần: `splunk restart`.

---

## Nội dung tệp `inputs.conf`

```ini
; Minimal inputs.conf optimized for low-volume port-scan detection
; Collect Sysmon Operational (network connects) + Windows Firewall dropped log

; ----------------------------
; 1) Sysmon Operational channel
; Sysmon cung cấp EventCode=3 (NetworkConnect) — process-level network events
; Note: Tinh chỉnh sysmonconfig.xml để chỉ ghi EventID=3 (và optional EventID=1)
; ----------------------------
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = main
sourcetype = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
checkpointInterval = 5
renderXml = false

; ----------------------------
; 2) Windows Firewall text logfile (pfirewall.log)
; Chỉ bật "Log dropped packets" trong Firewall GUI để giảm noise.
; followTail=1: stream dòng mới (không gửi toàn bộ file mỗi lần restart)
; ----------------------------
[monitor://C:\Windows\System32\LogFiles\Firewall\pfirewall.log]
disabled = 0
index = main
sourcetype = windows:firewall
followTail = 1
crcSalt = <SOURCE>

; ----------------------------
; 3) (Optional) Firewall Event Channel - disabled by default (enable nếu cần)
; Nếu bạn muốn event-level firewall logs, chuyển disabled = 1 -> 0
; ----------------------------
[WinEventLog://Microsoft-Windows-Windows Firewall With Advanced Security/Firewall]
disabled = 1
index = main
sourcetype = XmlWinEventLog:Microsoft-Windows-Windows Firewall With Advanced Security/Firewall
checkpointInterval = 5
renderXml = false
```

## Giải thích nhanh các lựa chọn

* `followTail = 1` : chỉ gửi dòng mới. Giảm IO và tránh ghi lại toàn bộ file khi restart.
* `crcSalt = <SOURCE>` : giúp Splunk nhận diện file theo nguồn nếu file path giống nhau trên nhiều hosts.
* `sourcetype` : đặt phù hợp để dễ tạo sourcetypes và parsing trong indexer/searchhead.
* `checkpointInterval` : hướng tới tần suất checkpoint cho WinEventLog. Giá trị 5 là hợp lý cho nhiều hệ thống.

---

## Gợi ý bổ sung

* Tinh chỉnh `sysmonconfig.xml` để chỉ kích hoạt event cần thiết.
* Trên Splunk Indexer: tạo sourcetype transforms/props nếu cần parse thêm `pfirewall.log` thành các trường (action, proto, src_ip, dst_ip, dst_port).
* Thiết lập một rule cơ bản trên SIEM/Splunk để detect tần suất kết nối theo nguồn -> nếu một IP thử nhiều port trên cùng host trong khoảng thời gian ngắn thì raise alert.

