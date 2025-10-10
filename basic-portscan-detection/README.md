## Chuẩn bị môi trường

Mô hình gồm ba thành phần chính:

- **Attacker**:  
  Máy ảo **Ubuntu** có cài công cụ **Nmap** để thực hiện quét port và kiểm tra dịch vụ.

- **Victim**:  
  Máy ảo **Windows Server 2019** có cài **Splunk Universal Forwarder** để gửi logs về host.

- **Host (Splunk Server)**:  
  Máy chủ chạy **Splunk Enterprise** (Indexer/Search Head), tiếp nhận và phân tích log từ Victim, từ đó tạo **detection rules**.

## Lưu ý
Bài lab này khó có thể làm nổi bật hết được những dấu hiệu của hành vi port scanning, tuy nhiên người thực hiện bài lab cũng cố gắng liệt kê một số dấu hiệu nổi bật.

Bài lab trên được thực hiện trong môi trường được kiểm soát, không nên dùng port scanning trong môi trường thực tế để thực hiện hành vi bất hợp pháp.
