# Runbook: Kiểm chứng Supply Chain Security (Trivy + Cosign)

## Mục tiêu
Đảm bảo luồng DevSecOps chặt chẽ: Image phải được quét lỗi bảo mật (Trivy) và ký điện tử (Cosign) trước khi được phép chạy trên Cluster (Sigstore Policy Controller).

## Cấu trúc luồng CI/CD
1. **Build:** Mã nguồn được build thành Docker Image.
2. **Scan (Trivy):** Quét Image. Nếu phát hiện lỗi `HIGH` hoặc `CRITICAL`, pipeline sẽ đánh Fail (exit-code 1) và dừng lại.
3. **Sign (Cosign):** Nếu qua được Trivy, CI dùng Private Key để ký Image.
4. **Deploy (ArgoCD & Admission Controller):** K8s sử dụng `policy-controller` để kiểm tra chữ ký. Chỉ những image có chữ ký khớp với `cluster-image-policy.yaml` mới được tạo Pod.

## Hướng dẫn nghiệm thu 3 tình huống
*(Yêu cầu đã gắn label cho namespace: `kubectl label namespace demo policy.sigstore.dev/include=true`)*

### 1. Push image chứa CVE HIGH
- **Hành động:** Sửa `Dockerfile` để cài một thư viện cũ nhiều lỗi (VD: `requests==2.19.0`), sau đó push code.
- **Kết quả (Kỳ vọng):** Github Actions báo lỗi Đỏ tại step Trivy. Image bị bỏ qua bước ký Cosign.

### 2. Deploy image chưa ký
- **Hành động:** Sửa file `app-api/rollout.yaml` để trỏ tới một image ngẫu nhiên chưa được ký bởi bạn (VD: `nginx:latest` hoặc chính image lỗi vừa bị Trivy chặn ở trên), sau đó push lên Git.
- **Kết quả (Kỳ vọng):** ArgoCD sync thất bại. Kiểm tra log hoặc events (`kubectl get events -n demo`) sẽ thấy Admission Controller reject với lỗi `no signatures found`.

### 3. Deploy image đã ký (từ CI chuẩn)
- **Hành động:** Xóa các thư viện lỗi trong `Dockerfile`, commit và push. 
- **Kết quả (Kỳ vọng):** CI chạy Xanh toàn bộ (Scan pass, Cosign sign). Github Actions tự động update `rollout.yaml` với version mới. ArgoCD sync thành công. Pod mới được tạo và Running vì Admission Controller xác nhận chữ ký hợp lệ.
