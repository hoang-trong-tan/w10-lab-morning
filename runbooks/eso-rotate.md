# Runbook: Rotate Secret không cần Restart Pod với ESO

## Tình huống
Ứng dụng API trước đây đọc Database Password từ Secret dạng plaintext hoặc biến môi trường (ENV). Mỗi khi thay đổi mật khẩu, quản trị viên phải khởi động lại (restart) Pod để ứng dụng nhận giá trị mới. Điều này gây gián đoạn dịch vụ và tốn công sức.

## Giải pháp (External Secrets Operator - ESO)
Chúng ta đã chuyển sang sử dụng AWS Secrets Manager (hoặc Fake Provider) làm nguồn chứa Secret gốc, và dùng ESO để tự động đồng bộ về Kubernetes.

### Tại sao đổi Secret không cần restart Pod?
Bí quyết nằm ở cách chúng ta cung cấp Secret cho Pod:
1. **Dùng Volume Mount thay vì Biến môi trường (ENV):** 
   Nếu dùng ENV, giá trị chỉ được đọc 1 lần duy nhất khi container khởi động. Nếu dùng Volume Mount, K8s sẽ mount Secret như một file trên file system của container.
2. **Kubelet tự động cập nhật:** 
   Khi ESO đồng bộ (sync) giá trị mới từ AWS về K8s Secret (thời gian cấu hình `refreshInterval < 60s`), Kubelet sẽ tự động cập nhật lại nội dung file được mount bên trong container.
3. **Ứng dụng tự đọc lại file:** 
   App chỉ cần được thiết kế để đọc file mỗi khi cần (hoặc dùng thư viện tự động reload) thì sẽ lập tức nhận được mật khẩu mới mà không cần restart container.

## Các bước kiểm chứng (Nghiệm thu)
1. Lấy thông tin Pod hiện tại và thời gian sống (AGE):
   ```bash
   kubectl get pod -n demo -l app=api
   ```
2. Đổi giá trị password trên nguồn (AWS Secrets Manager / Fake Provider).
3. Đợi < 60s, kiểm tra lại K8s Secret:
   ```bash
   kubectl get secret <secret-name> -n demo -o jsonpath='{.data.password}' | base64 -d
   ```
   *Kết quả: Giá trị đã thay đổi.*
4. Kiểm tra lại Pod AGE:
   ```bash
   kubectl get pod -n demo -l app=api
   ```
   *Kết quả: AGE tiếp tục tăng, Pod không bị restart.*
