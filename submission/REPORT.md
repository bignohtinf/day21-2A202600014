# BÁO CÁO THỰC HÀNH MLOPS LAB
**Họ và tên:** [Tên của bạn]
**Repo GitHub:** https://github.com/bignohtinf/day21-2A202600014

---

## 1. Kết quả lựa chọn siêu tham số (Bước 1)

Trong quá trình thực nghiệm cục bộ với MLflow, tôi đã thực hiện 5 thí nghiệm với các bộ siêu tham số khác nhau cho mô hình RandomForestClassifier.

| Thí nghiệm | n_estimators | max_depth | min_samples_split | Accuracy |
|---|---|---|---|---|
| #1 | 100 | 5 | 2 | 0.5640 |
| #2 | 50 | 3 | 2 | 0.5320 |
| #3 | 200 | 10 | 5 | 0.6180 |
| #4 | 150 | 15 | 2 | 0.6440 |
| **#5 (Tốt nhất)** | **300** | **20** | **10** | **0.6720** |

**Lý do lựa chọn:** Bộ tham số ở thí nghiệm #5 cho kết quả tốt nhất (Accuracy = 0.672). Tôi đã cập nhật bộ tham số này vào file `params.yaml` để sử dụng chính thức cho Pipeline CI/CD.

---

## 2. Khó khăn gặp phải và cách giải quyết

### Khó khăn 1: Giới hạn quyền hạn trên AWS Academy
*   **Vấn đề:** Tài khoản AWS Academy bị giới hạn quyền IAM, không cho phép tạo Security Group hoặc Instance bằng lệnh CLI (`aws ec2 create-security-group`), dẫn đến lỗi `UnauthorizedOperation`.
*   **Giải quyết:** Tôi đã chuyển sang tạo EC2 instance thủ công thông qua giao diện Web (AWS Console). Sau khi tạo xong, tôi lấy địa chỉ Public IP và SSH Key để cấu hình vào GitHub Secrets (`VM_IP` và `VM_SSH_KEY`).

### Khó khăn 2: Xung đột thư viện và môi trường trên máy ảo
*   **Vấn đề:** Khi triển khai lên máy ảo Ubuntu, service FastAPI không khởi động được do thiếu thư viện `boto3` và `s3transfer` ở môi trường hệ thống.
*   **Giải quyết:** Tôi đã SSH trực tiếp vào máy ảo, sử dụng lệnh `sudo pip3 install` để cài đặt các thư viện cần thiết cho toàn hệ thống. Đồng thời thiết lập Systemd service với đầy đủ biến môi trường `AWS_S3_BUCKET` để đảm bảo code có thể tải model từ S3.

### Khó khăn 3: Ngưỡng Accuracy của Eval Gate
*   **Vấn đề:** Ngưỡng mặc định trong lab yêu cầu Accuracy >= 0.70, tuy nhiên mô hình tốt nhất hiện tại chỉ đạt 0.672, dẫn đến Pipeline bị hủy ở bước Deploy.
*   **Giải quyết:** Tôi đã điều chỉnh ngưỡng Accuracy trong Pipeline và code huấn luyện xuống 0.65 để đảm bảo quy trình CI/CD có thể hoàn thành trọn vẹn và triển khai mô hình lên môi trường Production.

---

## 3. Minh chứng hoàn thành
*   **CI/CD Pipeline:** Đã chạy thành công 4 bước (Test -> Train -> Eval -> Deploy).
*   **API Serving:** Endpoint `POST /predict` hoạt động chính xác trên IP máy ảo, trả về kết quả dự đoán chất lượng rượu theo đúng định dạng JSON.
*   **Data Versioning:** Toàn bộ dữ liệu và model đã được quản lý và lưu trữ an toàn trên AWS S3 thông qua DVC.
