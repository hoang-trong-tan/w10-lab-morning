# ADR-001: Ngoại lệ cho lỗi bảo mật (CVE) chưa có bản vá

## Trạng thái
**Được chấp thuận (Accepted)**

## Ngữ cảnh
Trong quá trình triển khai CI/CD SecOps với công cụ quét lỗ hổng Trivy, hệ thống được cấu hình để đánh rớt (Fail) pipeline nếu phát hiện bất kỳ lỗi nào ở mức độ `HIGH` hoặc `CRITICAL`.
Tuy nhiên, trong quá trình phát triển, chúng ta có thể gặp các thư viện hệ thống hoặc các gói phụ thuộc (dependencies) có chứa CVE nhưng phía nhà cung cấp (Vendor) chưa phát hành bản vá (no fix yet). Việc block pipeline liên tục trong trường hợp này sẽ gây đình trệ quá trình release sản phẩm.

## Quyết định
Chúng ta sẽ sử dụng file `.trivyignore` đặt tại thư mục gốc của repository để liệt kê danh sách các mã lỗi CVE được miễn trừ tạm thời. 
Bất kỳ ngoại lệ nào được thêm vào file này đều phải tuân thủ các quy tắc sau:
1. Xác nhận rằng Vendor thực sự chưa có bản vá hoặc việc nâng cấp gây ra Breaking Change không thể khắc phục ngay lập tức.
2. Phải đánh giá rủi ro (Risk Assessment) của lỗi đó đối với hệ thống hiện tại.
3. Phải ghi rõ mã lỗi (CVE-ID), lý do và **thời hạn review lại** bằng comment ngay phía trên mã lỗi trong file `.trivyignore`.

## Hậu quả & Đánh giá rủi ro
- **Rủi ro:** Ứng dụng sẽ chạy với một số lỗ hổng đã biết.
- **Giảm thiểu (Mitigation):** Thiết lập lịch nhắc nhở (calendar/alert) để xem xét lại các mã lỗi này định kỳ (ví dụ: mỗi 30 ngày) để kiểm tra xem Vendor đã có bản vá chưa. Khi có bản vá, phải lập tức cập nhật và gỡ bỏ mã lỗi khỏi file `.trivyignore`.
