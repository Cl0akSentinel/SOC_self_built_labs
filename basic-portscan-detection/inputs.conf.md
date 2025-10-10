# Cấu hình `outputs.conf` cho Splunk Universal Forwarder

File `outputs.conf` ghi ra các nguồn log cần thiết rồi gửi vào Splunk để phát hiện port scan.
Cụ thể: thu EventID=3 của Sysmon (network connect) và log firewall (pfirewall.log).


## Nội dung file `inputs.conf`

```ini
; Collect Sysmon Operational (network connects) + Windows Firewall dropped log

; ----------------------------
; 1) Sysmon Operational channel
; Sysmon cung cấp EventCode=3 (NetworkConnect) — process-level network events
; Chú ý: Tinh chỉnh sysmonconfig.xml để chỉ ghi EventID=3 (và optional EventID=1)
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
; Nếu muốn event-level firewall logs, chuyển disabled = 1 -> 0
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

