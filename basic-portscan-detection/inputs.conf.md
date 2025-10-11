# Cấu hình `outputs.conf` cho Splunk Universal Forwarder

File `outputs.conf` ghi ra các nguồn log cần thiết rồi gửi vào Splunk để phát hiện port scan.
Cụ thể: thu EventID=3 của Sysmon (network connect) và log firewall (pfirewall.log).

Đối với firewall logs, ta cần thực hiện:

Properties của Window Defender Firewall with Advanced Security -> Properties -> Private Profile (tối thiểu) -> Customize (Logging) -> Quyết định địa chỉ lưu file log và loại log

  <img width="494" height="462" alt="image" src="https://github.com/user-attachments/assets/61c04370-c3d8-4aa7-a1c9-f737035ca8ef" />


## Nội dung file `inputs.conf`

```ini
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = main
sourcetype = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
checkpointInterval = 5
renderXml = false

; ----------------------------
; 2) Windows Firewall text logfile
; Chỉ bật “Log dropped packets” trong Windows Firewall
; followTail=1: chỉ gửi dòng mới (tránh gửi toàn bộ file khi restart)
; ----------------------------
[monitor://C:\Windows\System32\logs_for_SOC_lab\pfirewall.log]
disabled = 0
index = main
sourcetype = windows:firewall:lab
followTail = 1
crcSalt = <SOURCE>
```

## Giải thích nhanh các lựa chọn

* `followTail = 1` : chỉ gửi dòng mới. Giảm IO và tránh ghi lại toàn bộ file khi restart.
* `crcSalt = <SOURCE>` : giúp Splunk nhận diện file theo nguồn nếu file path giống nhau trên nhiều hosts.
* `sourcetype` : đặt phù hợp để dễ tạo sourcetypes và parsing trong indexer/searchhead.
* `checkpointInterval` : hướng tới tần suất checkpoint cho WinEventLog. Giá trị 5 là hợp lý cho nhiều hệ thống.

